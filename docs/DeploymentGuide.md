# Microsoft Sentinel SIEM & SOAR Deployment Guide

## 1. Overview

This document describes the implementation process I followed to deploy and configure the Microsoft Sentinel SIEM and SOAR environment used in this project.

The deployment was built around an existing Azure environment rather than creating unnecessary infrastructure specifically for the lab.

The primary Azure resources used were:

| Resource | Value |
|---|---|
| Resource Group | `rg-contoso-prod-001` |
| Log Analytics Workspace | `law-contoso-prod-001` |
| Azure Key Vault | `kv-contoso-prod-001` |
| SIEM/SOAR Platform | Microsoft Sentinel |
| Security Operations Portal | Microsoft Defender |
| SOAR Platform | Azure Logic Apps |
| Playbook | `MDTI-Automated-Triage` |

The implementation progressed through the following stages:

```text
Existing Azure Environment
        |
        v
Microsoft Sentinel Onboarding
        |
        v
Data Integration
        |
        v
Content Hub
        |
        v
Telemetry Validation
        |
        v
KQL Analysis
        |
        v
Analytics Rule
        |
        v
Threat Hunting
        |
        v
MITRE ATT&CK
        |
        v
Workbooks
        |
        v
Native Automation
        |
        v
Logic App Playbook
        |
        v
Managed Identity & RBAC
        |
        v
SOAR Validation
```

# 2. Prerequisites

Before beginning the Sentinel deployment, the Azure environment already contained the resources required to support the project.

The most important prerequisite was the existing Log Analytics workspace:

```text
law-contoso-prod-001
```

I also had an existing Azure Key Vault:

```text
kv-contoso-prod-001
```

The Key Vault became an important telemetry source later in the project because diagnostic events from the resource were available within Log Analytics.

The project did not require a virtual machine, and I did not deploy one simply to match a generic Sentinel architecture.

# 3. Onboarding the Log Analytics Workspace to Microsoft Sentinel

The first implementation step was enabling Microsoft Sentinel against the existing Log Analytics workspace.

I opened Microsoft Sentinel in the Azure environment and selected the option to add Microsoft Sentinel to a workspace.

Rather than creating another workspace, I selected:

```text
law-contoso-prod-001
```

After completing the onboarding process, Microsoft Sentinel was enabled for the workspace.

This established the relationship:

```text
law-contoso-prod-001
        |
        v
Microsoft Sentinel
```

The existing Log Analytics workspace remained the underlying telemetry store while Sentinel provided the security operations capabilities layered on top of it.

These included:

- Analytics
- Incidents
- Hunting
- MITRE ATT&CK
- Workbooks
- Automation
- Content Hub
- Data Connectors

# 4. Working with the Microsoft Defender Portal

During the project, several Microsoft Sentinel capabilities were presented through the Microsoft Defender portal rather than only through the traditional Azure Sentinel interface.

I therefore worked across both:

```text
Azure Portal
     +
Microsoft Defender Portal
```

The Azure portal remained useful for:

- Log Analytics
- Azure resource configuration
- Logic Apps
- Managed identities
- Azure RBAC

The Microsoft Defender portal was used for several Sentinel security operations functions, including:

- Analytics
- Incidents
- Hunting
- MITRE ATT&CK
- Workbooks
- Automation

This was important because some older Sentinel documentation and tutorials reference navigation paths that no longer match the current Defender-based experience.


# 5. Reviewing Data Connectors

After Sentinel onboarding, I reviewed the available Data Connectors.

The environment displayed eight available Microsoft security integrations, including:

```text
Azure Key Vault
Microsoft 365 Insider Risk Management
Microsoft Defender for Cloud
Microsoft Defender for Endpoint
Microsoft Defender for Identity
Microsoft Defender for Office
Microsoft Defender XDR
Microsoft Entra ID Protection
```

I reviewed their connection status and confirmed the available integrations were connected in the environment.

However, I did not treat a connected status as proof that useful telemetry was currently available.

Telemetry ingestion was validated separately through KQL.

# 6. Installing Relevant Content Hub Solutions

I next opened Microsoft Sentinel Content Hub.

