# Tips and Tricks for working with Activiti
This document contains a hand full of helpful procedures to do certain things around Activiti using its REST API.
We use the `curl` tool below since it is very convenient to use, it is scriptable, and you can copy and paste entire command lines from this guide to your shell prompt. Everything below, except for deploying workflows, works also with other REST tools, e.g., the browser based RestClient.

For simplicity, we use `localhost` below. Please replace with the name of the host of your IIS service tier.

To display the headers of the HTTP response, you can add `-i` to the curl command (after `-k`).

To pretty print the json output you can add ` | python -mjson.tool` at the end of each command (but don't combine with `-i`!)

For simple GET requests you can just put the URL into the address bar of your browser.

# List Activiti users
Point your browser to https://localhost:9443/ibm/iis/activiti-rest/service/identity/users

or issue this command:
```
curl -k -s -u isadmin:password https://localhost:9443/ibm/iis/activiti-rest/service/identity/users
```

# Workflow deployment
## List currently deployed process definitions
```
curl -k -s -u isadmin:password https://localhost:9443/ibm/iis/activiti-rest/service/repository/process-definitions
```
You can add further url parameters to narrow down the list. See https://www.activiti.org/userguide/#_list_of_process_definitions for details.

## Deploy a simple Hello World workflow

### Create a simple Hello World workflow
Create a file `SimpleTest.bpmn` using a text editor:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions id="definitions" xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" targetNamespace="http://www.activiti.org/bpmn2.0">
<process id="helloWorld">
<startEvent id="start" />
<sequenceFlow id="flow1" sourceRef="start" targetRef="script" /> <scriptTask id="script" name="HelloWorld" scriptFormat="javascript">
<script>
print("Hello JS world");
</script>
</scriptTask>
<sequenceFlow id="flow2" sourceRef="script" targetRef="theEnd" /> <endEvent id="theEnd" />
</process> </definitions>
```

### Deploy a workflow
This command deploys the file `SimpleTest.bpmn` located in the current directory. This approach allows you to deploy directly from the system with your Activiti development environment, without having to copy the .bpmn file to the IIS system.
```
curl -k -s -X POST -H "Accept: application/json" -F "deployment=@SimpleTest.bpmn" -u isadmin:password https://localhost:9443/ibm/iis/activiti-rest/service/repository/deployments
```

If you want to deploy multiple files at once (into the same deployment), then please add them to a zip archive and provide the zip archive to the above command.

Alternatively, you can also use istool to deploy a workflow file or a zip archive containing multiple workflow files and other resources. For this, you need to copy the files to the system where istool is running.
```
/opt/IBM/InformationServer/Clients/istools/cli/istool.sh workflow deployWorkflow -u isadmin -p password -au isadmin -ap password -dom localhost:9443 -f SimpleTest.bpmn
```

# Run a workflow
```
curl -k -s -X POST -H "Content-Type: application/json" -d '{"processDefinitionKey":"helloWorld"}' -u isadmin:password https://localhost:9443/ibm/iis/activiti-rest/service/runtime/process-instances | python -mjson.tool
```

## Run a specific version of a process definition
First, [list the currently deployed process definitions](#list-currently-deployed-process-definitions) to obtain the id of the process definition. The response looks like this:
```
{"data":[{"id":"helloWorld:1:3","url":"https://loca...
```
Secondly, trigger the process as follows:
```
curl -k -s -X POST -H "Content-Type: application/json" -d '{"processDefinitionId":"helloWorld:1:3"}' -u isadmin:password https://localhost:9443/ibm/iis/activiti-rest/service/runtime/process-instances
```


# Process instances
## List currently running process instances
```
https://localhost:9443/ibm/iis/activiti-rest/service/runtime/process-instances
```
You can add further url parameters to narrow down the search, see https://www.activiti.org/userguide/#restProcessInstancesGet for details.

## Terminate a process instance
Note: It is always better to interact with a process in the designed way and make it come to a defined end. Only if you run out of options in doing so, you can forcefully terminate a running process by providings its processInstanceId to the following command:
```
curl -k -s -X DELETE -u isadmin:password https://localhost:9443/ibm/iis/activiti-rest/service/runtime/process-instances/{processInstanceId}
```

# Deploy the Eclipse-based Activiti Editor
- Get a suitable Eclipse distribution, e.g., Eclipse IDE for Java Developers (x64) from here: http://www.eclipse.org/downloads/packages/release/Kepler/SR2
- Extract the zipfile to somewhere and run eclipse.exe
- Follow the steps here to deploy the Activiti Editor: https://www.activiti.org/userguide/#eclipseDesignerInstallation
    - The following installation instructions are verified on Eclipse Kepler and Indigo. Note that Eclipse Helios is NOT supported.
    - Go to Help → Install New Software. In the following panel, click on Add button and fill in the following fields:
        - *Name:*Activiti BPMN 2.0 designer
        - *Location:*http://activiti.org/designer/update/
    - Make sure the "Contact all updates sites.." checkbox is checked, because all the necessary plugins will then be downloaded by Eclipse.

