# Installation

The SymphonySummit Connector installation consists of multiple steps that are required to complete the installation successfully. 

The connector requires a SMA OpCon Windows Agent to provide the connection between Notification Manager in the OpCon System and the SymphonySummit Connector software. 
It requires the OpCon Rest-API to extract the unique id of the task in the daily tables and the retrieval of the task's job log.
It uses the OpCon connection to update the returned incident number in the task in the Daily tables.

## Supported Software Levels
The following software levels are required to implement the SymphonySummit Connector.

- OpCon Release 21.0 or higher.
- Embedded Java OpenJDK 11 (part of installation).
- OpCon Rest API Configured to use TLS.
- OpCon Windows Agent to provide link to EasyVista Connector.
- OpCon Notification Manager.
- A Symphony Summit implementation that supports the required Rest API.

## Installation
The installation process consists of the following steps:

- OpCon Windows Agent Installation.
- SymphonySummit Connector Installation.
- SymphonySummit Connector Configuration.
- OpCon Notification Manager Definition
 
### OpCon Windows Agent Installation
Copy the supplied install file SMASymphonySummitConnector-win.zip and extract it into the installation directory.

After the installation is complete, the root installation directory contains the connector executable (SMASymphonySummit.exe), the encryption software executable (EncryptValue.exe), the Connector.config file and four directories, java, joblogs, templates and log. The java directory contains the java software required to execute the connector (OpenJDK 11), the joblogs directory is used to temporarily contain job logs extracted from the OpCon system, the log directory contains the connector log files and the templates directory contains the SymphonySummit template files.

### SymphonySummit Connector Installation
The SymphonySummit connector must be installed on the OpCon Windows Server as the Run Command option of the Notificaton Manager is used to invoke it.
Copy the downloaded install file SMASymphonySummitConnector-win.zip and extract it into a temp directory (c:\temp). Extract the information including sub-directories into the required directory.

#### Create $SCHEDULE DATE-SSUM Global Property
Create the special **$SCHEDULE DATE-SSUM** global property that contains the schedule date in the yyyy-MM-dd format from the standard $SCHEDULE DATE property. Set the value to **yyyy-MM-dd**.

### SymphonySummit Connector Configuration
The configuration of the SymphonySummit Connector requires setting the OpCon connection information for the OpCon system associated with the connector, setting the default ticket values and creating a template.

All user and password values placed in the configuration and template files must be encrypted using the Encrypt.exe utility provided with the connector. 

#### EncryptValue Utility
The EncryptValue utility uses standard 64 bit encryption.

Supports a -v argument and displays the encrypted value

On Windows, example on how to encrypt the value "abcdefg":

```
EncryptValue.exe -v "abcdefg"

```

#### Connector.config configuration
Configure the Connector.config file in the installation directory setting the required information.
The Connector.config contains the following values

Property Name | Value
--------- | -----------
**[GENERAL]**                       | header
**JOBLOGDIR**                       | The name of the directory where the retrieved log files are stored. After successful attachment to the EasyVista Incident, the log file is deleted. The name is a sub directory of the installation directory (default joblogs).
**TEMPLATESDIR**                    | The name of the directory where the template definitions are stored. The name is a sub directory of the installation directory (default templates).
**DAILY_START_HOUR**                | The hour start of the daily batch processing (value is hour, i.e. 07 for 07:00)  
**DEBUG**                           | The Connector supports a debug mode which can be enabled by setting the value to ON. The connector should be run with DEBUG disabled (OFF) and enabled (ON) when requested to capture an error condition. Value either ON or OFF (default OFF).
**[DEFAULTS]**                      | header - Used to set ticket default values
**PRIORITY_NAME_VALUE**             | Sets the default value for the CategoryName attribute (default P3).
**IMPACT_NAME_VALUE**               | Sets the default value for the ImpactName attribute (default Medium).
**URGENCY_NAME_VALUE**              | Sets the default value for the UrgencyName attribute (default Medium).
**SUP_FUNCTION_VALUE**              | Sets the default value for the SupFunction attribute (default IT).
**MEDIUM_VALUE**                    | Sets the default value for the Medium attribute (default Web).
**CLASSIFICATION_NAME_VALUE**       | Sets the default value for the ClassificationName attribute (default Application Support).
**SOURCE_VALUE**                    | Sets the default value for the Source attribute (default Event Trigger).
**CATEGORY_NAME_VALUE**             | Sets the default value for the CategoryName attribute (default ElasticSearch).
**ASSIGNED_WORK_GROUP_NAME_VALUE**  | Sets the default value for the AssignedWorkGroupName attribute (default DevOps).

