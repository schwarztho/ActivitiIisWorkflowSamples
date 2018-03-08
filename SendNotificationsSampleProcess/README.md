# How to configure and deploy the sample workflows
This project contains sample processes to send email notifications when IA data quality scores fail the defined benchmark or when IGC terms or categories get changed, or to send Slack notifications for assets imported via IMAM.
* [Preparing IIS and Activiti for the sample processes](#preparing-iis-and-activiti-for-the-sample-processes)
* [Deploy sample workflows](#deploy-sample-workflows)
* [Test sample workflows](#test-sample-workflows)

# Preparing IIS and Activiti for the sample processes
The sample processes do require a few manual configuration steps:
* [Additional IIS setup steps for the sample scenarios](#additional-iis-setup-steps-for-the-sample-scenarios)
* [Additional setup steps to connect to IIS](#additional-setup-steps-to-connect-to-iis)
* [Setup the connection to Slack](#setup-the-connection-to-slack)
* [Configure email notifications](#configure-email-notifications)
* [Deploy SampleConfigProps](#deploy-sampleconfigprops)

Several of the config steps become only active after the server has been restarted. You don't need to restart the server after each step, one final restart suffices. 

[--> top](#how-to-configure-and-deploy-the-sample-workflows)

## Additional IIS setup steps for the sample scenarios

### Create DB2 SAMPLE database - optional
(useful for quickly setting up a demo environment)
Create DB2 and import sample DB with IMAM and DB2 connector: 
```
sudo su - db2inst1
bash
/opt/IBM/DB2/bin/db2sampl
```

### Import sample terms and categories to IGC - optional
(useful for quickly setting up a demo environment)
On the IGC Administration page
* go to Tools / Import...
* select XML and click Next
* click Next again
* browse for [src/main/resources/setup/business-glossary-xml-export.xml](src/main/resources/setup/business-glossary-xml-export.xml)
* and click Import

### Create the iadb data connection
Import "iadb" (must be lower case!) connection via IMAM (connection name: IADB, user: iauser), uncheck all objects, run express import

[-->Preparing IIS and Activiti for the sample processes](#preparing-iis-and-activiti-for-the-sample-processes)

## Additional setup steps to connect to IIS

### WLP only: Import IIS SSL certificate into local trust store
- cd /opt/IBM/InformationServer/wlp/usr/servers/iis

Export IIS SSL Certificate
- keytool -export -alias iisssl -keystore resources/security/iis-server-keystore.p12 -rfc -file resources/security/iis-server.cert
  - default passwort is iiskeypass, unless explicitly changed in the IIS installer

Import IIS SSL Certificate to trust store
- keytool -importcert -alias iis-server -file resources/security/iis-server.cert -keystore resources/security/iis-server-truststore.jks

### configure JAAS user alias for REST calls to IIS
Default JAAS user alias is `activiti-iis-jaas-user-alias`.
If you wish to use a different user alias, then please update sampleConfig.properties as well, and repeat [Deploy SampleConfigProps](#deploy-sampleconfigprops).

#### WLP: configure JAAS user alias for REST calls to IIS
- Modify /opt/IBM/InformationServer/wlp/usr/servers/iis/server.xml
  - Add this feature ```<feature>passwordUtilities-1.0</feature>``` to the ```<featureManager>```
  - Add this JAAS user alias definition ```<authData id="activiti-iis-jaas-user-alias" user="your-user" password="<your-password"/>``` right below the closing ```</featureManager>```
    - password can also be encoded: use the securityUtility encode command
    - you can use a different alias name. If you do so, you need to update the com.ibm.iis.useralias property in the sampleConfig.properties file.
  - restart the server

#### WAS-ND: configure JAAS user alias for REST calls to IIS
- Create jaas user alias for activiti-iis-jaas-user-alias
  - In the WAS admin console, navigate to "Security"/"Global Security" then "Java Authentication and Authorization Service"/"J2C authentication data"
  - Remove the checkmark at "Prefix new alias names with the node name of the cell (for compatibility with earlier releases)" and Click apply
  - Create a new entry in the table below using activiti-iis-jaas-user-alias as the alias name and provide IIS credentials.
  - Click ok and save the changes to the master configuration

[-->Preparing IIS and Activiti for the sample processes](#preparing-iis-and-activiti-for-the-sample-processes)

## Setup the connection to Slack

### Import Slack signer certificate

#### WLP: Import Slack signer certificate
Import Slack Certificate
- cd /opt/IBM/InformationServer/wlp/usr/servers/iis
- save the slack certificate to /home/workflow/slackcom.crt, e.g., by copying SendNotificationsSampleProcess/src/main/resources/setup/slackcom.crt or by directing your browser to https://hooks.slack.com and storing the certificate used there.
- keytool -import -alias slackcom -file /home/workflow/slackcom.crt -keystore resources/security/iis-server-truststore.jks
  - default passwort is iiskeypass, unless explicitly changed in the IIS installer

Requires a server restart to become active.
  
#### WAS-ND: Import Slack signer certificate
Import Slack Certificate
- Open WAS admin console and navigate to "Security"/"SSL certificate and key management" then "Key stores and certificates"/"NodeDefaultTrustStore"/"Signer certificates"
  - Click "Retrieve from port"
  - use hooks.slack.com as host, and 443 as port, and slack.com as alias
  - Click "Retrieve signer information"
  - Click Ok, then save to master configuration
  
### Configure the URL of the inbound Webhook
Slack notifications are sent using a so called Incoming Webhook offered by Slack. You need to configure the URL of that incoming Webhook as follows:
- cd SampleConfigProps
- update com.slack.postmessageurl property in src/main/resources/diagrams/sampleConfig.properties

After updating sampleConfig.properties, please perform [Deploy SampleConfigProps](#deploy-sampleconfigprops).

[-->Preparing IIS and Activiti for the sample processes](#preparing-iis-and-activiti-for-the-sample-processes)

## Configure email notifications


### Configure email provider settings
Configure the email provider in /opt/IBM/InformationServer/ASBServer/conf/common_event_mail.properties as discussed in [Configuring an email provider for event notifications](https://www.ibm.com/support/knowledgecenter/en/SSZJPZ_11.5.0/com.ibm.swg.im.iis.bg.doc/topics/t_configuring_email_provider.html).

### Configure email addreses of iis users
Open the IIS Admin Console, go to Administration / Users and Groups / Users, select a user and open it, then give it an email address and save it.

[-->Preparing IIS and Activiti for the sample processes](#preparing-iis-and-activiti-for-the-sample-processes)

## Deploy SampleConfigProps
Create the sampleConfigProps.bar file
- cd SampleConfigProps
- update properties in src/main/resources/diagrams/sampleConfig.properties
- mvn package
  - this creates sampleConfigProps.bar in the target folder

Deploy sampleConfigProps.bar to Activiti
 - copy sampleConfigProps.bar to /home/workflow/
 - cd /opt/IBM/InformationServer/Clients/istools/cli
 - ./istool.sh workflow deployWorkflow -u isadmin -p yourpassword -au isadmin -ap yourpassword -dom iis-server-name:iis-port -f /home/workflow/sampleConfigProps.bar

[-->Preparing IIS and Activiti for the sample processes](#preparing-iis-and-activiti-for-the-sample-processes)

# Deploy sample workflows
- cd SendNotificationsSampleProcess/src/main/resources/diagrams
- create zip archive SendNotificationSamples.zip containing initEventProcessHelper.bpmn, IgcTermEventTrigger.bpmn, IgcCategoryEventTrigger.bpmn, sendDataQualityBenchmarkScoreFailureEmailSampleProcess.bpmn, sendIgcNotificationEmailSampleProcess.bpmn, sendNotificationSampleProcess.bpmn
- copy SendNotificationSamples.zip to /home/workflow/
- cd /opt/IBM/InformationServer/Clients/istools/cli
- ./istool.sh workflow deployWorkflow -u isadmin -p yourpassword -au isadmin -ap yourpassword -dom localhost:9443 -f /home/workflow/SendNotificationSamples.zip

[--> top](#how-to-configure-and-deploy-the-sample-workflows)

# Check Activiti
- Check deployments https://localhost:9443/ibm/iis/activiti-rest/service/repository/process-definitions
- Check open tasks for all users: https://localhost:9443/ibm/iis/activiti-rest/service/runtime/tasks
- Check open tasks for isadmin: https://localhost:9443/ibm/iis/activiti-rest/service/runtime/tasks?candidateOrAssigned=isadmin&includeTaskLocalVariables=true

# Test sample workflows
* [Test the IGC email notification](#test-the-igc-email-notification)
* [Test the Slack notification](#test-the-slack-notification)
* [Test the IA Data Quality Benchmark Score Failure workflow](#test-the-ia-data-quality-benchmark-score-failure-workflow)

[--> top](#how-to-configure-and-deploy-the-sample-workflows)

## Test the IGC email notification
Don't forget to enable Event Notification on the Administration page in the IGC web UI!
* In the Catelog (or in the Development Glossary), create a new category or a new term, or modify an existing one.
* Clicking Save will trigger an event, which triggers the sample workflow, which sends a notification email, by default to isadmin.
* You can specify the recipients of the notification emails by assigning Stewards to the assets. Each Data Steward needs to have an email address configured in the IIS Admin Console.

## Test the Slack notification
Just do an IMAM import, and you should see a message in the slack channel that is connected the provided incoming webhook.


## Test the IA Data Quality Benchmark Score Failure workflow
- In IA Web UI, go to the settings of your workspace (e.g., UGDefaultWorkspace), and set the Data Quality Threshold value to 99%. Click Save
- On the Data Catalog tab, add the EMPLOYEE table of the SAMPLE database to your workspace
- On the Workspaces tab, open your workspace, and select View All data sets.
- On the EMPLOYEE data set select Analyze Data.
The result of that analysis should be a quality score of 95%, which is below the configured threshold of 99%. This triggers an IA_DATAQUALITY_SCORE_BENCHMARK_FAILURE event, which triggers the sendDataQualityBenchmarkScoreFailureEmailSampleProcess workflow, which sends a notification email to isadmin by default.
- You can configure the recipients of that notification email

### Configure the subscribers
As of 11.7 RC1 you need to use a REST Client to configure the subscribers that receive the emails sent by the IA Data Quality Benchmark Score Failure workflow. Subscribing just the current user can be done with a regular browser as well.

**Display currently subscribed users:** `GET` https://localhost:9443/ibm/iis/ia/api/configuration/dataQuality/subscribers

**Subscribe current user:** `GET` https://localhost:9443/ibm/iis/ia/api/configuration/dataQuality/subscribers/doSubscribe

**Subscribe several users:** `PUT` https://localhost:9443/ibm/iis/ia/api/configuration/dataQuality/subscribers/doSubscribe
- Header: `Content-Type: application/json`
- Body: `{"subscribers":["isadmin","anotheruser"]}`

You can also configure this on a per project level. For this, inject `/project/<projectname>` into the URL, like in the following one:
https://localhost:9443/ibm/iis/ia/api/configuration/project/UGDefaultWorkspace/dataQuality/subscribers

[-->Test sample workflows](#test-sample-workflows)