The environment contained approximately 490 available solutions.

Rather than installing large amounts of unrelated content, I selected solutions relevant to the resources and security capabilities used in the project.

I installed:

```text
Microsoft Defender for Cloud
Azure Key Vault
Azure Activity
Threat Intelligence
```

I initially looked for Microsoft Entra ID content, but the required option was not available in the environment as expected.

I therefore installed Azure Activity instead.

The installed Content Hub packages provided access to relevant Sentinel content, including templates for:

- Analytics
- Workbooks
- Threat intelligence
- Azure monitoring

# 7. Validating Telemetry Ingestion

After configuring Sentinel and reviewing the integrations, I validated whether the Log Analytics workspace actually contained data.

I used:

```kusto
search *
| summarize EventCount = count() by $table
| order by EventCount desc
```

My initial investigation used a 24-hour time range.

The query returned no useful results.

Rather than assuming Sentinel or the ingestion pipeline was broken, I changed the investigation period to:

```text
Last 7 days
```

The query then returned data from:

```text
AzureMetrics
Usage
AzureDiagnostics
```

During the validation period, the results included:

| Table | Event Count |
|---|---:|
| `AzureMetrics` | 24 |
| `Usage` | 7 |
| `AzureDiagnostics` | 4 |

This confirmed that telemetry was available within the workspace.

# 8. Identifying Azure Key Vault Telemetry

I next investigated the `AzureDiagnostics` table.

I started with:

```kusto
AzureDiagnostics
| order by TimeGenerated desc
| take 50
```

The results contained Azure Key Vault events.

Relevant fields included:

```text
TimeGenerated
ResourceGroup
ResourceProvider
Resource
ResourceType
OperationName
ResultType
```

The events showed activity associated with:

```text
Resource Group:
RG-CONTOSO-PROD-001

Resource Provider:
MICROSOFT.KEYVAULT

Resource:
KV-CONTOSO-PROD-001

Resource Type:
VAULTS

Operation:
VaultGet

Result:
Success
```

This confirmed that the Key Vault telemetry could be used for security analysis and threat hunting.

# 9. Developing the KQL Investigation Queries

With the telemetry identified, I developed queries specifically for Key Vault analysis.

## 9.1 Key Vault Activity Monitoring

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

This returned four successful Key Vault events during the selected investigation period.


## 9.2 Failed Key Vault Operations

I then searched for unsuccessful activity:

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

The query returned:

```text
0 results
```

This indicated that no events matching the unsuccessful-operation condition were present within the available telemetry and selected investigation period.

## 9.3 Activity Frequency Analysis

I also summarized the activity:

```kusto
AzureDiagnostics
| where ResourceProvider =~ "MICROSOFT.KEYVAULT"
| summarize EventCount = count() by Resource, OperationName, ResultType
| order by EventCount desc
```

This provided a simple baseline of the operations observed within the dataset.

# 10. Configuring the Analytics Rule

After validating telemetry, I moved into Microsoft Sentinel Analytics.

At the beginning of this stage, the environment showed:

```text
0 Active Rules
```

Three relevant templates were available:

```text
Anomalous RDP Login Detection

Anomalous SSH Login Detection

Microsoft Defender Threat Intelligence Analytics
```

Because the environment did not contain virtual machines used for RDP or SSH monitoring, I did not enable the RDP and SSH templates merely to increase the number of active rules.

I selected:

```text
Microsoft Defender Threat Intelligence Analytics
```

The rule configuration showed:

| Property | Value |
|---|---|
| Rule Name | Microsoft Defender Threat Intelligence Analytics |
| Severity | Medium |
| Status | Enabled |
| ATT&CK Tactic | Persistence |
| ATT&CK Tactic | Lateral Movement |

The template description indicated that the rule generates alerts when Microsoft Defender Threat Intelligence indicators match activity in event logs.

I reviewed the configuration and created the rule.

The environment then had an active analytics capability.

# 11. Reviewing Incident Management

After configuring Analytics, I opened Microsoft Sentinel Incidents.

At the time of validation:

```text
Active Incidents: 0
```

The interface provided filters including:

```text
Status
- New
- In Progress

Severity
- High
- Medium
- Low
```