**[PROXY SERVER]**                  | header - Used to define the full URL of a proxy server if required
**USES_PROXY**                      | Indicates if this connector should use a Proxy Server connection. Values are True or False (default False).
**ADDRESS**                         | The address of the Proxy Server.
**PORT**                            | The port of the Proxy Server.
**[OPCON API]**                     | header - Used to define the connection to the OpCon API
**ADDRESS**                         | The server address of the OpCon API.
**PORT**                            | The port number used by the OpCon API server.
**USES_TLS**                        | Must be set to True.
**TOKEN**                           | An application token used for Authentication when communicating with the OpCon-API. 

Example configuration file. 

```
[GENERAL]
JOBLOGDIR=joblogs
TEMPLATESDIR=templates
DAILY_START_HOUR=07
DEBUG=OFF

[DEFAULTS]
PRIORITY_NAME_VALUE=P3
IMPACT_NAME_VALUE=Medium
URGENCY_NAME_VALUE=Medium
SUP_FUNCTION_VALUE=IT
MEDIUM_VALUE=Web
CLASSIFICATION_NAME_VALUE=Application Support
SOURCE_VALUE=Event Trigger
CATEGORY_NAME_VALUE=ElasticSearch
ASSIGNED_WORK_GROUP_NAME_VALUE=DevOps

[PROXY SERVER]
USES_PROXY=False
SERVER=
PORT=

[OPCON API]
SERVER=BVHTEST02
PORT=9010
USES_TLS=True
TOKEN=fc0520dc-fc93-4d3a-bf2f-7d0584c69df2

```
The OPCON API section provides the information about connecting to the OpCon System using the OpCon Rest-API so the failed task information (incident ID number, tags, job logs, etc) can be retrieved from the OpCon task. The SERVER, PORT USES_TLS and TOKEN statements provide the definitions that will allow the SymphonySummit connector to connect to the OpCon Rest API. The TOKEN statement contains an application token (see OpCon Rest-API documentation on how to generate an application token).

#### Templates
Templates provide information about the SymphonySummit connection and the definitions that will be submitted in the JSON payload as part of the request. A default template can be found in the templates directory of the connector installation.

The template includes the address of the Symphony Summit instance, the credentials, the rules associated with the connector, descriptions to be used, attributes to be included and tag routing definitions.

A template includes the following definitions:

Attribute Name Name | Value
--------- | -----------
**ticketDescription**   	                     | Defines the description to use when creating a ticket. See section on Customized Description and Title Definitions.
**ticketInformation**                          | Defines the information field to use when creating a ticket. See section on Customized Description and Title Definitions.
**address**                                    | header - Symphony Summit instance address information
**name**                                       | A name to indicate the Symphony Summit instance.
**value**                                      | The address of the SymphonySummit instance. 
**credentials**                                | header	- credential information.
**apiKey**                                     | The api key which has the required privileges to connect to the Symphony Summit Instance to submit requests. The apiKey must be encrypted using the EncryptValue.exe utility .
**rules**                                      | header - Defines what functions are supported by the connector.
**includeJobLogAttachment**                    | Indicates if the job log should be attached to the incident ticket. Value either true or false (default true).
**includeTagRouting**                          | Indicates if User defined tags should be used for incident routing purposes. Value either true or false (default false). See section Tag Routing. Note that either includeTagRouting or includeWorkGroupNameTag can be enabled, not both. 
**includeCategoryNameTag**                     | Indicates if using tags to define the CategoryName attribute is enabled. Value either true or false (default false). See section Category Naming using tags.
**includeWorkGroupNameTag**                    | Indicates if using tags to define the AssignedWorkGroupName attribute is enabled. Value either true or false (default false). See section Assigning WorkGroup Names using tags. Note that either includeTagRouting or includeWorkGroupNameTag can be enabled, not both.
**submitSingleIncidentPerDay**                 | Indicates if a single incident should be submitted if the task fails after restart on the same calendar day. The configuration value DAILY_START_HOUR determines the hour to check from.  
**urls**	                                     | header - Defines urls used by the connector. 
**name**                                       | A name that indicates the url usage (i.e. incident, attachment, etc).
**value**                                      | The url definition minus the address portion. 
**workingHours**                               | header - Defines what is working hours. Using these definitions allows the definitions of a different set of attributes for working and non-workings hours. Note Consists of start,stop pairs.
**monday**	                                   | header - Defines the start and stop hours for Monday.
**start**                                      | The start time. Value consists of four digits (HHMM).
**stop**                                       | The start time. Value consists of four digits (HHMM)
**tuesday**	                                   | header - Defines the start and stop hours for Tuesday.
**start**                                      | The start time. Value consists of four digits (HHMM).
**stop**                                       | The start time. Value consists of four digits (HHMM)
**wednesday**                                  | header - Defines the start and stop hours for Wednesday.
**start**                                      | The start time. Value consists of four digits (HHMM).
**stop**                                       | The start time. Value consists of four digits (HHMM)
**thursday**                                   | header - Defines the start and stop hours for Thursday.
**start**                                      | The start time. Value consists of four digits (HHMM).
**stop**                                       | The start time. Value consists of four digits (HHMM)
**friday**	                                   | header - Defines the start and stop hours for Friday.
**start**                                      | The start time. Value consists of four digits (HHMM).
**stop**                                       | The start time. Value consists of four digits (HHMM)
**saturday**                                   | header - Defines the start and stop hours for Saturday.
**start**                                      | The start time. Value consists of four digits (HHMM).
**stop**                                       | The start time. Value consists of four digits (HHMM)
**sunday**	                                   | header - Defines the start and stop hours for Sunday.
**start**                                      | The start time. Value consists of four digits (HHMM).
**stop**                                       | The start time. Value consists of four digits (HHMM)
**attributes**                                 | header - defines the attribute names that will override the default ticket values defined in the Connector.config.
**name**                                       | A name that indicates the ticket attribute name (i.e. Priority_Name, Impact_Name, Urgency_Name, Classification_Name, Sup_Function, Medium, Source).
**value**                                      | The value to be assigned to the attribute. 
**workingHoursAttributes**                     | header - defines the attribute names that will override the default ticket values defined in the Connector.config during working hours.
**name**                                       | A name that indicates the ticket attribute name (i.e. Priority_Name, Impact_Name, Urgency_Name, Classification_Name, Sup_Function, Medium, Source).
**value**                                      | The value to be assigned to the attribute. 
**nonWorkingHoursAttributes**                  | header - defines the attribute names that will override the default ticket values defined in the Connector.config during non working hours.
**name**                                       | A name that indicates the ticket attribute name (i.e. Priority_Name, Impact_Name, Urgency_Name, Classification_Name, Sup_Function, Medium, Source).
**value**                                      | The value to be assigned to the attribute. 
**customAttributes**                           | header - defines the custom attribute that will added to the ticket information.
**groupName**                                  | A group name associated with the custom attribute.
**name**                                       | the name of the custom attribute.
**value**                                      | The value to be assigned to the attribute. 
**tags**                                       | header - Defines information if OpCon User defined Tags are to be used to include attributes in the ServiceNow submission. This is enabled if the rule **includeTagRouting** is set to True.
**indicator**                                  | Defines what part of the tag should be used to identify the request. Supports TAG_END,  TAG_START, , DEFAULT or CATNAME. The DEFAULT value is used if there is no TAG_END or TAG_START match and TAG Routing is enabled. The EXIT tag is used to stop a ticket from being created and the CATNAME tag is used to define what value should be set for the Category_Name attribute.
**indicatorValue**	                           | The value that is matched to the OpCon User defined tag (either the end or the start).
**attribute**	                                 | Defines the attribute name that will be added to the JSON payload along with the value.
**value**                                      | The value associated with the attribute.

The address section of the template defines the address information associated with the Symphony Summit Instance. This allows a single connector the ability to submit requests to multiple Symphony Summit instances by simply creating multiple templates. The address section is required.

