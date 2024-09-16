# Operation

Once the SymphonySummit connector has been installed and configured, there is very little operational requirements.

If tag routing (**includeTagRouting**), Workgroup Naming (**includeWorkGroupNameTag**) or Category Naming (**includeCategoryNamingTag**) rules are enabled, then OpCon tags should be defined for each job.

It should be noted that the rules **includeTagRouting** and **includeWorkGroupNameTag** are mutually inclusive as the result is the addition of the Assigned_WorkGroup_name tag.
The precedence is to check if the rule **includeWorkGroupNameTag** is enabled (true). If the rule is not enabled, then a check is made to see if the **includeTagRouting** rule is enabled.
Tag routing allows OpCon tag names to be used to determine the Assigned_WorkGroup_Name attribute of the ticket. 

The Tag Manager can be used to simplify the task.  

The rule **submitSingleIncidentPerDay** rule can be used to prevent multiple incidents being created for the same task on the calendar day. When this rule is enabled, the failed calendar date is
written to the job documentation field of the task. If documentation values exist for the task, the date is written as a prefix to the existing documentation in a separate line at the beginning.
(i.e. Last Incident Date : 2024-09-21).

When checking the last incident creation, this date is used and the configuration value DAILY_START_HOUR is added to this value. The current date and time is then checked against this value to 
determine if a new incident should be created.