There was no active incident requiring investigation.

I did not manufacture an incident or classify normal Key Vault activity as malicious solely to demonstrate the incident interface.

Instead, I continued building the surrounding detection and response workflow so that future incidents could enter the automation process.


# 12. Configuring Threat Hunting

When I initially opened Sentinel Hunting, the environment showed:

```text
0 Hunting Queries
```

I used Content Hub to install the relevant security content and then created a custom Hunt.

The Hunt was named:

```text
Azure Key Vault Suspicious Activity Investigation
```

The Hunt configuration included:

```text
Hunt Name
Description
Owner
Status
Hypothesis
```

The available hypothesis states were:

```text
Unknown
Invalidated
Validated
```

I initially used:

```text
Unknown
```

because the investigation had not yet established whether the available evidence supported the security hypothesis.


# 13. Adding Queries to the Hunt

Inside the Hunt, I created three custom queries.

## Query 1

```text
Key Vault Activity Monitoring
```

Purpose:

```text
Review Key Vault operations available within AzureDiagnostics.
```

## Query 2

```text
Failed Key Vault Operations
```

Purpose:

```text
Identify unsuccessful Key Vault operations that may require investigation.
```

## Query 3

```text
Key Vault Activity Frequency Analysis
```

Purpose:

```text
Summarize activity frequency and outcomes to establish a basic baseline.
```

I ran all three queries from within the Hunt.

The results were consistent with my earlier Log Analytics investigation.

I observed:

```text
4 successful Key Vault events
0 unsuccessful operations matching the failure query
```

The available results did not provide evidence requiring escalation.

After reviewing the findings, I completed the Hunt and updated the hypothesis based on the evidence.


# 14. Reviewing MITRE ATT&CK Coverage

I next opened Microsoft Sentinel's MITRE ATT&CK coverage view.

The matrix allowed me to review detection coverage across ATT&CK tactics and techniques.

I inspected available technique information and reviewed fields such as:

```text
Technique ID
Incidents
Alerts
Description
Tactic
```

I then returned to the enabled Microsoft Defender Threat Intelligence Analytics rule and confirmed its ATT&CK mappings:

```text
Persistence
Lateral Movement
```

This allowed me to connect the individual analytics rule to the wider detection-coverage model.


# 15. Configuring the Azure Key Vault Security Workbook

I next moved to Microsoft Sentinel Workbooks.

Initially, the environment showed:

```text
0 Saved Workbooks
```

Four relevant templates were available:

```text
Azure Activity
Azure Key Vault Security
Azure Service Health Workbook
Threat Intelligence
```

I selected:

```text
Azure Key Vault Security
```

and opened the template.

The workbook initially displayed errors because the required environment parameters were not configured.

I configured:

```text
Workspace:
law-contoso-prod-001

Key Vault:
kv-contoso-prod-001
```

After refreshing the workbook, the visualizations populated successfully.

I reviewed sections covering:

- Key Vault analytics
- Diagnostic-log coverage
- Key Vault monitoring
- Request activity
- Latency
- Operational telemetry

I then saved the configured workbook.


# 16. Creating the Native Sentinel Automation Rule

After completing the SIEM components, I moved to Sentinel Automation.

The environment initially contained:

```text
0 Existing Automation Rules
```

I selected:

```text
Create
```

and configured a Standard rule.

The rule was named:

```text
Contoso Sentinel Incident Triage
```

The trigger was:

```text
When incident is created
```

I configured two automated actions.

## Action 1: Add Tag

```text
Automated-Triage
```

## Action 2: Add Investigation Task

```text
Perform Initial SOC Investigation
```

The task directs the analyst to review the incident, alerts, affected entities, ATT&CK mappings, supporting telemetry, and related activity before determining whether escalation, containment, or closure is appropriate.

I did not configure automatic incident closure or automatic severity changes.

The final native automation workflow was:

```text
New Incident
     |
     v
Contoso Sentinel Incident Triage
     |
     +----------------------+
     |                      |
     v                      v
Add Automated-Triage    Create Initial SOC
Tag                     Investigation Task
     |                      |
     +----------+-----------+
                |
                v
          Analyst Review
```