The rules section of the template defines which rules should be used for the request. Values are either true or false. The rules section is required.

The credentials section of the template defines the Symphony Summit ApiKey to be used for the connection. This value must be encrypted using the EncryptValue.exe program. The credentials section is required.

The workingHours section defines what is working hours. This allows a different set of attributes to defined when creating an incident for working and non-working hours.

The attributes section defines ticket attribute values that will override the default values, while the workingHoursAttributes and nonWorkingHoursAttributes sections define ticket attribute values that will be overridden in those time slots.

The customAttributes section defines attributes values that will obe includes in teh CustomFields section of the ticket information. 

The tags section defines incident routing information if OpCon Job User defined tags are used to route tickets. This functionality is enabled when the includeTagRouting rule is set to true. 
The OpCon task tag definition can therefore be used to determine the routing of the ticket within the SymphonySummit environment. 

```
{
  "ticketDescription" : "OpCon Task Failure ( date @EV_Date schedule @EV_Schedule job @EV_Job server @EV_Agent error code @EV_errorcode )",
  "ticketInformation" : "Test ticket created from API. Please ignore!!",
  "address": {
    "name": "production",
    "value": "Symphony Summit Instance address"
  },
  "rules": {
    "includeJobLogAttachment": true,
    "includeTagRouting": true,
    "includeCategoryNameTag": true,
    "includeWorkGroupNameTag": false,
    "submitSingleIncidentPerDay": day
  },
  "credentials": {
    "apiKey": "encrypted key"
  },
  "urls": [
    {
      "name": "incident",
      "value": "api_integration/REST/Summit_RESTWCF.svc/RESTService/CommonWS_JsonObjCall_JSON"
    },
        {
      "name": "attachment",
      "value": "api_integration/REST/Summit_RESTWCF.svc/RESTService/Summit_UploadAttachmentBase64Encoded"
    }
  ],
  "workingHours": {
    "monday": {
      "start": "0800",
      "stop": "1900"
    },
    "tuesday": {
      "start": "0800",
      "stop": "1900"
    },
    "wednesday": {
      "start": "0800",
      "stop": "1900"
    },
    "thursday": {
      "start": "0800",
      "stop": "1900"
    },
    "friday": {
      "start": "0800",
      "stop": "1900"
    },
    "saturday": {
      "start": "0800",
      "stop": "1100"
    },
    "sunday": {
      "start": "0000",
      "stop": "0000"
    }
  },
  "attributes": [
    {
      "name": "Impact_Name",
      "value": "Low"
    }, {
      "name": "Urgency_Name",
      "value": "High"
    }
    
  ],
  "workingHoursAttributes": [],
  "nonWorkingHoursAttributes": [],
  "customAttributes": [
    {
      "groupName": "Other Details",
      "name": "Job Name",
      "value": "@EV_Job"
    },
    {
      "groupName": "Other Details",
      "name": "Abend",
      "value": "Yes"
    }
  ],
  "tags": [
    {
      "indicator": "TAG_END",
      "indicatorValue": "ROUTE1",
      "attribute": "Assigned_WorkGroup_Name",
      "value": "DevOps"
    },
    {
      "indicator": "TAG_START",
      "indicatorValue": "ROUTE2",
      "attribute": "Assigned_WorkGroup_Name",
      "value": "Operating SystemOrg2"
    },
    {
      "indicator" : "EXIT",
      "indicatorValue" : "NOTICKET",
      "attribute" : "",
      "value" : ""
    },
    {
      "indicator" : "CATNAME",
      "indicatorValue" : "CATNAME",
      "attribute" : "Category_Name",
      "value" : "testcatvalue"
    },
    {
      "indicator": "DEFAULT",
      "indicatorValue": "DEFAULT",
      "attribute": "Assigned_WorkGroup_Name",
      "value": "DevOps"
    }
  ]
}

```

### Customized Description and Information Definitions
The ticketDescription and ticketInformation attribute values can be customized by using specific place holders associated with the arguments passed to the connector. 

Place Holder  | Value
--------------| -----------------
@EV_Agent     | The name of teh agent that executed the failing task.
@EV_Date      | The date when the failure occurred (format yyyy-MM-dd).
@EV_errorcode | The job termination code.
@EV_Job       | The name of the job that failed.
@EV_Schedule  | The name of the schedule that contained the failed job.

