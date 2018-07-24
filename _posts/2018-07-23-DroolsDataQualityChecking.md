---
layout: post
title: Using Drools (Business Rules Management System) for data quality checking. 
categories: Data analysis
tags: [Drools, Data quality, Elasticsearch, Apache Kafka]
author: TimurSamkharadze
---

# Introduction

In SYSCO AS we deal with a lot of data both structured and unstructured, streamed and batch.    
It comes from different sources and goes to different destinations, and in some situations we want to measure its quality.    
Data format may evolve over time and we need to have possibility to change rules dynamically without restarting working systems.   

In this article we will explain how Drools rule engine may be used to measure data quality,  
where rules are separate from application logic and can be changed at runtime.  

# Tools 

JDK 7+, Maven, preferred IDE or text editor installed.

# Problem description

A system containing data about electrical installations with a lot of data sources from different organizations.   
Data quality problems may occur often in such systems. Date formats, floating point delimiters,  
geodata representation may differ. Typo and incorrect values may occur from time to time. So, let’s gonna fix it!  

We wrote a little micro-service application that fetches data from Elasticsearch instance,  
processes it and puts into a Apache Kafka topic.  

To be able to load rules at runtime we place them in external folder.  

```java
KieServices ks = KieServices.Factory.get();
KieFileSystem kfs = ks.newKieFileSystem();
KieRepository kr = ks.getRepository();

try (Stream<Path> paths = Files.walk(Paths.get("<path to folder with rules>"))) {
	paths.filter(Files::isRegularFile).forEach((path) -> {
			Resource resource = ks.getResources().newFileSystemResource(path.toFile())
					.setResourceType(ResourceType.DRL);
			kfs.write(resource);
		});
}

KieBuilder kb = ks.newKieBuilder(kfs);
kb.buildAll();

KieContainer kContainer = ks.newKieContainer(kr.getDefaultReleaseId());
```

Code that runs batch processing as simple as that

```java
	while (elasticService.hasNextBatch()) {
		Collection<Component> processedBatch = processBatch(kContainer, elasticService.getNextBatch());

		processedBatch.forEach((element) -> {
			kafkaService.put(element.id, objectMapper.toJson(element));
		});
	}
```

Component is simple POJO containing information about processed piece of data and messages from rule engine.

```java
public class Component {
    final String id;
    final String type;
    List<String> messages;
    final Map<String, String> metadata;
	}
```

and here is function that processes batches 

```java
private static Collection<Component> processBatch(KieContainer kContainer, SearchHit[] batch) {

	//create new drools session
	KieSession kSession = kContainer.newKieSession();
	
	//insert every object from batch into drools session
	for (SearchHit hit : batch) {
		Map<String, Object> sourceMap = hit.getSourceAsMap();
		Component c = new Component(hit.getId(), (String) sourceMap.get("type"),
				(Map<String, String>) sourceMap.get("metadata"));
		kSession.insert(c);
	}

	//fire all rules 
	kSession.fireAllRules();

	//getting all processed objects from drools session
	Collection<Component> objects = (Collection<Component>) kSession.getObjects();

	//marking session for garbage colleciton
	kSession.dispose();

	//and returning processed objects
	return objects;
}
```

That's all! 

Now we can take a look of heart and soul of drools powered applications - rules.  
Rules are made using "native" rule language and placed typically into a file with a .drl extension.  

A rule has following structure:  

```
rule "name"
    attributes
    when
        condition 
    then
        action
end
```

Several rules may be placed into one rule file.

### 1. Check component type.

Every component we check should belong to one of two types "GENERIC" or "METER"  
however we may find other types in data we get. Let's check it with one simple drools rule.  

this rule will trigger on every object that has something else than "GENERIC" or "METER" in its type property.  

```
rule "Type check"
    when //condition           
        $component : Component(type not in ("METER", "GENERIC"))
    then //action to take
        $component.messages.add("Type not allowed");
end
```

For every object that satisfies condition an action will be taken. That's all, as simple as it is.  

### 2. Find date and fix date format.

This rule will be applied to every object that has something that sounds like "installed" in it's metadata  
(Yeah, drools may find things using `soundslike` keyword) and convert value to SQL compliant format.  
This rule uses external function written in Java. To be able to use java functions inside of drl files  
we can import them with "import" keyword, as in Java.  

```
rule "Installed date check"
    when
        component : Component($metadata: metadata)
        entry : Entry() from $metadata.entrySet()
        key : String() from entry.getKey()
        value: String() from entry.getValue()
        Boolean(booleanValue == true) from key soundslike  "installed"
    then
        $metadata.put(key, DateUtils.convertDateFormat(value));
end
```

### 3. Validate object id

In our application every component has an id expressed as string that starts with 111 if component has type "METER"  
or 222 if object has type "GENERIC" and we want find all components that violate this naming schema.  

```
rule "ID-Type validation"
    when 
        $component : Component(
            (type == "METER" && id not matches "111*") ||
            (type == "GENERIC" && id not matches "222*")
        )
    then
        $component.messages.add("ID invalid");
end
```

# Conclusion

In this article we covered only one little bit of functionality provided by Drools rule engine that is still enough  
to make rules for data validation and conversion. More complicated rules can do everything that usual Java code can do  
but with full separation from application logic. Rule files may be loaded from jar file as well as from file system. In  
second case you don't need to redeploy your application every time when rules changed but simply reload them.  

# Further reading

[Drools documentation site](https://docs.jboss.org/drools/release/latestFinal/drools-docs/html_single/)