# 17. Reviewing Available Playbooks

After configuring native automation, I opened Sentinel Playbooks.

Initially:

```text
Active Playbooks: 0
```

Seven Microsoft Defender Threat Intelligence templates were available, including:

```text
MDTI-Automated-Triage
MDTI-Intel-Reputation
MDTI-Data-Cookies
MDTI-Data-Trackers
MDTI-Data-ReverseDnS
MDTI-Data-PassiveDns
MDTI-Data-WebComponents
```

I selected:

```text
MDTI-Automated-Triage
```

because it aligned most closely with the automated incident-triage objective of the project.


# 18. Deploying MDTI-Automated-Triage

I reviewed the template requirements before deployment.

The documentation referenced prerequisites and post-deployment configuration involving:

- Microsoft Defender Threat Intelligence
- Microsoft Sentinel
- Managed identity
- Azure RBAC
- API permissions

I proceeded with deployment to determine what the environment supported.

The deployment completed successfully.

The resulting playbook showed:

```text
Playbook:
MDTI-Automated-Triage

Status:
Active

Type:
Logic App

Input:
Sentinel Incident

Plan:
Consumption

Resource Group:
rg-contoso-prod-001

Location:
East US 2
```

This confirmed successful creation of the Logic App resource.

# 19. Reviewing the Logic App Workflow

I opened the Logic App Designer to inspect the workflow created by the template.

The visible workflow contained steps associated with:

```text
Sentinel incident trigger

Initialize classification

Get Host entities

Get IP entities

Initialize host results

Initialize IP results

Process each Host

Process each IP Address

Evaluate Malicious or Suspicious condition
```

The playbook therefore provided a more advanced automated triage workflow than the native Sentinel automation rule.

The relationship became:

```text
Sentinel Incident
       |
       v
MDTI-Automated-Triage
       |
       +-- Get Hosts
       |
       +-- Get IPs
       |
       +-- Process Entities
       |
       +-- Threat Classification
       |
       v
Sentinel Response
```

# 20. Validating the Microsoft Sentinel Connection

Within the Logic App Designer, I reviewed the configured connections.

The workflow contained a Microsoft Sentinel connection.

The connection displayed a healthy status and exposed Sentinel actions including:

```text
Incident creation trigger

Entities - Get Hosts

Entities - Get IPs

Add comment to incident

Update incident
```

I retained the existing connection rather than recreating it unnecessarily.


# 21. Enabling and Validating Managed Identity

I opened the Logic App's Identity configuration.

The system-assigned managed identity showed:

```text
Status: On
```

This provided the Logic App with an Azure identity that could be granted access through RBAC without embedding credentials in the workflow.

The authorization model was:

```text
Logic App
    |
    v
System-Assigned Managed Identity
    |
    v
Azure RBAC
    |
    v
Microsoft Sentinel Resources
```


# 22. Configuring Azure RBAC

I checked the Azure role assignments associated with the Logic App managed identity.

Initially:

```text
Role Assignments: 0
```

I then assigned:

```text
Microsoft Sentinel Contributor
```

to the `MDTI-Automated-Triage` managed identity.

The assignment was scoped to:

```text
rg-contoso-prod-001
```

rather than the entire subscription.

The role assignment completed successfully.

This allowed the playbook identity to interact with Sentinel resources within the relevant resource-group scope.

# 23. MDTI API Permission Limitation

The playbook documentation also referenced:

```text
ThreatIntelligence.Read.All
```

as an additional permission requirement.

I attempted to locate the Logic App identity through Microsoft Entra Enterprise Applications to verify the permission.

The identity was not discoverable through the interface available to me.

I therefore did not claim that this specific API permission had been independently verified.

Instead, I retained it as a documented limitation and proceeded to validate the actual Logic App execution.


# 24. Validating the Logic App Trigger

I tested the Sentinel trigger associated with the playbook.

The environment reported that the trigger:

```text
Successfully ran
```

This confirmed that the Sentinel incident trigger could initiate the workflow.

However, I did not treat trigger success alone as proof that the entire playbook was functioning.