### Customized Attribute, CustomAttribute Value definitions
The attribute values can be customized by using specific place holders associated with the arguments passed to the connector. 

Place Holder  | Value
--------------| -----------------
@EV_Agent     | The name of teh agent that executed the failing task.
@EV_Date      | The date when the failure occurred (format yyyy-MM-dd).
@EV_errorcode | The job termination code.
@EV_Job       | The name of the job that failed.
@EV_Schedule  | The name of the schedule that contained the failed job.

### Tag Routing
Requires that the rule **includeTagRouting** is enabled. 

It should be noted that the rules **includeTagRouting** and **includeWorkGroupNameTag** are mutually inclusive as the result is the addition of the Assigned_WorkGroup_name tag.
The precedence is to check if the rule **includeWorkGroupNameTag** is enabled (true). If the rule is not enabled, then a check is made to see if the **includeTagRouting** rule is enabled.
Tag routing allows OpCon tag names to be used to determine the Assigned_WorkGroup_Name attribute of the ticket. 

When using tag routing the OpCon tag includes the routing indicator and this is used to perform a match to a defined tag value in the template. 
The following tag indicators are supported

Indicator     | Description
------------- | --------------------------------------------------------
**EXIT**      | Performs a match of against the complete job tag. 
**TAG_START** | Indicates that the job tags will be checked for a matching value (**Indicator Value** field) from the start of each job tag in the list of tags.
**TAG_END**   | Indicates that the job tags will be checked for a matching value (**Indicator Value** field) from the end of each job tag in the list of tags.
**DEFAULT**   | Defines the attributes to used when there is no tag match. 
 
If the tag indicator is TAG_START, then a check is made to determine if there is an OpCon tag thats starts with the indicator value. If there is a 
match, the value associated with  the template tag definition will be assigned to the Assigned_WorkGroup_Name attribute of the ticket. If there is
no match, then the DEFAULT tag value will be used.
A special tag EXIT can be defined that can be used to not create a ticket if the job fails. In this case, a match is made between the OpCon tag name and the template EXIT tag indicator value.

Tag routing is defined using the **tags** structure.


Examples

```
  OpCon Tag : APP1_ROUTE1

  "tags": [
    {
      "indicator": "TAG_END",
      "indicatorValue": "ROUTE1",
      "attribute": "Assigned_WorkGroup_Name",
      "value": "Application One"
    },
    {
      "indicator": "DEFAULT",
      "indicatorValue": "DEFAULT",
      "attribute": "Assigned_WorkGroup_Name",
      "value": "DevOps"
    }
  ]

```
In the above example the ticket Assigned_WorkGroup_Name attribute will be assigned the value **Application One**.

```
  OpCon Tag : APP_ONE

  "tags": [
    {
      "indicator": "TAG_END",
      "indicatorValue": "ROUTE1",
      "attribute": "Assigned_WorkGroup_Name",
      "value": "Application One"
    },
    {
      "indicator": "DEFAULT",
      "indicatorValue": "DEFAULT",
      "attribute": "Assigned_WorkGroup_Name",
      "value": "DevOps"
    }
  ]

```
In the above example the ticket Assigned_WorkGroup_Name attribute will be assigned the value **DevOps**.


```
  OpCon Tag : APP1_ROUTE1, NOTICKET

  "tags": [
    {
      "indicator": "TAG_END",
      "indicatorValue": "ROUTE1",
      "attribute": "Assigned_WorkGroup_Name",
      "value": "Application One"
    {
      "indicator" : "EXIT",
      "indicatorValue" : "NOTICKET",
      "attribute" : "",
      "value" : ""
    },
      "indicator": "DEFAULT",
      "indicatorValue": "DEFAULT",
      "attribute": "Assigned_WorkGroup_Name",
      "value": "DevOps"
    }
  ]

```

In the above example no ticket will be created as the check for the EXIT is performed before the tag routing evaluation. 

### Assigning WorkGroup Names using OpCon tags
Requires the rule **includeWorkGroupNameTag** to be enabled.

