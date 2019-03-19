---
layout: post
title: Setting up Camunda BPMN workflow engine
categories: bpm
tags: [BPM, DMN, Camunda]
author: IvanStevanovic
---



### Motivation

This post is created with an intention to provide an example for creating a business process workflow using Camunda platform. 
Considering the level of details for configuration and runtime of a business process, 
provided demo process will be used to highlight crucial components.

### Introduction

Camunda is a workflow and decision automation platform. It consists of tools for defining and running business processes 
(using BPMN 2.0), as well as defining and executing complex decisions during the execution of a business process (using DMN 1.1).

Following components of Camunda platform were used in order to create this business process sample:
- Modeler
- BPMN process engine
- DMN decision engine

### Demo
The purpose of the demo is to present the basic Camunda functionalities using simple example. The focus of this demo 
will be on technical capabilities of the BPM framework, mocking the data used in the demo itself. BPMN process engine 
(the main component for executing business processes) used in this demo is deployed on a Tomcat server, which can 
be downloaded from [Camunda.org](https://camunda.org/release/camunda-bpm/). The other way of deploying the process engine 
is to embed it into your own application and execute business processes inside.

Camunda process engine support business processes defined using BPMN2.0 format, so the Camunda platform comes with [Camunda Modeler](https://camunda.com/products/modeler/) 
\- desktop application which can be used to visually model and deploy the process definition. The output of the Camunda Modeler is a 
```.bpmn``` file which will be examined in the following sections. 

The demo business process will simulate starting an external RPA process, collect the data and send the data for 
validation to a user. The process can be defined in BPMN2.0 as:
![Process](/images/2019-03-20-Setting-up-Camunda-BPMN-workflow-engine/bpm_process.jpeg)

####  Sending the request to RPA system
In order to execute business tasks performed by machine, [Service task](https://docs.camunda.org/manual/7.7/reference/bpmn20/tasks/service-task/)
type of task is used. Camunda provides Java extension point for integrating a custom code into task execution in form of 
[JavaDelegate](https://docs.camunda.org/javadoc/camunda-bpm-platform/7.11/org/camunda/bpm/engine/delegate/JavaDelegate.html) interface. 
Also, in the process definition, Camunda provides custom attribute ```camunda:class``` for binding between service task and instance of JavaDelegate.
Configuration and mock implementation of the "Send request to RPA" task are defined in following manner (together with transition elements):
```xml
<bpmn:sequenceFlow id="start-send_to_rpa" sourceRef="StartEvent_demo" targetRef="sendRequestToRPATask" />
<bpmn:serviceTask id="sendRequestToRPATask" name="Send request to RPA"
                  camunda:class="org.sysco.camunda.tasks.SendToRPADelegate">
  <bpmn:incoming>start-send_to_rpa</bpmn:incoming>
  <bpmn:outgoing>send_to_rpa-wait_for_rpa</bpmn:outgoing>
</bpmn:serviceTask>
<bpmn:sequenceFlow id="send_to_rpa-wait_for_rpa" sourceRef="sendRequestToRPATask" targetRef="waitForRPA"/>
```
```java
import org.camunda.bpm.engine.delegate.DelegateExecution;
import org.camunda.bpm.engine.delegate.JavaDelegate;
import org.sysco.camunda.service.RPASystem;
import org.sysco.camunda.service.impl.RPASystemMock;

public class SendToRPADelegate implements JavaDelegate {

    // mock implementation which can be bootstrapped with Spring framework
    private RPASystem rpaSystem = new RPASystemMock();

    public void execute(DelegateExecution delegateExecution) {
        String requestID = rpaSystem.sendRequest();
        delegateExecution.setVariable("rpaRequestId", requestID);
    }
}
```
[DelegateExecution](https://docs.camunda.org/javadoc/camunda-bpm-platform/7.11/org/camunda/bpm/engine/delegate/DelegateExecution.html) is a way to get the context of the running business process (from now on **process instance**). This context contains information about process instance and process definition, but most interesting are process variables which are suitable for storing all sort of data necessary during the instance execution.
#### Checking te result from RPA system
A RPA process can last for some time, so one way to model the fetching of the data from an external system is to 
pause the execution of running process instance for some time and check the result afterwards. This is accomplished by using an 
```intermediateCatchEvent``` element with specified waiting time. After waiting for specified time, service task is used 
to check if the RPA process is finished. If it is finished, execution continues, otherwise execution will wait 
(30sec in this case) and check the RPA process again. The interesting point is the notation of the timer 
(```<bpmn:timeDuration>``` element), which uses the [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format.
```xml
<bpmn:sequenceFlow id="send_to_rpa-wait_for_rpa" sourceRef="sendRequestToRPATask" targetRef="waitForRPA"/>
<bpmn:intermediateCatchEvent id="waitForRPA" name="30 sec RPA wait">
  <bpmn:incoming>send_to_rpa-wait_for_rpa</bpmn:incoming>
  <bpmn:incoming>check_rpa_gateway-wait_for_rpa</bpmn:incoming>
  <bpmn:outgoing>wait_for_rpa-check_rpa</bpmn:outgoing>
  <bpmn:timerEventDefinition>
    <bpmn:timeDuration>PT30S</bpmn:timeDuration>
  </bpmn:timerEventDefinition>
</bpmn:intermediateCatchEvent>
<bpmn:sequenceFlow id="wait_for_rpa-check_rpa" sourceRef="waitForRPA" targetRef="checkRPATask"/>
<bpmn:serviceTask id="checkRPATask" name="Check RPA" camunda:class="org.sysco.camunda.tasks.CheckRPADelegate">
  <bpmn:incoming>wait_for_rpa-check_rpa</bpmn:incoming>
  <bpmn:outgoing>check_rpa-check_rpa_gateway</bpmn:outgoing>
</bpmn:serviceTask>
<bpmn:sequenceFlow id="check_rpa-check_rpa_gateway" sourceRef="checkRPATask" targetRef="checkIfRPAFinishedGateway"/>
<bpmn:exclusiveGateway id="checkIfRPAFinishedGateway">
  <bpmn:incoming>check_rpa-check_rpa_gateway</bpmn:incoming>
  <bpmn:outgoing>check_rpa_gateway-wait_for_rpa</bpmn:outgoing>
  <bpmn:outgoing>check_rpa_gateway-determine_parser_rule</bpmn:outgoing>
</bpmn:exclusiveGateway>
<bpmn:sequenceFlow id="check_rpa_gateway-wait_for_rpa" name="RPA not finished"
                   sourceRef="checkIfRPAFinishedGateway" targetRef="waitForRPA">
  <bpmn:conditionExpression>${rpaFinished == false}</bpmn:conditionExpression>
</bpmn:sequenceFlow>
<bpmn:sequenceFlow id="check_rpa_gateway-determine_parser_rule" name="RPA finished"
                   sourceRef="checkIfRPAFinishedGateway" targetRef="determineParserRuleTask">
  <bpmn:conditionExpression>${rpaFinished == true}</bpmn:conditionExpression>
</bpmn:sequenceFlow>
```
```java
import org.camunda.bpm.engine.delegate.DelegateExecution;
import org.camunda.bpm.engine.delegate.JavaDelegate;
import org.sysco.camunda.model.CheckRPAResponse;
import org.sysco.camunda.service.RPASystem;
import org.sysco.camunda.service.impl.RPASystemMock;

public class CheckRPADelegate implements JavaDelegate {

    // mock implementation which can be bootstrapped with Spring framework
    private RPASystem rpaSystem = new RPASystemMock();

    public void execute(DelegateExecution delegateExecution) {
        Object requestId = delegateExecution.getVariable("rpaRequestId");
        CheckRPAResponse response = rpaSystem.checkRpa((String) requestId);

        delegateExecution.setVariable("rpaFinished", response.isFinished());

        if (response.isFinished()) {
            delegateExecution.setVariable("contentType", response.getContentType());
            delegateExecution.setVariable("content", response.getContent());
        }
    }
}
```

#### Parsing the response from RPA system
After making sure that the RPA process has finished, requirement for this business process is to parse the response from RPA system.
Based on a type of the response from RPA, decision model can be created for determining the type of parser required for the specific response. 
Considering that RPA response contains ```contentType``` field, decision model can be created in form of 
[DMN Decision Table](https://docs.camunda.org/manual/7.7/reference/dmn11/decision-table/). 
Using [DMN 1.1](https://docs.camunda.org/manual/7.7/reference/dmn11/), it's very convenient to integrate into existing 
process definition by creating [Business rule](https://docs.camunda.org/manual/7.7/reference/bpmn20/tasks/business-rule-task/) task.
Camunda Modeler also supports creation and deployment of DMN Decision Tables.

The following decision table is very simple, with an intention to show capabilities for integrating into process definition, rather than have complex decision model. 
![DecissionTable](/images/2019-03-20-Setting-up-Camunda-BPMN-workflow-engine/dmn_decision_table.png)

```xml
<decision id="fileFormatDecision" name="FileFormatDecision">
<decisionTable id="fileFormatDecisionTable">
  <input id="contentTypeInput">
    <inputExpression id="contentTypeExpression" typeRef="string">
      <text>contentType</text>
    </inputExpression>
  </input>
  <output id="fileFormatOutput" name="fileFormat" typeRef="string" />
  <rule id="jsonRule">
    <inputEntry id="jsonContentType">
      <text>"application/json"</text>
    </inputEntry>
    <outputEntry id="jsonFileFormat">
      <text>"json"</text>
    </outputEntry>
  </rule>
  <rule id="csvRule">
    <inputEntry id="csvContentType">
      <text>"text/csv"</text>
    </inputEntry>
    <outputEntry id="csvFileFormat">
      <text>"comma-separated"</text>
    </outputEntry>
  </rule>
</decisionTable>
</decision>
```

Binding between decision table and process definition is achieved using [custom attributes](https://docs.camunda.org/manual/7.7/reference/bpmn20/tasks/business-rule-task/#camunda-extensions):
- ```camunda:decisionRef``` - the *id* of deployed decision  
- ```camunda:mapDecisionResult``` - mapper between decision result and process instance variable
- ```camunda:resultVariable``` - process instance variable which will contain the decision result

```decisionResult``` is the variable name (hardcoded in Camunda) which is used to map the output to desired process variable.
```xml
<bpmn:businessRuleTask id="determineParserRuleTask" name="Determine Parser" camunda:decisionRef="fileFormat" 
                           camunda:resultVariable="fileFormat" camunda:mapDecisionResult="singleEntry">
  <bpmn:extensionElements>
    <camunda:inputOutput>
      <camunda:outputParameter name="fileFormat">${decisionResult.getSingleResult().fileFormat}
      </camunda:outputParameter>
    </camunda:inputOutput>
  </bpmn:extensionElements>
  <bpmn:incoming>check_rpa_gateway-determine_parser_rule</bpmn:incoming>
  <bpmn:outgoing>determine_parser_rule-determine_parser-gateway</bpmn:outgoing>
</bpmn:businessRuleTask>
<bpmn:sequenceFlow id="determine_parser_rule-determine_parser-gateway" sourceRef="determineParserRuleTask"
                       targetRef="determineParserGateway"/>
<bpmn:exclusiveGateway id="determineParserGateway">
  <bpmn:incoming>determine_parser_rule-determine_parser-gateway</bpmn:incoming>
  <bpmn:outgoing>determine_parser_gateway-parse_csv</bpmn:outgoing>
  <bpmn:outgoing>determine_parser_gateway-parse_json</bpmn:outgoing>
</bpmn:exclusiveGateway>
<bpmn:sequenceFlow id="determine_parser_gateway-parse_json" name="json" sourceRef="determineParserGateway"
                   targetRef="parseJsonTask">
  <bpmn:conditionExpression>${fileFormat == 'json'}</bpmn:conditionExpression>
</bpmn:sequenceFlow>
<bpmn:serviceTask id="parseJsonTask" name="Parse JSON" camunda:class="org.sysco.camunda.tasks.JsonParserDelegate">
  <bpmn:incoming>determine_parser_gateway-parse_json</bpmn:incoming>
  <bpmn:outgoing>parse_json-parsing_finished</bpmn:outgoing>
</bpmn:serviceTask>
<bpmn:sequenceFlow id="determine_parser_gateway-parse_csv" name="csv" sourceRef="determineParserGateway"
                   targetRef="parseCsvTask">
  <bpmn:conditionExpression>${fileFormat == 'comma-separated'}</bpmn:conditionExpression>
</bpmn:sequenceFlow>
<bpmn:serviceTask id="parseCsvTask" name="Parse CSV" camunda:class="org.sysco.camunda.tasks.CsvParserDelegate">
  <bpmn:incoming>determine_parser_gateway-parse_csv</bpmn:incoming>
  <bpmn:outgoing>parse_csv-parsing_finished</bpmn:outgoing>
</bpmn:serviceTask>
<bpmn:sequenceFlow id="parse_json-parsing_finished" sourceRef="parseJsonTask" targetRef="parsingFinishedGateway"/>
<bpmn:sequenceFlow id="parse_csv-parsing_finished" sourceRef="parseCsvTask" targetRef="parsingFinishedGateway"/>
<bpmn:exclusiveGateway id="parsingFinishedGateway">
  <bpmn:incoming>parse_json-parsing_finished</bpmn:incoming>
  <bpmn:incoming>parse_csv-parsing_finished</bpmn:incoming>
  <bpmn:outgoing>parsing_finished-validate_data</bpmn:outgoing>
</bpmn:exclusiveGateway>
```

Beside using DMN for making the decisions in running workflow, Business Rule Management (BRM) tools can also be integrated within *Business rule* task.
It does require some configuration tweaking, but it can be used as a powerful way to both descriptively and programmatically define desired set of rules. 
Great tutorial for using Drools as BRM can be found [here](http://blog.sysco.no/data/analysis/DroolsDataQualityChecking/).

#### Validating the data
As a last task of this workflow, user has to validate the result of the parsing. Therefore [User task](https://docs.camunda.org/manual/7.7/reference/bpmn20/tasks/user-task/) 
type of task is necessary.
```xml
<bpmn:sequenceFlow id="parsing_finished-validate_data" sourceRef="parsingFinishedGateway" targetRef="validateDataTask" />
<bpmn:userTask id="validateDataTask" name="Validate Data" camunda:assignee="john">
  <bpmn:incoming>parsing_finished-validate_data</bpmn:incoming>
  <bpmn:outgoing>validate_data-end</bpmn:outgoing>
</bpmn:userTask>
<bpmn:sequenceFlow id="validate_data-end" sourceRef="validateDataTask" targetRef="EndEvent_demo" />
``` 
In this case custom attribute ```camunda:assignee``` has to be used, in order to define a user (or a group) capable of performing the task. 
What is interesting is that this type of task is highly extendable, so source of the assignee user (or the group) can be any system.
For this demo process, user is registered in existing [Tomcat web application](#demo).

#### Conclusion
Even though this demo process is not robust as it should be in real life situation, it can be used as a starting point 
for building complex workflows or generally learning about BPM. Camunda as a product, provides tools for creating, 
managing and running business processes and decision models, with great integration capabilities. 
Another huge advantage is that the process engine can be deployed and ready for use within few minutes, 
so you can execute your first workflow in less than an hour. 