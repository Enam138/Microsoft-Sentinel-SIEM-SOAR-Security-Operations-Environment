# Troubleshooting and Technical Challenges

## 1. Overview

Building the Microsoft Sentinel SIEM/SOAR environment involved several technical challenges across telemetry ingestion, KQL, Workbooks, Logic Apps, Azure RBAC, and Microsoft Entra licensing.

Rather than treating these issues as failed parts of the project, I used them as opportunities to troubleshoot the environment systematically.

The major challenges encountered were:

```text
1. No telemetry returned within the initial 24-hour query window

2. Key Vault KQL filtering initially returned no expected records

3. Azure Key Vault Security Workbook displayed configuration errors

4. Logic App managed identity initially had 0 Azure role assignments

5. Role eligibility required Microsoft Entra ID P2 or Governance

6. Additional MDTI permission requirements could not be fully configured

7. Full playbook testing was limited by the absence of an applicable live incident

8. Distinguishing successful trigger execution from complete MDTI workflow validation
```

The troubleshooting approach used throughout the project was:

```text
Observe
   |
   v
Validate
   |
   v
Isolate
   |
   v
Correct
   |
   v
Retest
   |
   v
Document
```

# 2. Troubleshooting Principles

Several principles guided the troubleshooting process.

## Validate Before Changing Configuration

Before assuming that a service was broken, I first checked:

```text
Time range

Telemetry availability

Query syntax

Resource parameters

Permissions

Connections

Licensing
```

## Change One Variable at a Time

Where possible, I avoided changing several settings simultaneously.

This made it easier to determine which change resolved the problem.

## Separate Configuration from Validation

A resource being successfully deployed does not automatically mean it is operational.

For example:

```text
Logic App Deployed
        !=
Logic App Fully Authorized
```

Similarly:

```text
Workbook Saved
        !=
Workbook Receiving Data
```

## Document Environment Limitations

Where functionality depended on unavailable licensing or permissions, I documented the limitation rather than trying to represent the feature as fully implemented.


# 3. Issue 1 — No Telemetry Returned

## Problem

During the initial Log Analytics investigation, I attempted to identify which tables contained data.

I used:

```kusto
search *
| summarize EventCount = count() by $table
| order by EventCount desc
```

The selected timeframe was:

```text
Last 24 hours
```

The query did not return the useful telemetry expected for the investigation.


# 4. Initial Assumption

A zero-result workspace query could potentially indicate:

```text
Diagnostic settings not configured

Logs not reaching Log Analytics

Incorrect workspace

Resource not generating activity

Telemetry ingestion delay

Incorrect time range
```

Rather than immediately changing diagnostic settings, I first tested the simplest variable:

```text
Time Range
```

---

# 5. Resolution

I changed the query timeframe from:

```text
Last 24 hours
```

to:

```text
Last 7 days
```

and reran the query.

The workspace returned:

```text
AzureMetrics       24
Usage               7
AzureDiagnostics    4
```

This confirmed that telemetry existed.


# 6. Root Cause

The issue was not a complete telemetry-ingestion failure.

The useful records were outside the initial 24-hour investigation window.

The actual troubleshooting path was:

```text
No Results
    |
    v
Check Workspace
    |
    v
Expand Time Range
    |
    v
7 Days
    |
    v
Telemetry Found
```

# 7. Lesson Learned

A zero-result query should not immediately be interpreted as a broken connector or logging pipeline.

Before modifying infrastructure, verify:

```text
Time range

Filters

Table

Resource activity

Ingestion timing
```

This avoided unnecessary configuration changes.

# 8. Issue 2 — Key Vault Filter Returned No Expected Records

## Problem

After confirming that `AzureDiagnostics` contained data, I attempted to filter the results specifically for Azure Key Vault activity.

The expected Key Vault telemetry was visible when inspecting the table directly.

However, an early filtering attempt did not return the expected records.

# 9. Telemetry Inspection

I first inspected the table without the restrictive filter:

```kusto
AzureDiagnostics
| order by TimeGenerated desc
| take 50
```

This revealed records containing:

```text
ResourceProvider:
MICROSOFT.KEYVAULT
```

This was important because it proved:

```text
The telemetry existed.
```

The problem therefore shifted from:

```text
Data ingestion
```

to:

```text
Query filtering
```


# 10. Resolution

I changed the comparison to use the case-insensitive KQL operator:

```kusto
=~
```

The final filter became:

```kusto
| where ResourceProvider =~ "MICROSOFT.KEYVAULT"
```

The complete query was:

```kusto
AzureDiagnostics
| where ResourceProvider =~ "MICROSOFT.KEYVAULT"
| project
    TimeGenerated,
    ResourceGroup,
    ResourceProvider,
    Resource,
    ResourceType,
    OperationName,
    ResultType
| order by TimeGenerated desc
```

The query then returned:

```text
4 Key Vault events
```

# 11. Root Cause

The issue was related to string comparison in the KQL filter rather than missing Key Vault telemetry.

The troubleshooting process was:

```text
Filtered Query Returns Nothing
           |
           v
Remove Filter
           |
           v
Inspect Raw Field Values
           |
           v
Confirm ResourceProvider Value
           |
           v
Use Case-Insensitive Comparison
           |
           v
Expected Results Returned
```


# 12. Lesson Learned

When a filtered KQL query returns no results but the table contains data:

```text
Do not immediately assume the data source is broken.
```

Inspect the actual field values first.

Useful troubleshooting techniques include:

```kusto
AzureDiagnostics
| take 50
```

and:

```kusto
AzureDiagnostics
| summarize count() by ResourceProvider
```

This helps identify the exact values that should be used in subsequent filters.


# 13. Issue 3 — Zero Results from Failed-Operation Query

## Problem

The failed Key Vault operations query returned:

```text
0 results
```

The query was:

```kusto
AzureDiagnostics
| where ResourceProvider =~ "MICROSOFT.KEYVAULT"
| where ResultType !in~ ("Success", "Succeeded")
| project
    TimeGenerated,
    ResourceGroup,
    Resource,
    OperationName,
    ResultType
| order by TimeGenerated desc
```

At first glance, a zero-result query could be mistaken for a query problem.


# 14. Validation

The broader Key Vault activity query successfully returned:

```text
4 events
```

Those events showed successful results.

Therefore, the failed-operation query returning zero matches was consistent with the available telemetry.


# 15. Resolution

No query correction was required.

The correct interpretation was:

> No Key Vault events matching the unsuccessful-operation condition were found within the available telemetry and selected investigation period.


# 16. Lesson Learned

There is an important difference between:

```text
Query failed
```

and:

```text
Query executed successfully and returned zero matches
```

In threat hunting, zero-result queries can still provide useful evidence.


# 17. Issue 4 — Azure Key Vault Security Workbook Errors

## Problem

I opened the:

```text
Azure Key Vault Security
```

workbook template.

The workbook initially displayed errors and some visualizations did not populate.

The errors indicated that required parameters could not be resolved correctly.


# 18. Investigation

Because Key Vault telemetry had already been confirmed through KQL, the issue was unlikely to be a complete absence of data.

I therefore investigated the workbook configuration.

The workbook required environment-specific values for parameters such as:

```text
Workspace

KeyVault
```

These had not yet been configured correctly.


# 19. Resolution

I configured:

```text
Workspace:
law-contoso-prod-001

KeyVault:
kv-contoso-prod-001
```

I then refreshed the workbook.

The visualizations populated successfully.


# 20. Root Cause

The workbook template existed, but it did not yet know which environment resources it should query.

The troubleshooting sequence was:

```text
Workbook Opened
      |
      v
Visualization Errors
      |
      v
Check Telemetry
      |
      v
Telemetry Already Confirmed
      |
      v
Inspect Workbook Parameters
      |
      v
Workspace Missing / Incorrect
      |
      v
Key Vault Missing / Incorrect
      |
      v
Configure Parameters
      |
      v
Refresh
      |
      v
Dashboard Populated
```


# 21. Lesson Learned

A prebuilt Microsoft Sentinel workbook is not automatically configured for a specific environment.

The implementation stages are different:

```text
Template Available

Template Opened

Parameters Configured

Queries Working

Visualizations Populated

Workbook Saved
```

Validation should reach the visualization stage before the workbook is considered operational.


# 22. Issue 5 — No Existing Sentinel Automation Rules

## Problem

When I opened:

```text
Microsoft Sentinel
→ Automation
```

the environment showed:

```text
0 existing rules
```

This was not an error.

It indicated that no automation rules had yet been created.


# 23. Resolution

I created:

```text
Contoso Sentinel Incident Triage
```

using:

```text
Trigger:
When incident is created
```

The rule was configured with:

```text
Add tag:
Automated-Triage

Add task:
Perform Initial SOC Investigation
```


# 24. Lesson Learned

An empty Sentinel feature page does not necessarily indicate a configuration problem.

It may simply represent:

```text
No user-created resources yet
```

The same distinction applied when the Playbooks environment initially showed:

```text
0 active playbooks
```


# 25. Issue 6 — Automation Rule Create Button Initially Inactive

## Problem

During native automation-rule creation, the:

```text
Create
```

button remained inactive until required fields were completed.

The interface required more than just a rule name.

Required configuration included a trigger and action information.


# 26. Available Triggers

The interface presented:

```text
When incident is created

When incident is updated

When alert is created
```

I selected:

```text
When incident is created
```

Additional action configuration was then required.


# 27. Available Actions

The action field provided options including:

```text
Run Logic App Playbook

Change Status

Change Severity

Assign Owner

Add Tags

Add Task
```

After the required rule configuration was completed, the rule could be created.

# 28. Lesson Learned

When an Azure portal action button remains unavailable, check for:

```text
Required fields

Unselected dropdown values

Missing trigger configuration

Missing action configuration

Validation warnings
```

before assuming there is a portal or permission problem.


# 29. Issue 7 — No Active Playbooks

## Problem

The Sentinel Playbooks environment initially displayed:

```text
0 active playbooks
```

Available options included:

```text
Create Playbook

Playbook Templates
```

# 30. Investigation

I reviewed the available playbook templates.

Visible MDTI-related templates included:

```text
MDTI-Automated-Triage

MDTI-Intel-Reputation

MDTI-Data-Cookies

MDTI-Data-Trackers

MDTI-Data-ReverseDns

MDTI-Data-PassiveDns

MDTI-Data-WebComp...
```

# 31. Resolution

I selected:

```text
MDTI-Automated-Triage
```

and deployed the playbook.

The Azure deployment completed successfully.

The environment therefore moved from:

```text
0 active playbooks
```

to having a deployed Logic App security workflow.


# 32. Lesson Learned

As with automation rules, an empty Playbooks list was not a platform failure.

The required workflow simply had not yet been deployed.


# 33. Issue 8 — Understanding the Sentinel Logic App Connection

## Problem

After deployment, I needed to determine whether the playbook was connected to Microsoft Sentinel.

I inspected the Logic App's connection information.

The connection displayed:

```text
Microsoft Sentinel
```

# 34. Validation

This confirmed that a Sentinel API connection existed for the workflow.

However, I did not treat the existence of the connection as proof that every workflow operation was authorized.

The next step was therefore to investigate identity and permissions.


# 35. Lesson Learned

A connection answers one question:

```text
Can the workflow connect to the service?
```

It does not necessarily answer:

```text
Does the workflow identity have every required permission?
```

Connection configuration and authorization must be validated separately.


# 36. Issue 9 — Managed Identity Had Zero Role Assignments

## Problem

The Logic App had:

```text
System Assigned Identity:
On
```

However, the Azure role-assignment section showed:

```text
0 role assignments
```

This meant the Logic App had an identity but no Azure RBAC role assigned through that view.

# 37. Security Interpretation

The situation can be represented as:

```text
Managed Identity
      |
      v
Authentication Identity Exists
      |
      X
      |
No Azure RBAC Authorization Yet
```

The identity itself did not automatically grant Sentinel permissions.

# 38. Resolution

I assigned:

```text
Microsoft Sentinel Contributor
```

to the Logic App's system-assigned managed identity.

The role assignment completed successfully.

The authorization path became:

```text
MDTI-Automated-Triage
        |
        v
System Assigned Identity
        |
        v
Microsoft Sentinel Contributor
        |
        v
Sentinel Resources
```

# 39. Validation

After the role assignment, the managed identity had an explicit Sentinel-related Azure RBAC role.

### Result

```text
Microsoft Sentinel Contributor:
Successfully assigned
```

# 40. Lesson Learned

Managed identity and RBAC solve different problems.

