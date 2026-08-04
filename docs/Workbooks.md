# Microsoft Sentinel Workbooks and Security Visualization

## 1. Overview

This document describes the Microsoft Sentinel workbook implementation used to visualize security and operational telemetry from the Azure environment.

After validating telemetry through Kusto Query Language (KQL), configuring analytics, conducting threat hunting, and reviewing MITRE ATT&CK coverage, I moved to the visualization layer of Microsoft Sentinel.

The objective was to use a workbook that aligned with a resource actually deployed and generating telemetry in the environment.

The selected workbook was:

```text
Azure Key Vault Security
```

It was configured against:

```text
Log Analytics Workspace:
law-contoso-prod-001

Azure Key Vault:
kv-contoso-prod-001
```

The workbook initially failed to populate correctly because the required parameters had not been configured.

After selecting the correct workspace and Key Vault and refreshing the workbook, the visualizations populated successfully.

This provided a graphical monitoring layer over the same Azure Key Vault environment previously investigated using KQL.


# 2. Role of Workbooks in Microsoft Sentinel

Microsoft Sentinel Workbooks provide interactive visualization and monitoring capabilities over security and operational telemetry.

While KQL allows an analyst to interact directly with logs:

```text
Telemetry
    |
    v
KQL Query
    |
    v
Query Results
```

workbooks provide a visual layer:

```text
Telemetry
    |
    v
Log Analytics
    |
    v
Workbook Queries
    |
    v
Charts / Metrics / Tables
    |
    v
Security Visualization
```

Both approaches use telemetry, but they serve different operational purposes.

KQL is particularly useful for investigation and ad hoc analysis.

Workbooks are useful for continuously presenting relevant information in a structured visual format.


# 3. Workbook Architecture

The workbook architecture implemented in the project was:

```text
Azure Key Vault
kv-contoso-prod-001
        |
        v
Diagnostic Telemetry
        |
        v
AzureDiagnostics / Azure Metrics
        |
        v
Log Analytics
law-contoso-prod-001
        |
        v
Microsoft Sentinel
        |
        v
Azure Key Vault Security Workbook
        |
        v
Security Visualization
```

This meant the workbook was not operating as an isolated dashboard.

It was another way of consuming telemetry from the same monitored Azure environment.


# 4. Initial Workbook State

When I initially opened Microsoft Sentinel Workbooks, the environment showed:

```text
0 Saved Workbooks
```

This meant that no workbook had yet been saved and configured specifically for the environment.

However, several templates were available.

The available templates relevant to the installed Sentinel content were:

```text
Azure Activity

Azure Key Vault Security

Azure Service Health Workbook

Threat Intelligence
```

The next step was to determine which workbook best aligned with the environment.


# 5. Evaluating the Available Templates

I reviewed the four available workbook templates before making a selection.

| Workbook | Primary Focus |
|---|---|
| Azure Activity | Azure subscription and resource activity |
| Azure Key Vault Security | Key Vault security and operational monitoring |
| Azure Service Health Workbook | Azure platform health and service events |
| Threat Intelligence | Threat-intelligence monitoring and analysis |

Each workbook had potential value, but I wanted the project to maintain a direct relationship between:

```text
Deployed Resource
      |
      v
Collected Telemetry
      |
      v
Security Investigation
      |
      v
Visualization
```

Azure Key Vault provided the strongest match.


# 6. Why I Selected Azure Key Vault Security

The selected workbook was:

```text
Azure Key Vault Security
```

This decision was based on the telemetry already validated during the KQL and threat-hunting phases.

I had already confirmed that:

```text
AzureDiagnostics
```

contained events from:

```text
kv-contoso-prod-001
```

The available events included:

```text
ResourceProvider:
MICROSOFT.KEYVAULT

OperationName:
VaultGet

ResultType:
Success
```

The Key Vault workbook therefore provided an opportunity to visualize telemetry from a resource I had already investigated directly.


# 7. Consistency Across the Project

Selecting the Key Vault workbook created a consistent investigation path:

```text
Azure Key Vault
      |
      v
Diagnostic Telemetry
      |
      v
KQL Investigation
      |
      v
Threat Hunt
      |
      v
Security Workbook
```

This was preferable to selecting a workbook simply because it looked visually impressive.

The visualization remained connected to the same resource and telemetry used throughout the technical investigation.


# 8. Opening the Workbook Template

I selected:

```text
Azure Key Vault Security
```

and opened the workbook template.

The template exposed the workbook structure and visualization components.

However, the workbook did not immediately display the expected information.

Several sections showed configuration-related errors.

At this point, the workbook existed as a template, but it had not yet been connected to the correct environment parameters.


# 9. Initial Workbook Problem

The workbook initially displayed errors similar to:

```text
Failed to resolve parameter
```

and some visualizations could not execute.

This did not mean the Key Vault telemetry was unavailable.

The earlier KQL investigation had already confirmed that the telemetry existed.

The problem was therefore investigated as a workbook configuration issue.


# 10. Identifying the Required Parameters

The workbook required environment-specific parameters before its queries could execute correctly.

The important parameters were:

```text
Workspace

KeyVault
```

The workbook needed to know:

```text
Which Log Analytics workspace should be queried?

Which Azure Key Vault should be monitored?
```

Without those values, the workbook could not correctly scope its queries.


# 11. Configuring the Workspace

I configured the workbook workspace parameter as:

```text
law-contoso-prod-001
```

This was the same Log Analytics workspace that had been onboarded to Microsoft Sentinel and used throughout the KQL investigation.

The relationship became:

```text
Azure Key Vault Security Workbook
             |
             v
law-contoso-prod-001
```

This gave the workbook the correct telemetry source.


# 12. Configuring the Key Vault

I then configured the Key Vault parameter as:

```text
kv-contoso-prod-001
```

The complete workbook configuration became:

```text
Workspace:
law-contoso-prod-001

KeyVault:
kv-contoso-prod-001
```

This allowed the workbook to scope its monitoring views to the correct Azure Key Vault.


# 13. Refreshing the Workbook

After configuring the required parameters, I refreshed the workbook.

The previously failing visualizations populated successfully.

The troubleshooting path was:

```text
Open Workbook
      |
      v
Visualization Errors
      |
      v
Inspect Parameters
      |
      v
Configure Workspace
      |
      v
Configure Key Vault
      |
      v
Refresh
      |
      v
Workbook Populated
```

This confirmed that the issue was related to workbook parameter configuration rather than the absence of telemetry.


# 14. Saving the Workbook

After validating that the workbook populated successfully, I saved it to the environment.

This changed the state from:

```text
Available Template
```

to:

```text
Configured and Saved Workbook
```

This distinction is important.

A workbook template being available does not mean it has been configured for the environment.

Likewise, saving a workbook does not automatically prove that its underlying visualizations are successfully retrieving data.

I therefore validated the workbook after configuration.


# 15. Workbook Validation

The workbook was considered successfully implemented only after the following sequence was completed:

```text
Template Available
      |
      v
Template Opened
      |
      v
Parameters Configured
      |
      v
Workspace Selected
      |
      v
Key Vault Selected
      |
      v
Workbook Refreshed
      |
      v
Visualizations Populated
      |
      v
Workbook Saved
```

This provided stronger evidence than simply showing that the workbook template existed.


# 16. Workbook Sections

After successful configuration, the workbook provided several monitoring and visualization areas related to Azure Key Vault.

The visible sections included information associated with:

```text
Key Vault analytics

Diagnostic-log coverage

Monitoring timelines

Requests

Latency

Operational behavior
```

These visualizations provided a higher-level view of the Key Vault environment than the individual KQL queries used earlier.


# 17. Diagnostic Log Coverage

One of the useful workbook capabilities was visibility into diagnostic logging.

Diagnostic logging is critical because security analytics cannot operate effectively without telemetry.

The monitoring relationship is:

```text
Azure Key Vault
      |
      v
Diagnostic Settings
      |
      v
Log Analytics
      |
      v
Security Monitoring
```

The workbook provided visual context around this telemetry pipeline.

This complemented the direct KQL validation I had already performed.