I opened the run details to inspect the downstream workflow.


# 25. Validating the SOAR Workflow Execution

The Logic App run details showed successful execution indicators through the visible automated-triage workflow.

The successful path included:

```text
Sentinel Incident Trigger
        |
        v
Initialize Classification
        |
        +-------------------+
        |                   |
        v                   v
Get Hosts               Get IPs
        |                   |
        v                   v
Initialize Results      Initialize Results
        |                   |
        v                   v
Process Hosts           Process IP Addresses
        |                   |
        +---------+---------+
                  |
                  v
        Malicious / Suspicious
              Evaluation
```

The visible actions showed successful execution.

I interpreted this result carefully.

The successful condition execution demonstrated that the workflow reached and processed the classification logic.

It did not, by itself, prove that a malicious host or IP address had been identified.

The validated conclusion was therefore:

> The Microsoft Sentinel-triggered Logic App successfully executed through the visible automated-triage processing path.


# 26. Final Deployment State

At the end of the implementation, the environment contained the following major components:

| Component | Final State |
|---|---|
| Microsoft Sentinel | Deployed |
| Log Analytics Integration | Operational |
| Azure Key Vault Telemetry | Available |
| Data Connectors | Configured |
| Content Hub Solutions | Installed |
| KQL Analysis | Validated |
| Analytics Rule | Enabled |
| Incident Workflow | Configured |
| Threat Hunt | Completed |
| Custom Hunting Queries | 3 |
| MITRE ATT&CK Analysis | Completed |
| Key Vault Workbook | Configured |
| Native Automation Rule | Active |
| MDTI Playbook | Active |
| Managed Identity | Enabled |
| Sentinel Contributor RBAC | Assigned |
| Logic App Execution | Validated |
| UEBA | Limited by tenant permissions |
| `ThreatIntelligence.Read.All` | Not independently verified |


# 27. Deployment Validation Checklist

The following checklist summarizes the implementation:

```text
[✓] Existing Log Analytics workspace identified
[✓] Microsoft Sentinel enabled
[✓] Data connectors reviewed
[✓] Relevant Content Hub solutions installed
[✓] Telemetry queried directly
[✓] AzureDiagnostics identified
[✓] Key Vault events validated
[✓] Custom KQL queries created
[✓] Analytics rule enabled
[✓] Incident management reviewed
[✓] Structured Hunt created
[✓] Three hunting queries added
[✓] Hunt executed
[✓] ATT&CK mappings reviewed
[✓] Key Vault workbook configured
[✓] Native automation rule created
[✓] Incident triage actions configured
[✓] MDTI playbook deployed
[✓] Sentinel connection validated
[✓] System-assigned identity enabled
[✓] Microsoft Sentinel Contributor assigned
[✓] Logic App trigger executed
[✓] Workflow run validated

[!] UEBA limited by tenant administrative permissions
[!] ThreatIntelligence.Read.All not independently verified
```


# 28. Deployment Outcome

The deployment resulted in an end-to-end Microsoft Sentinel security operations environment covering:

```text
Telemetry Collection
        |
        v
Centralized Logging
        |
        v
KQL Analysis
        |
        v
Security Detection
        |
        v
Incident Management
        |
        v
Threat Hunting
        |
        v
MITRE ATT&CK Analysis
        |
        v
Security Visualization
        |
        v
Native Automation
        |
        v
Logic App SOAR
        |
        v
Automated Triage
```

The project was considered complete only after the major components were not only configured but, where possible, validated against the actual Azure environment.

## Related Documentation

- [Architecture](Architecture.md)
- [Data Connectors](DataConnectors.md)
- [KQL Queries](KQLQueries.md)
- [Analytics Rules](AnalyticsRules.md)
- [Incident Management](IncidentManagement.md)
- [Threat Hunting](ThreatHunting.md)
- [MITRE ATT&CK Mapping](MITRE-ATTACK-Mapping.md)
- [Workbooks](Workbooks.md)
- [SOAR Automation](SOAR-Automation.md)
- [Validation Report](ValidationReport.md)
- [Troubleshooting](Troubleshooting.md)
- [Lessons Learned](LessonsLearned.md)