```text
Managed Identity
      |
      v
Authentication
```

while:

```text
Azure RBAC
      |
      v
Authorization
```

A secure Azure automation workflow often requires both.

# 41. Issue 10 — Microsoft Entra Role Eligibility Licensing

## Problem

While reviewing role eligibility, the Azure interface displayed a requirement for:

```text
Microsoft Entra ID P2

or

Microsoft Entra ID Governance
```

The tenant did not have the required licensing.

# 42. Investigation

The message related specifically to role eligibility functionality.

It did not mean that every Azure RBAC role assignment required Entra ID P2.

This distinction was important.

# 43. Resolution

I used the standard Azure role-assignment capability available in the environment and successfully assigned:

```text
Microsoft Sentinel Contributor
```

The advanced role-eligibility functionality remained unavailable.


# 44. Root Cause

The limitation was:

```text
Licensing
```

rather than:

```text
Broken RBAC
```

or:

```text
Logic App deployment failure
```

# 45. Lesson Learned

When Azure reports that a feature requires a higher license, determine exactly which capability is affected.

Do not assume that every related feature is unavailable.

In this case:

```text
Role Eligibility:
Unavailable due to licensing

Standard RBAC Assignment:
Available and successfully configured
```


# 46. Issue 11 — ThreatIntelligence.Read.All Requirement

## Problem

During advanced MDTI playbook configuration, an additional permission requirement related to:

```text
ThreatIntelligence.Read.All
```

was encountered.

The required permission could not be located or configured as expected within the available environment.

# 47. Investigation

The playbook relied on more than Sentinel RBAC.

Advanced threat-intelligence enrichment may involve additional API permissions and service requirements.

The complete authorization chain can include:

```text
Logic App
    |
    v
Sentinel Connection
    |
    v
Managed Identity
    |
    v
Azure RBAC
    |
    v
Additional API Permissions
    |
    v
Threat Intelligence Service
```

The Sentinel Contributor role therefore did not automatically solve every downstream MDTI permission requirement.

# 48. Resolution

The limitation was documented rather than bypassed with unnecessarily broad permissions.

The implementation continued with the parts of the workflow that could be safely validated.

The final state was classified as:

```text
Full MDTI Enrichment:
Partially Validated
```

# 49. Lesson Learned

Cloud automation often crosses multiple authorization models.

A workflow may require combinations of:

```text
Azure RBAC

Managed Identity

API Connections

Microsoft Graph Permissions

Service-Specific Permissions

Licensing
```

Successfully configuring one authorization layer does not automatically satisfy the others.


# 50. Issue 12 — No Applicable Active Sentinel Incident

## Problem

A complete SOAR workflow is ideally validated using an actual Sentinel incident.

During the primary validation period, there was no applicable active incident available for a full real-world triage workflow.


# 51. Why I Did Not Manufacture a Security Finding

Creating an artificial incident solely to claim end-to-end success would weaken the integrity of the validation.

The project therefore distinguished between:

```text
Configured Incident Automation
```

and:

```text
Observed Real Incident Automation
```

The native automation rule was documented as:

```text
Configured
```

rather than fully exercised against a live security incident.


# 52. Alternative Validation

Although a full live incident was unavailable, the Logic App trigger itself could still be tested.

The trigger execution returned:

```text
Successfully ran the trigger
```

This provided evidence that the Sentinel trigger layer was operational.

# 53. Lesson Learned

Security validation should clearly state the boundary of the test.

For example:

```text
Trigger Tested:
Yes

Real Malicious Incident Processed:
No

Full MDTI Enrichment Confirmed:
No
```

This is more useful than simply labeling the entire workflow as successful.


# 54. Issue 13 — Interpreting Successful Trigger Execution

## Problem

The Logic App produced:

```text
Successfully ran the trigger
```

It would have been easy to document this as:

```text
The entire playbook successfully completed.
```

That would not have been technically accurate.


# 55. Correct Interpretation

The successful trigger confirmed:

```text
Logic App deployed

Workflow accessible

Microsoft Sentinel trigger available

Trigger execution successful
```

It did not automatically confirm:

```text
Every entity enrichment action completed

Every MDTI API call succeeded

Every permission requirement was satisfied

A real malicious incident was processed
```

# 56. Lesson Learned