# 18. Request Monitoring

The workbook also provided views related to Key Vault request activity.

Request monitoring can help analysts understand:

```text
How frequently the service is being accessed

Whether activity changes over time

Whether unusual request patterns emerge

Whether operational behavior differs from the expected baseline
```

In this project, the available dataset was small.

I therefore used the workbook as a visualization and monitoring demonstration rather than claiming that the limited activity represented a mature production baseline.


# 19. Latency Monitoring

The workbook included latency-related monitoring information.

Although latency is primarily an operational metric, it can still be useful during security monitoring because unusual service behavior may require investigation alongside other telemetry.

Security operations often benefit from combining:

```text
Security Signals
       +
Operational Context
```

rather than treating them as completely separate domains.


# 20. Key Vault Analytics

The workbook also provided Key Vault-specific analytical views.

These helped demonstrate how Microsoft Sentinel and Azure Monitor can transform raw telemetry into information that is easier to consume visually.

The relationship was:

```text
Raw Events
    |
    v
KQL / Workbook Queries
    |
    v
Aggregation
    |
    v
Visualization
    |
    v
Analyst Interpretation
```

This does not remove the need for raw-log investigation.

Instead, it provides another analytical layer.


# 21. KQL vs Workbooks

Both KQL and Workbooks were used in the project, but for different purposes.

| Capability | KQL | Workbooks |
|---|---|---|
| Raw log investigation | Strong | Limited |
| Custom filtering | Strong | Depends on workbook |
| Ad hoc hunting | Strong | Limited |
| Visualization | Basic | Strong |
| Dashboards | Limited | Strong |
| Reusable monitoring view | Possible | Strong |
| Deep investigation | Strong | Supporting |
| Executive/operational visibility | Limited | Strong |

The two capabilities complement one another.


# 22. Example Investigation Workflow

A SOC analyst could begin with the workbook:

```text
Workbook
    |
    v
Notice Unusual Pattern
    |
    v
Identify Time Period / Resource
    |
    v
Open Logs
    |
    v
Run KQL
    |
    v
Investigate Events
    |
    v
Determine Whether Escalation Is Required
```

Alternatively, an analyst could start with KQL and later use the workbook to understand broader trends.


# 23. Relationship with Threat Hunting

The workbook was configured after the Key Vault threat-hunting investigation.

This created two different investigation perspectives over the same resource.

### Threat Hunting

```text
Hypothesis
    |
    v
Custom KQL
    |
    v
Evidence
    |
    v
Conclusion
```

### Workbook Monitoring

```text
Telemetry
    |
    v
Prebuilt Queries
    |
    v
Visualizations
    |
    v
Monitoring Context
```

Threat hunting is more hypothesis-driven.

Workbooks are better suited to persistent visual monitoring and contextual analysis.


# 24. Relationship with Analytics Rules

Workbooks and analytics rules also serve different functions.

```text
Analytics Rule
      |
      v
Automatically Detect Condition
      |
      v
Alert / Incident
```

compared with:

```text
Workbook
      |
      v
Visualize Telemetry
      |
      v
Analyst Interpretation
```

A workbook does not replace an analytics rule.

A visual anomaly does not automatically become an alert unless corresponding detection logic exists.


# 25. Relationship with Incident Investigation

Workbooks can also support incident investigations.

An analyst reviewing an incident could use a workbook to gain broader environmental context.

For example:

```text
Sentinel Incident
      |
      v
Affected Azure Resource
      |
      v
Open Relevant Workbook
      |
      v
Review Historical / Operational Context
      |
      v
Pivot to KQL
      |
      v
Continue Investigation
```

This makes workbooks useful as supporting investigation tools.


# 26. Workbook Selection Strategy

The project reinforced that workbook selection should be based on the environment rather than appearance.

A useful selection process is:

```text
What resources exist?
      |
      v
What telemetry is available?
      |
      v
What monitoring question exists?
      |
      v
Which workbook addresses it?
      |
      v
Configure Parameters
      |
      v
Validate Results
```

This is preferable to:

```text
Install Everything
      |
      v
Save Every Workbook
      |
      v
Assume Monitoring Exists
```


# 27. Available vs Configured vs Validated

The workbook implementation demonstrated three different states.

## Available

```text
Workbook template appears in Sentinel
```

This means the content exists.

## Configured

```text
Required workspace/resource parameters have been selected
```

This means the workbook has been adapted to the environment.

## Validated

```text
Workbook queries execute and visualizations populate
```

This demonstrates that the workbook is functioning against the selected environment.

The project reached the third state for the Azure Key Vault Security workbook.


# 28. Troubleshooting Summary

The main workbook issue encountered was:

```text
Problem:
Workbook visualizations failed to populate correctly.

Cause:
Required environment parameters had not been configured.

Resolution:
Configured:

Workspace = law-contoso-prod-001

KeyVault = kv-contoso-prod-001

Then refreshed the workbook.

Result:
Workbook visualizations populated successfully.
```

This troubleshooting experience reinforced that a prebuilt workbook still requires correct environment configuration.


# 29. Workbook Security Considerations

Workbooks should be treated as security monitoring tools rather than decorative dashboards.

Several considerations are important.

## Data Quality

A workbook can only visualize the telemetry available to its queries.

If required logs are missing, the workbook may provide incomplete visibility.

## Time Range

Workbook results depend on the selected monitoring period.

## Parameter Configuration

Incorrect workspace or resource parameters can cause failed or misleading results.

## Access Control

Users viewing security workbooks should have appropriate access to the underlying telemetry.

## Analyst Validation

Unexpected visual patterns should be investigated using the underlying logs rather than interpreted solely from a chart.


# 30. Workbook Validation Results

The implementation produced the following validation state:

| Component | Result |
|---|---|
| Sentinel Workbooks interface | Accessible |
| Initial saved workbooks | 0 |
| Relevant templates observed | 4 |
| Azure Activity template | Available |
| Azure Key Vault Security | Selected |
| Azure Service Health Workbook | Available |
| Threat Intelligence workbook | Available |
| Initial Key Vault workbook state | Configuration errors |
| Workspace parameter | Configured |
| Key Vault parameter | Configured |
| Workbook refresh | Successful |
| Visualizations | Populated |
| Workbook | Saved |
| Data source | `law-contoso-prod-001` |
| Monitored resource | `kv-contoso-prod-001` |


# 31. Evidence Captured

The workbook implementation can be supported with screenshots showing:

```text
Sentinel Workbooks interface

Available workbook templates

Azure Key Vault Security template

Initial workbook configuration state

Workspace selection

Key Vault selection

Populated Key Vault dashboard

Saved workbook
```

Recommended repository structure:

```text
images/
└── workbooks/
    ├── sentinel-workbooks-overview.png
    ├── available-workbook-templates.png
    ├── key-vault-security-template.png
    ├── key-vault-workbook-parameters.png
    ├── key-vault-workbook-dashboard.png
    └── saved-key-vault-workbook.png
```

Only screenshots actually captured during implementation should be added.


# 32. Limitations

The workbook implementation had several limitations.

## Small Telemetry Dataset

The Key Vault environment contained limited telemetry compared with a production environment.

## Single Saved Workbook

The project focused on validating one relevant workbook rather than saving every available template.

## Limited Historical Analysis

The available telemetry did not provide enough historical depth for advanced trend analysis.

## No SOC-Wide Executive Dashboard

The project did not build a custom dashboard combining incidents, alerts, identities, endpoints, and cloud-resource activity.

## No Custom Workbook Development

The implementation used and configured a Microsoft-provided workbook rather than developing a completely custom workbook from scratch.

These limitations define the scope of the implementation rather than representing hidden functionality.


# 33. Future Improvements

A future version of the environment could expand workbook capabilities.

### Custom SOC Overview Workbook

Create a dashboard showing:

```text
Incident count

Incident severity

Alert trends

Top affected entities

ATT&CK coverage

Automation activity

Threat-hunting findings
```

### Key Vault Security Dashboard

Build a custom Key Vault workbook focusing on:

```text
Successful operations

Failed operations

Operation frequency

Top identities

Source IP addresses

Administrative changes

Secret access activity
```

### Identity Workbook

Add identity-focused monitoring if Microsoft Entra telemetry becomes available.

### Endpoint Workbook

Add endpoint-security visualization after onboarding Defender for Endpoint telemetry.

### SOAR Metrics

Create visualizations for:

```text
Automation executions

Successful playbook runs

Failed playbook runs

Automated incident actions
```

### SOC Performance Metrics

Track metrics such as:

```text
Mean Time to Acknowledge

Mean Time to Investigate

Mean Time to Respond

Mean Time to Resolve
```


# 34. Example Custom SOC Workbook Architecture

A future custom workbook could combine several data sources:

```text
                  Microsoft Sentinel
                         |
        +----------------+----------------+
        |                |                |
        v                v                v
     Incidents         Alerts         Hunting
        |                |                |
        +----------------+----------------+
                         |
                         v
                  Custom SOC Workbook
                         |
        +----------------+----------------+
        |                |                |
        v                v                v
 Incident Trends    ATT&CK Coverage   Entity Activity
        |
        v
Automation Metrics
```

This would provide a broader operational dashboard than the resource-specific Key Vault workbook used in the current project.


# 35. Skills Demonstrated

The workbook phase demonstrated experience with:

```text
Microsoft Sentinel Workbooks

Azure Monitor Workbooks

Azure Key Vault monitoring

Log Analytics integration

Security visualization

Workbook parameters

Telemetry validation

Dashboard troubleshooting

Security monitoring

Operational monitoring
```

It also demonstrated the ability to move between:

```text
Raw Telemetry

KQL Investigation

Threat Hunting

Visual Monitoring
```

without treating them as isolated capabilities.


# 36. Key Lessons

Several practical lessons came from the workbook implementation.

## A Template Is Not a Finished Dashboard

Prebuilt workbooks still require environment-specific configuration.

## Parameters Matter

The workbook could not function correctly until the appropriate workspace and Key Vault were selected.

## Visualization Depends on Telemetry

A dashboard cannot provide meaningful information when the underlying logs are missing.

## KQL and Workbooks Complement Each Other

Workbooks provide visual context while KQL supports deeper investigation.

## Resource-Relevant Workbooks Are More Valuable

Selecting the Key Vault workbook created a direct relationship with the telemetry already investigated in the project.

## Validation Should Go Beyond Saving

The workbook was only considered implemented after its visualizations populated successfully.


# 37. Final Workbook Outcome

The final implementation can be summarized as:

```text
Initial Saved Workbooks:
0

Available Relevant Templates:
4

Selected Workbook:
Azure Key Vault Security

Workspace:
law-contoso-prod-001

Key Vault:
kv-contoso-prod-001

Initial State:
Configuration errors

Resolution:
Configured required parameters

Final State:
Visualizations populated successfully

Workbook:
Saved and validated
```

The final monitoring flow was:

```text
Azure Key Vault
      |
      v
Diagnostic Telemetry
      |
      v
Log Analytics
      |
      +--------------------------+
      |                          |
      v                          v
Custom KQL                Security Workbook
      |                          |
      v                          v
Deep Investigation        Visual Monitoring
      |                          |
      +-------------+------------+
                    |
                    v
             Analyst Context
```

The workbook implementation completed the visualization layer of the Sentinel environment and demonstrated how the same Azure telemetry used for KQL and threat hunting could also support reusable security monitoring dashboards.


## Related Documentation

- [Architecture](Architecture.md)
- [Deployment Guide](DeploymentGuide.md)
- [Data Connectors](DataConnectors.md)
- [KQL Queries](KQLQueries.md)
- [Analytics Rules](AnalyticsRules.md)
- [Incident Management](IncidentManagement.md)
- [Threat Hunting](ThreatHunting.md)
- [MITRE ATT&CK Mapping](MITRE-ATTACK-Mapping.md)
- [SOAR Automation](SOAR-Automation.md)
- [Validation Report](ValidationReport.md)
- [Troubleshooting](Troubleshooting.md)
