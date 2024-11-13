# Teams Audit with Azure Logic Apps

Start an audit process of all Teams in a M365 tenant. The solution sends out one email to each team owner for a permission review and a confirmation. 
The result is tracked and can be viewed in a Power BI report.
Reminder emails are sent out if the team owner has not confirmed the teams review. The solution is built with Azure Logic Apps.

## config

All Logic Apps are using a central configuration. The configuration is stored in a table in a storage account. The configuration is read by the Logic Apps and used in the Logic Apps.

### 11 config

~~~
{ 
    "reminderInterval":  "30",  
    "remindersCount": "2", 
    "tableCampaigns": "s11Campaigns", 
    "tableOwners": "s11Owners", 
    "tableRecipients": "s11Recipients",  
    "tableTeams": "s11Teams", 
    "submitActionUrl": "<submiturl>",
    "flow1Url": "<flow1url>", 
    "flow2Url": "<flow2url>", 
    "adminEmail": "<email@email.com;email2@email.com>", 
    "showMembers": false, 
    "debug": true 
}
~~~

| Key               | Description                                                                                       |
|-------------------|---------------------------------------------------------------------------------------------------|
| reminderInterval  | If the team owner not checked the guest list, a reminder is sent after the specified number of days. |
| remindersCount    | The times of reminders sent to the team owner.                                                    |
| tableCampaigns    | Name of the table where the campaign data is stored.                                              |
| tableOwners       | Name of the table where the team owners are stored.                                               |
| tableRecipients   | Name of the table where the recipients are stored.                                                |
| tableTeams        | Name of the table where the teams with inactive users are stored.                                 |
| submitActionUrl   | Endpoint for the email check & confirm action.                                                    |
| flow1Url          | Trigger endpoint for the run teams flow.                                                          |
| flow2Url          | Trigger endpoint for the send emails flow.                                                        |
| adminEmail        | All admin emails as control & fallback in the format of "email@email.com;email2@email.com".       |
| showMembers       | If true, the members of the team are shown in the email.                                          |
| debug             | If debug is true, emails are sent to the adminEmail address instead of sending to the teams owners.|

### app config

~~~

{  
    "tenantId": "<tenantId>",
    "clientId": "<clientId>"
}
~~~

### App Registration

For some operations, an app registration is needed with the following permissions:  
- User.ReadWrite.All
- Group.ReadWrite.All
- ... extend as needed for other operations like sending emails, etc.

### Key Vault

The client secret is stored in the key vault. The key vault is referenced in the Logic Apps connection.

keyvault config
~~~

secret: <clientSecret>
~~~
The client secret from the app registration.

## Flows

The flows include the business logic for this scenario, as follows.

### 11-01-ReportTeamsAudit-NewCampaign

This flow creates a new campaign. The campaign name is named as "2024-1" - per year. The admins get a confirmation about the started campaign.

### 11-02-ReportTeamsAudit-RunTeams

This flow creates the entities to track the check & confirmation status of the team and the same for the owners individually to track their progression.
The first owner's confirmation will be saved as the main confirmation for the team.

### 11-03-ReportTeamsAudit-Send

This flow will send out the emails to the team owners with the guest list and the check & confirm action link.
The flow will also send out the reminder emails if the owner has not checked the guest list.

### 11-04-ReportTeamsAudit-Request

This flow handles the check & confirmation actions. The check will open the team in the browser and updates the table.
The confirmation will check if the team has already been checked. If not, it updates the table.
The request id can not be manipulated, so the confirmation can not be faked.
There is always a response page with the relevant data.

### 11-05-ReportTeamsAudit-Reminder

This flow takes care of the reminders. It will check the next reminder date, updates the next reminder date if the next reminder date is today and if the reminder count has not reached the max count and triggers the third flow.

## Deployment

You can deploy this solution with the provided ARM templates in the [Deploy](./Deploy) folder. The ARM template will deploy all resources and the Logic Apps. 

## Reporting

Download the Power BI template file, and connected to the storage account. The Power BI file will show the status of the teams and the owners.