SOAR workflows should be validated in layers.

A useful model is:

```text
Deployment
    |
    v
Connection
    |
    v
Identity
    |
    v
Authorization
    |
    v
Trigger
    |
    v
Individual Actions
    |
    v
End-to-End Workflow
```

Each layer should be tested independently where possible.


# 57. Issue 14 — Limited Key Vault Dataset

## Problem

Only:

```text
4 Key Vault events
```

were available during the investigation period.

This limited the depth of behavioral analysis that could be performed.

# 58. Impact

The limited dataset affected:

```text
Baseline development

Anomaly analysis

Frequency analysis

Trend analysis

Failed-operation investigation

Identity correlation
```

The activity-frequency query worked, but the dataset was not large enough to support strong statistical conclusions.

# 59. Resolution

Rather than inventing additional findings, the limitation was documented.

The frequency analysis was described as:

```text
A basic summary of the available telemetry
```

rather than:

```text
A production behavioral baseline
```

# 60. Lesson Learned

Security analytics should not make conclusions stronger than the underlying dataset supports.

```text
Small Dataset
     |
     v
Limited Confidence
     |
     v
Careful Interpretation
```

This is particularly important in portfolio and lab environments.


# 61. Issue 15 — RDP and SSH Analytics Templates Without Supporting Workloads

## Problem

The available analytics templates included:

```text
Anomalous RDP Login Detection

Anomalous SSH Login Detection
```

However, the project did not contain the required RDP or SSH workloads and telemetry to meaningfully support those detections.


# 62. Decision

I did not enable the templates simply to increase the number of active analytics rules.

The reasoning was:

```text
Analytics Template
       |
       v
Does Relevant Workload Exist?
       |
      No
       |
       v
Required Telemetry Missing
       |
       v
Do Not Enable for Appearance
```


# 63. Lesson Learned

Detection engineering should prioritize:

```text
Relevant Coverage
```

over:

```text
Rule Count
```

An enabled rule without its required telemetry may provide little practical security value.


# 64. Issue 16 — No Alerts or Incidents from the Enabled Analytics Rule

## Problem

The analytics rule was successfully enabled, but the environment did not generate an applicable alert or incident during the validation period.


# 65. Interpretation

This did not mean:

```text
The analytics rule failed.
```

It meant that no matching detection event was observed during the available validation period.

The project therefore separated:

```text
Rule Enabled:
Validated

Detection Match:
Not Observed

Alert Generated:
Not Observed

Incident Generated:
Not Observed
```


# 66. Lesson Learned

An enabled detection should not be judged solely by whether it immediately generates an alert.

Detection testing ideally requires:

```text
Known test activity

Expected telemetry

Known detection condition

Observed alert

Expected incident behavior
```

That level of controlled detection testing was outside the available project environment.


# 67. Troubleshooting Decision Tree

The troubleshooting experience from the project can be summarized using the following decision tree:

```text
Security Feature Not Working as Expected
                 |
                 v
         Is Telemetry Present?
           /           \
         No             Yes
         |               |
         v               v
Check Diagnostics     Check Query
Connectors            Filters
Time Range            Parameters
         |               |
         +-------+-------+
                 |
                 v
          Is Identity Required?
           /           \
         Yes            No
         |               |
         v               |
Check Managed Identity   |
         |               |
         v               |
       Check RBAC        |
         |               |
         v               |
Check API Permissions    |
         |               |
         v               |
   Check Licensing       |
         |               |
         +-------+-------+
                 |
                 v
              Retest
                 |
                 v
         Validate Result
                 |
                 v
             Document
```


# 68. Common KQL Troubleshooting Checklist

When a Sentinel query returns unexpected results:

```text
[ ] Verify the selected time range

[ ] Confirm the table contains data

[ ] Remove filters and inspect raw records

[ ] Confirm exact field names

[ ] Inspect actual field values

[ ] Check string case behavior

[ ] Test one filter at a time

[ ] Verify the resource name

[ ] Check the resource provider

[ ] Confirm the expected telemetry category

[ ] Determine whether zero results are actually valid
```


# 69. Workbook Troubleshooting Checklist

When a workbook does not populate:

```text
[ ] Confirm underlying telemetry exists

[ ] Check the selected subscription

[ ] Check the Log Analytics workspace

[ ] Check resource parameters

[ ] Check the workbook time range

[ ] Refresh the workbook

[ ] Inspect failed visualization queries

[ ] Verify required diagnostic logs

[ ] Confirm access to the underlying workspace
```


# 70. Logic App Troubleshooting Checklist

When a Sentinel playbook does not behave as expected:

```text
[ ] Confirm deployment succeeded

[ ] Open the Logic App designer

[ ] Verify the Sentinel trigger

[ ] Review API connections

[ ] Check managed identity status

[ ] Review Azure RBAC assignments

[ ] Confirm role scope

[ ] Review additional API permissions

[ ] Check licensing requirements

[ ] Test the trigger

[ ] Inspect run history

[ ] Review individual failed actions

[ ] Distinguish trigger success from workflow success
```


# 71. RBAC Troubleshooting Checklist

When an Azure resource cannot perform an expected action:

```text
[ ] Determine which identity performs the action

[ ] Confirm the identity exists

[ ] Check system-assigned/user-assigned identity status

[ ] Review current role assignments

[ ] Identify the required role

[ ] Check assignment scope

[ ] Apply least privilege

[ ] Allow time for permission propagation where necessary

[ ] Retest

[ ] Check whether additional API permissions are required
```


# 72. Licensing Troubleshooting Checklist

When Azure reports a licensing limitation:

```text
[ ] Identify the exact feature affected

[ ] Determine whether the limitation blocks the entire workflow

[ ] Check whether an alternative supported capability exists

[ ] Avoid unnecessary license assumptions

[ ] Document unavailable functionality

[ ] Continue validating unaffected components
```

This approach was used when Entra role eligibility required a higher license.

# 73. Troubleshooting Summary Table

| Issue | Root Cause / Finding | Resolution | Final Status |
|---|---|---|---|
| No telemetry in 24 hours | Investigation window too narrow | Expanded to 7 days | Resolved |
| Key Vault filter returned no expected data | String comparison/filtering | Used `=~` and validated raw values | Resolved |
| Failed-operation query returned 0 | No matching failed events | Correctly interpreted result | Valid |
| Key Vault workbook errors | Parameters not configured | Selected workspace and Key Vault | Resolved |
| 0 automation rules | No rules created yet | Created native triage rule | Resolved |
| Create button inactive | Required fields incomplete | Completed trigger/actions | Resolved |
| 0 active playbooks | No playbook deployed | Deployed MDTI playbook | Resolved |
| Sentinel connection verification | Needed connection validation | Confirmed Sentinel connection | Validated |
| Managed identity had 0 roles | No RBAC assignment | Added Sentinel Contributor | Resolved |
| Role eligibility unavailable | Entra licensing requirement | Used standard RBAC assignment | Limited |
| MDTI permission requirement | Additional authorization required | Documented limitation | Partial |
| No active incident | No applicable incident generated | Trigger-level validation used | Limited |
| Trigger success interpretation | Trigger != full workflow | Documented validation boundary | Resolved |
| Small telemetry dataset | Lab environment | Limited conclusions appropriately | Accepted |
| RDP/SSH templates | No supporting workloads | Not enabled | Intentional |
| No analytics-generated incident | No matching event observed | Documented accurately | Accepted |


# 74. Troubleshooting Evidence

Screenshots supporting troubleshooting can be stored under:

```text
images/
└── troubleshooting/
    ├── no-data-24-hours.png
    ├── telemetry-seven-days.png
    ├── key-vault-query-results.png
    ├── workbook-parameter-error.png
    ├── workbook-fixed.png
    ├── zero-role-assignments.png
    ├── entra-license-limitation.png
    ├── sentinel-contributor-assigned.png
    └── successful-playbook-trigger.png
```

Only screenshots actually captured during implementation should be included.

Sensitive information should be reviewed before screenshots are committed to a public repository.


# 75. Security Considerations When Troubleshooting

Troubleshooting should not weaken the environment simply to make a feature work.

I avoided approaches such as:

```text
Assigning Owner unnecessarily

Granting broad subscription permissions without justification

Disabling security controls to eliminate errors

Creating false security findings

Claiming unsupported detection results
```

Instead, the troubleshooting approach prioritized:

```text
Least Privilege

Evidence

Controlled Changes

Accurate Validation

Transparent Documentation
```

