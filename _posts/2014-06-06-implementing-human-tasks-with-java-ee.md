---
layout: post
title: Implementing Human Tasks with any Java EE framework
categories: javaee
tags: [human, task, oracle, soa, bpm, javaee]
author: jeqo
---

Oracle SOA Suite includes a component to handle user interactions into a SCA application:

![SOA Composite with Human Task](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_0840-610x210.png)

When you have completed Human Task definition, JDeveloper gives you the option to implement this tasks using Oracle ADF. This could be the easiest way if you have developers trained to use this framework. In this case you can go ahead and start implementing your task flows and pages. As this option is shown by default, you may think this is the only option to implement a Human Task.

[As Mark Nelson published a couple of years ago](http://redstack.wordpress.com/2012/02/10/writing-a-human-task-ui-in-net-casp-net-or-in-fact-anything-other-than-adf/), you can implement these Human Tasks with .Net and any other technology capable to consume SOAP web services.

In this post, I will explain how to implement Human Tasks with any Java EE framework (including ADF).

This post will include two steps:

How to call your application from Oracle Business Process Workspace
How to integrate your application with Human Task services
Let's started with one simple business process: [https://github.com/jeqo/htjavaee/tree/base_bpm_project](https://github.com/jeqo/htjavaee/tree/base_bpm_project)

This project includes:

![SCA Application](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_0904-610x237.png)

![BPMN Process](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_0904_001.png)

![Human Task Definition](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_0907-610x278.png)

![XML Schema](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_0914.png)

This project was created with [JDeveloper 11g (11.1.1.7)](http://www.oracle.com/technetwork/developer-tools/jdev/downloads/jdeveloper11117-1917330.html)

We could deploy and test this process from Enterprise Manager application.

## Invoke my application from Workspace

Now, how my application would be invoked from Workspace application? As ADF does: adding an URI on Human Task configuration.

1. Go to Enterprise Manager application.
2. Go to the Composite Application deployed and select the Human Task component:

![Composite on EM](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_0925-610x387.png)

3. Go to the Administration tab and add a new URI:

![Human Task configuration](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_0927.png)

This form ask for your application name (only descriptive), and the URL parameters: host, port and context.

4. To test your configuration, let's start a new process instance to create a new task on Workspace application:

![Human Task configuration](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_0934-610x280.png)

And this should be the result:

![Oracle BPM Workspace](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_0936-610x248.png)

This is because obviously we don't deploy an application with this URL (yet).

5. Let's create a new Java EE web application. I'm using NetBeans and Glassfish, you can use your favorite IDE and Java EE application server. Then I will add some JavaScript code to print the request parameters sent by Workspace application:

![NetBeans project - 1](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_0940-610x422.png)

![NetBeans project - 2](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_0941-610x302.png)

![NetBeans project - 3](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_0942-610x195.png)

![NetBeans project - 4](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_0944.png)

Now, try to open your task again:

![NetBeans project - 5](images/2014-06-06-implementing-human-tasks-with-java-ee/Captura.png)

Let's add this code to your HTML page to print the parameters:

```html
    <!DOCTYPE html>
    <html>
        <head>
            <title>Start Page</title>
            <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        </head>
        <body>
            <h1>Hello World!</h1>
            <script>
                // get the current URL
                var url = window.location.toString();
                //get the parameters
                url.match(/\?(.+)$/);
                var params = RegExp.$1;
                // split up the query string and store in an
                // associative array
                var params = params.split("&");
                var queryStringList = {};

                for (var i = 0; i < params.length; i++)
                {
                    var tmp = params[i].split("=");
                    queryStringList[tmp[0]] = unescape(tmp[1]);
                }

                // print all querystring in key value pairs
                for (var i in queryStringList)
                    document.write(i + " = " + queryStringList[i] + "<br/>");
            </script>
        </body>
    </html>
```

Now, you can refresh your task to see all the parameters:

![Human Task parameters](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_1001-610x366.png)

The most important parameters here are: **bpmWorklistTaskId (Task ID)** and **bpmWorklistContent (Security token)**.

Source code: [https://github.com/jeqo/htjavaee/tree/base_java_ee](https://github.com/jeqo/htjavaee/tree/base_java_ee)

## Integrating my Java EE application with Human Task engine

Now that I know how to invoke my application, how can I get the task details (payload, attachments, comments, task metadata)?

There are two basic Web Services available to ask for this information:

* **TaskQueryService**: to get the task details by ID. *http(s)://[host]:[port]/integration/services/TaskQueryService/TaskQueryService?wsdl*
* **TaskService**: to update the task. *http(s)://[host]:[port]/integration/services/TaskService/TaskServicePort?wsdl*

Now we need implement the following steps to implement our Human Task UI with Java EE:

1. Create the SOAP web services clients (JAX-WS)
2. Create the Java classes from Human Task XSDs (JAXB)
3. Implement the UI

### Creating Human Task Web Services Proxies

1. Create a Java project.

2. Then, create a Web Service Proxy using the URLs mentioned above:

![Create WS Proxies](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_1042-610x429.png)

![WS Proxies - 1](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_1046.png)

Validate *"Wrapper Style"* property is disable on Web Service Reference:

![WS Proxies - 2](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_1049-610x409.png)

Now we should have access to these services. To test this, you can use the test case below:

```java
    package com.jeqo.htcore;
 
    import com.oracle.xmlns.bpel.workflow.taskqueryservice.TaskQueryService;
    import com.oracle.xmlns.bpel.workflow.taskqueryservice.TaskQueryService_Service;
    import com.oracle.xmlns.bpel.workflow.taskqueryservice.WorkflowErrorMessage;
    import java.util.logging.Level;
    import java.util.logging.Logger;
    import oracle.bpel.services.workflow.common.model.CredentialType;
    import oracle.bpel.services.workflow.common.model.WorkflowContextType;
    import org.junit.Test;
    import static org.junit.Assert.*;

    /**
     *
     * @author Jorge Quilcate
     */
    public class ServiceAccessTest {

        public ServiceAccessTest() {
        }

        @Test
        public void testAccess() {
            try {
                TaskQueryService_Service taskQueryServiceClient = new TaskQueryService_Service();

                TaskQueryService taskQueryService = taskQueryServiceClient.getTaskQueryServicePort();

                CredentialType credentialType = new CredentialType();
                credentialType.setLogin("weblogic");
                credentialType.setPassword("welcome1");
                credentialType.setIdentityContext("jazn.com");

                System.out.println("Authenticating...");
                WorkflowContextType workflowContextType = taskQueryService.authenticate(credentialType);
                System.out.println("Authenticated to TaskQueryService");

                assertNotNull(workflowContextType);

            } catch (WorkflowErrorMessage ex) {
                Logger.getLogger(ServiceAccessTest.class.getName()).log(Level.SEVERE, null, ex);
            }
        }
    }
```

### Creating Java classes from Human Task XSD

1. Create a JAXB Binding from [Human Task name]Payload.xsd:

![XSD to Java - 1](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_1107-610x470.png)

Compile the project, and now we should have all the required classes to interact with the Human Task engine.

To test this, we can run the following use case:

```java
    package com.jeqo.htcore;

    import com.oracle.xmlns.bpel.workflow.task.Humantask1PayloadType;
    import com.oracle.xmlns.bpel.workflow.taskqueryservice.TaskQueryService;
    import com.oracle.xmlns.bpel.workflow.taskqueryservice.TaskQueryService_Service;
    import com.oracle.xmlns.bpel.workflow.taskqueryservice.WorkflowErrorMessage;
    import java.math.BigInteger;
    import java.util.logging.Level;
    import java.util.logging.Logger;
    import javax.xml.bind.JAXBContext;
    import javax.xml.bind.JAXBElement;
    import javax.xml.bind.JAXBException;
    import javax.xml.bind.Unmarshaller;
    import oracle.bpel.services.workflow.common.model.CredentialType;
    import oracle.bpel.services.workflow.common.model.WorkflowContextType;
    import oracle.bpel.services.workflow.query.model.TaskDetailsByNumberRequestType;
    import oracle.bpel.services.workflow.task.model.Task;
    import org.example.schema.processschema.ProcessType;
    import org.junit.Test;
    import static org.junit.Assert.*;
    import org.w3c.dom.Node;

    /**
     *
     * @author Jorge Quilcate
     */
    public class TaskDetailsAccessTest {

        public TaskDetailsAccessTest() {
        }

        @Test
        public void accessTaskDetails() {
            try {
                TaskQueryService_Service taskQueryServiceClient = new TaskQueryService_Service();

                TaskQueryService taskQueryService = taskQueryServiceClient.getTaskQueryServicePort();

                CredentialType credentialType = new CredentialType();
                credentialType.setLogin("weblogic");
                credentialType.setPassword("welcome1");
                credentialType.setIdentityContext("jazn.com");

                System.out.println("Authenticating...");
                WorkflowContextType workflowContextType = taskQueryService.authenticate(credentialType);
                System.out.println("Authenticated to TaskQueryService");

                TaskDetailsByNumberRequestType taskDetailsRequest = new TaskDetailsByNumberRequestType();
                //Enter a task number running on Oracle BPM
                taskDetailsRequest.setTaskNumber(new BigInteger("200023"));
                taskDetailsRequest.setWorkflowContext(workflowContextType);

                Task task = taskQueryService.getTaskDetailsByNumber(taskDetailsRequest);

                System.out.println("Task: " + task.getSystemAttributes().getTaskId());

                Node payload = (Node) task.getPayload();

                //JAXB Unmarshalling
                JAXBContext context = JAXBContext.newInstance("com.oracle.xmlns.bpel.workflow.task");
                Unmarshaller unmarshaller = context.createUnmarshaller();
                JAXBElement<Humantask1PayloadType> payloadObject = (JAXBElement<Humantask1PayloadType>) unmarshaller.unmarshal(payload, Humantask1PayloadType.class);

                ProcessType dataObject = payloadObject.getValue().getDataObject();

                System.out.println("Payload: " + dataObject.getName());

                assertNotNull(dataObject);

            } catch (WorkflowErrorMessage | JAXBException ex) {
                Logger.getLogger(TaskDetailsAccessTest.class.getName()).log(Level.SEVERE, null, ex);
            }
        }
    }
```

To set the task number, we can use the task already assigned. Go to EM and into the running instance you can check the task number:

![Instance Number](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_1123-610x285.png)

### Implementing the UI

In this case I will use Primefaces 5.0 as JSF implementation.

This is the page flow:

![JSF Page Flow](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_1444.png)

And this is the page code:

![JSF Page Code](images/2014-06-06-implementing-human-tasks-with-java-ee/2014-06-05_1444_001-610x386.png)

The managed bean is the most important class here:

```java
    package com.jeqo.htweb.view;

    import com.oracle.xmlns.bpel.workflow.task.Humantask1PayloadType;
    import com.oracle.xmlns.bpel.workflow.taskqueryservice.TaskQueryService;
    import com.oracle.xmlns.bpel.workflow.taskqueryservice.TaskQueryService_Service;
    import com.oracle.xmlns.bpel.workflow.taskqueryservice.WorkflowErrorMessage;
    import com.oracle.xmlns.bpel.workflow.taskservice.StaleObjectFaultMessage;
    import com.oracle.xmlns.bpel.workflow.taskservice.TaskService;
    import com.oracle.xmlns.bpel.workflow.taskservice.TaskServiceContextTaskBaseType;
    import com.oracle.xmlns.bpel.workflow.taskservice.TaskService_Service;
    import com.oracle.xmlns.bpel.workflow.taskservice.UpdateTaskOutcomeType;
    import java.util.logging.Level;
    import java.util.logging.Logger;
    import javax.annotation.PostConstruct;
    import javax.faces.bean.ManagedBean;
    import javax.faces.bean.ViewScoped;
    import javax.faces.context.FacesContext;
    import javax.servlet.http.HttpServletRequest;
    import javax.xml.bind.JAXBContext;
    import javax.xml.bind.JAXBElement;
    import javax.xml.bind.JAXBException;
    import javax.xml.bind.Marshaller;
    import javax.xml.bind.Unmarshaller;
    import oracle.bpel.services.workflow.common.model.WorkflowContextType;
    import oracle.bpel.services.workflow.query.model.TaskDetailsByIdRequestType;
    import oracle.bpel.services.workflow.query.model.WorkflowContextRequestType;
    import oracle.bpel.services.workflow.task.model.Task;
    import org.w3c.dom.Node;

    /**
     *
     * @author Jorge Quilcate
     */
    @ManagedBean
    @ViewScoped
    public class ApprovalTaskBean {

        private Task task;
        private Humantask1PayloadType payload;
        private WorkflowContextType workflowContext;
        private JAXBElement<Humantask1PayloadType> payloadObject;

        @PostConstruct
        public void init() {
            HttpServletRequest request = (HttpServletRequest) FacesContext.getCurrentInstance().getExternalContext().getRequest();

            String taskId = request.getParameter("bpmWorklistTaskId");
            String context = request.getParameter("bpmWorklistContext");

            System.out.println("Task ID: " + taskId);
            System.out.println("Context: " + context);

            TaskQueryService_Service taskQueryServiceClient = new TaskQueryService_Service();
            TaskQueryService taskQueryService = taskQueryServiceClient.getTaskQueryServicePort();

            try {
                WorkflowContextRequestType getWorkflowContextRequest = new WorkflowContextRequestType();
                getWorkflowContextRequest.setToken(context);

                workflowContext = taskQueryService.getWorkflowContext(getWorkflowContextRequest);
                TaskDetailsByIdRequestType getTaskDetailsByIdRequest = new TaskDetailsByIdRequestType();
                getTaskDetailsByIdRequest.setTaskId(taskId);
                getTaskDetailsByIdRequest.setWorkflowContext(workflowContext);

                task = taskQueryService.getTaskDetailsById(getTaskDetailsByIdRequest);

                Node payloadNode = (Node) task.getPayload();

                JAXBContext jaxbContext = JAXBContext.newInstance("com.oracle.xmlns.bpel.workflow.task");
                Unmarshaller unmarshaller = jaxbContext.createUnmarshaller();
                payloadObject = (JAXBElement<Humantask1PayloadType>) unmarshaller.unmarshal(payloadNode, Humantask1PayloadType.class);
                payload = payloadObject.getValue();
            } catch (WorkflowErrorMessage | JAXBException ex) {
                Logger.getLogger(ApprovalTaskBean.class.getName()).log(Level.SEVERE, null, ex);
            }
        }

        public String approve() {

            try {
                payloadObject.setValue(payload);

                JAXBContext jaxbContext = JAXBContext.newInstance("com.oracle.xmlns.bpel.workflow.task");
                Marshaller marshaller = jaxbContext.createMarshaller();

                marshaller.marshal(payloadObject, (Node) task.getPayload());

                TaskService_Service taskServiceClient = new TaskService_Service();
                TaskService taskService = taskServiceClient.getTaskServicePort();

                TaskServiceContextTaskBaseType updateTaskRequest = new TaskServiceContextTaskBaseType();
                updateTaskRequest.setTask(task);
                updateTaskRequest.setWorkflowContext(workflowContext);

                task = taskService.updateTask(updateTaskRequest);

                UpdateTaskOutcomeType updateTaskOutcomeRequest = new UpdateTaskOutcomeType();
                updateTaskOutcomeRequest.setOutcome("APPROVE");
                updateTaskOutcomeRequest.setTask(task);
                updateTaskOutcomeRequest.setWorkflowContext(workflowContext);

                taskService.updateTaskOutcome(updateTaskOutcomeRequest);
            } catch (StaleObjectFaultMessage | com.oracle.xmlns.bpel.workflow.taskservice.WorkflowErrorMessage | JAXBException ex) {
                Logger.getLogger(ApprovalTaskBean.class.getName()).log(Level.SEVERE, null, ex);
            }

            return "completed";
        }

        public String reject() {
            return "completed";
        }

        public Humantask1PayloadType getPayload() {
            return payload;
        }

        public void setPayload(Humantask1PayloadType payload) {
            this.payload = payload;
        }
    }
```

The init method use the security token to authenticate the application with the Human Task service and find the task by ID.

The approve method updates the task and its outcome.

This video shows the Workspace behavior:

<iframe width="560" height="315" src="//www.youtube.com/embed/bR76-9GSkiU" frameborder="0" allowfullscreen="allowfullscreen"></iframe>