It should be noted that the rules **includeTagRouting** and **includeWorkGroupNameTag** are mutually inclusive as the result is the addition of the Assigned_WorkGroup_name tag.
The precedence is to check if the rule **includeWorkGroupNameTag** is enabled (true). If the rule is not enabled, then a check is made to see if the **includeTagRouting** rule is enabled.
Tag routing allows OpCon tag names to be used to determine the Assigned_WorkGroup_Name attribute of the ticket. 

OpCon tag names can be used to determine the Assigned_WorkGroup_Name attribute of the ticket. 

When using work group naming by OpCon tags, an OpCon tag WRKGRP_workgroupname value must be used. The software will check OpCon tags for a tag that start with the WRKGRP prefix
and extract the work group name from the tag, setting the Assigned_WorkGroup_Name attribute value to the extracted value.

Indicator     | Description
------------- | --------------------------------------------------------
**WRKGRP**    | Defines the check value be made for work group name check. 
 
```
  OpCon Tag : APP1, WRKGRP_DevOps, TESTING

```
In the above example, DevOps will be extracted from the WRKGRP_ OpCon tag and will be assigned to the Assigned_WorkGroup_Name attribute value. 

### Category Names using OpCon tags
Requires the rule **includeCategoryNameTag** to be enabled.
OpCon tag names can be used to determine the Category_Name attribute of the ticket. 

When using category naming by OpCon tags, an OpCon tag CATNAME_categoryname value must be used. The software will check OpCon tags for a tag that start with the CATNAME prefix
and extract the category name from the tag, setting the Category_Name attribute value to thee xtracted value.

Indicator     | Description
------------- | --------------------------------------------------------
**CATNAME**   | Defines the check value be made for category name check. 
 
```
  OpCon Tag : APP1_ROUTE1, CATNAME_Elasticsearch

  "tags": [
    {
      "indicator": "TAG_END",
      "indicatorValue": "ROUTE1",
      "attribute": "Assigned_WorkGroup_Name",
      "value": "Application One"
    {
      "indicator" : "CATNAME",
      "indicatorValue" : "CATNAME",
      "attribute" : "Category_Name",
      "value" : ""
    },
      "indicator": "DEFAULT",
      "indicatorValue": "DEFAULT",
      "attribute": "Assigned_WorkGroup_Name",
      "value": "DevOps"
    }
  ]

```
In the above example, Elasticsearch will be extracted from the CATNAME_ OpCon tag and will be assigned to the Category_Name attribute value. 

### OpCon Notification Manager Definition
Notification Manager is used to execute the EasyVista Connector when a task completes with a failure condition. Using this approach allows the tasks to be added to the rule instead of defining a failure event on every task. 

#### Configuring Run Command
Using Notification Manager, select the Jobs tab and create a new Group called EasyVista.
Once the Group has been created, select the EasyVista Group, perform a ‘right-click’ and select Add Job Trigger. In the Add Job Trigger selection, select Job Failed.

In the Run Command tab, enter the following:

**Command**			       C:\Connectors\SymphonySummit\SMASyphonySummit.exe -a [[$MACHINE NAME]] -s [[$SCHEDULE NAME]] -jn [[$JOB NAME]] -e [[$JOB TERMINATION]] -sd [[$SCHEDULE DATE-SSUM]] -si [[$SCHEDULE ID]] -sn [[$SCHEDULE INST]] -t basic.json

                           Where 
                           C:\Connectors\EasyVista\EasyVista.exe 	is the location of the connector.
                           -a [[$MACHINE NAME]]		                resolves to the agent name
                           -s [[$SCHEDULE NAME]] 		              resolves to the schedule name
                           -jn [[$JOB NAME]]			                resolves to the job name
                           -e [[JOB TERMINATION]]		              resolves to the job termination code
	                        -sd [[$SCHEDULE DATE-SSUM]]	            resolves to the date (format YYYY-MM-DD)
	                        -si [[$SCHEDULE ID]]		                resolves to the schedule ID
	                        -sn [[$SCHEDULE INST]]		              resolves to the schedule instance
	                        -t basic.json		                        which template in the templates folder to use

**Working Directory**		C:\Connectors\SymphonySummit

**Batch User**			    Use Service Account 	                The batch User under which the task will be run.