# 76. What I Would Improve in a Production Environment

A production troubleshooting process would benefit from:

```text
Centralized diagnostic logging

Logic App failure alerts

Automation health monitoring

Connector health monitoring

Detection health dashboards

Documented escalation procedures

Formal change management

Infrastructure-as-Code deployments

Version-controlled KQL

Version-controlled playbooks

Automated validation tests

Permission reviews

Runbooks for common failures
```

This would make troubleshooting more repeatable across a SOC team.

# 77. Skills Demonstrated Through Troubleshooting

The challenges encountered during the project demonstrated practical experience with:

```text
Microsoft Sentinel troubleshooting

KQL debugging

Log Analytics investigation

Telemetry validation

AzureDiagnostics

Azure Workbooks

Azure Logic Apps

Managed Identity

Azure RBAC

Microsoft Entra licensing

SOAR troubleshooting

Security permission analysis

Detection validation

Incident workflow analysis
```

The troubleshooting process also required distinguishing between:

```text
Technical Failure

Configuration Gap

Expected Zero Result

Permission Limitation

Licensing Limitation

Telemetry Limitation
```

These require different responses.


# 78. Key Lessons

The most important lessons from troubleshooting the environment were:

## Start With the Data

Before changing detection logic, verify that the expected telemetry actually exists.

## Time Range Can Completely Change an Investigation

Moving from 24 hours to seven days revealed the telemetry required for the project.

## Inspect Raw Data Before Writing Complex Filters

The raw `AzureDiagnostics` records helped identify the correct Key Vault field values.

## Zero Results Are Not Automatically Errors

The failed-operation query successfully returned zero matching events.

## Templates Still Require Configuration

The Key Vault workbook did not work correctly until its environment-specific parameters were configured.

## Identity Does Not Equal Authorization

The Logic App's managed identity existed while still having zero Azure role assignments.

## RBAC Does Not Solve Every API Permission

The Sentinel Contributor assignment addressed Sentinel authorization but did not automatically provide every MDTI permission.

## Licensing Limitations Should Be Isolated

The Entra licensing limitation affected role eligibility rather than all Azure RBAC functionality.

## Test Automation in Layers

Deployment, connection, identity, RBAC, trigger execution, and complete workflow execution are separate validation stages.

## Do Not Force Successful Outcomes

A security project is stronger when limitations are documented accurately instead of hidden.


# 79. Final Troubleshooting Outcome

The overall troubleshooting process can be summarized as:

```text
No Data
   |
   v
Expand Time Range
   |
   v
Telemetry Found
   |
   v
KQL Filter Problem
   |
   v
Inspect Raw Data
   |
   v
Correct Comparison
   |
   v
Queries Working
   |
   v
Workbook Errors
   |
   v
Configure Parameters
   |
   v
Workbook Working
   |
   v
Deploy Playbook
   |
   v
Inspect Identity
   |
   v
0 RBAC Assignments
   |
   v
Assign Sentinel Contributor
   |
   v
Review Additional Permissions
   |
   v
Document Licensing / MDTI Limitations
   |
   v
Test Sentinel Trigger
   |
   v
Successful Trigger Execution
```

The troubleshooting work was an important part of the project because it demonstrated that implementing Microsoft Sentinel is not simply a sequence of successful portal clicks.

A working SIEM/SOAR environment requires understanding the relationships between:

```text
Telemetry

Queries

Time Ranges

Detection Logic

Workbooks

Connections

Identity

RBAC

API Permissions

Licensing

Automation
```

By troubleshooting each layer separately, I was able to identify what was actually failing, what was simply unconfigured, what was limited by the environment, and what was working correctly.


## Related Documentation

- [Architecture](Architecture.md)
- [Deployment Guide](DeploymentGuide.md)
- [Data Connectors](DataConnectors.md)
- [KQL Queries](KQLQueries.md)
- [Analytics Rules](AnalyticsRules.md)
- [Incident Management](IncidentManagement.md)
- [Threat Hunting](ThreatHunting.md)
- [MITRE ATT&CK Mapping](MITRE-ATTACK-Mapping.md)
- [Workbooks](Workbooks.md)
- [SOAR Automation](SOAR-Automation.md)
- [Validation Report](ValidationReport.md)
