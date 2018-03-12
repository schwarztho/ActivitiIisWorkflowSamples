# Sample workflows leveraging the embedded Activiti workflow engine in the IBM Infosphere Information Server
Since Version 11.7.0.1 the IBM Infosphere Information Server has an embedded Activiti workflow engine. This repository contains a couple of sample workflows to illustrate how to create such workflows, how to interact with the IBM Infosphere Information Server in such workflows, and how to deploy and manage them.

Sample workflows:
- [SampleConfigProps](SampleConfigProps/) - Configuration properties for the sample processes, along with the sub process readConfigProps to read the properties file.
- [SendNotificationsSampleProcess](SendNotificationsSampleProcess/) - Sample processes to send notifications via emails and Slack upon specific events. 

Guides:
- Activiti Usage Guide: [Tips and Tricks for working with Activiti](ActivitiUsageTips.md) as a workflow developer
