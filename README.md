# Microsoft Sentinel SIEM & SOAR Security Operations Environment

![Microsoft Sentinel](https://img.shields.io/badge/Microsoft%20Sentinel-SIEM%20%26%20SOAR-0078D4?style=for-the-badge&logo=microsoftazure)
![Microsoft Azure](https://img.shields.io/badge/Microsoft%20Azure-Cloud%20Security-0078D4?style=for-the-badge&logo=microsoftazure)
![KQL](https://img.shields.io/badge/KQL-Threat%20Hunting-5C2D91?style=for-the-badge)
![MITRE ATT&CK](https://img.shields.io/badge/MITRE%20ATT%26CK-Detection%20Mapping-red?style=for-the-badge)
![SOAR](https://img.shields.io/badge/SOAR-Automated%20Triage-orange?style=for-the-badge)
![Azure Logic Apps](https://img.shields.io/badge/Azure%20Logic%20Apps-Playbooks-0066FF?style=for-the-badge&logo=microsoftazure)
![Status](https://img.shields.io/badge/Project-Completed-success?style=for-the-badge)


##  Project Overview

I built this project to gain hands-on experience implementing and operating a cloud-native Security Information and Event Management (SIEM) and Security Orchestration, Automation and Response (SOAR) environment using Microsoft Sentinel.

Rather than stopping after deploying Microsoft Sentinel or connecting a Log Analytics workspace, I built the environment around the security operations lifecycle: telemetry collection, log analysis, detection, incident management, proactive threat hunting, MITRE ATT&CK coverage analysis, security visualization, and automated incident triage.

I used an existing Azure environment containing a production-style Azure Key Vault and Log Analytics workspace. This allowed me to work with actual Azure diagnostic telemetry rather than relying entirely on sample data.

During the implementation, I onboarded the existing Log Analytics workspace to Microsoft Sentinel, configured available security integrations, installed relevant Content Hub solutions, queried Azure telemetry using Kusto Query Language (KQL), enabled an analytics rule, created a structured threat hunt, developed custom hunting queries, analyzed MITRE ATT&CK detection coverage, configured a Key Vault security workbook, and implemented native Sentinel incident automation.

I then extended the SOAR implementation by deploying the `MDTI-Automated-Triage` Microsoft Sentinel playbook through Azure Logic Apps.

I enabled the Logic App's system-assigned managed identity, assigned Microsoft Sentinel Contributor permissions at the resource-group scope, validated the Microsoft Sentinel connection, and reviewed the underlying automated-triage workflow.

Finally, I validated execution through Azure Logic Apps run history and confirmed successful processing through the visible Sentinel incident and entity-triage workflow.

The resulting environment demonstrates an end-to-end security operations flow:

> **Azure Telemetry → Log Analytics → Microsoft Sentinel → KQL → Detection → Incident Management → Threat Hunting → MITRE ATT&CK → Workbooks → SOAR → Automated Triage**


##  Project Objectives

I designed this project around the following objectives:

- Deploy and configure Microsoft Sentinel using an existing Log Analytics workspace.
- Integrate available Azure and Microsoft security data sources.
- Verify that security telemetry was actually being ingested instead of relying solely on connector status.
- Analyze Azure security telemetry using Kusto Query Language (KQL).
- Configure and enable Microsoft Sentinel analytics rules.
- Understand the relationship between analytics rules, alerts, and incidents.
- Develop custom KQL queries for proactive threat hunting.
- Create and complete a structured Microsoft Sentinel Hunt.
- Analyze detection coverage using the MITRE ATT&CK framework.
- Configure Microsoft Sentinel Workbooks for security visualization.
- Implement native Sentinel automation for incident triage.
- Deploy an Azure Logic Apps-based Microsoft Sentinel playbook.
- Configure managed identity and Azure RBAC for the SOAR workflow.
- Validate automated workflow execution.
- Document implementation limitations, troubleshooting decisions, and lessons learned.


##  Architecture

The environment was designed around Microsoft Sentinel as the central security operations platform.

<img width="1536" height="1024" alt="ChatGPT Image Jul 31, 2026, 01_53_28 AM" src="https://github.com/user-attachments/assets/2532b6d8-57ab-416b-8399-2f81455ef0bd" />


## Data Integration & Content Hub

After onboarding `law-contoso-prod-001` to Microsoft Sentinel, I reviewed the data connectors available within the environment and connected the supported Microsoft security services.

The available integrations included services such as:

- Azure Key Vault
- Microsoft Defender for Cloud
- Microsoft Defender for Endpoint
- Microsoft Defender for Identity
- Microsoft Defender for Office
- Microsoft Defender XDR
- Microsoft Entra ID Protection
- Microsoft 365 Insider Risk Management

I also used Microsoft Sentinel **Content Hub** to install security content relevant to the environment.

The solutions I installed included:

- **Azure Key Vault**
- **Microsoft Defender for Cloud**
- **Azure Activity**
- **Threat Intelligence**

Rather than assuming that a connected integration meant telemetry was available, I verified ingestion directly through Log Analytics.

 **Implementation Evidence**

<img width="949" height="905" alt="connectors" src="https://github.com/user-attachments/assets/a2d87e76-e01c-4736-94b8-3363c48daa4a" />

> Detailed implementation: [`docs/DataConnectors.md`](docs/DataConnectors.md)


## Verifying Log Ingestion with KQL

One of my first validation steps was determining whether the Log Analytics workspace actually contained telemetry.

I queried the workspace to identify tables containing events:

```kusto
search *
| summarize EventCount = count() by $table
| order by EventCount desc
```

My initial investigation using a 24-hour time range returned no useful results.

Instead of immediately assuming that ingestion had failed, I expanded the investigation window to **7 days**.

The query then returned data from:

| Table | Observed Events |
|---|---:|
| `AzureMetrics` | 24 |
| `Usage` | 7 |
| `AzureDiagnostics` | 4 |

`AzureDiagnostics` was particularly important because it contained diagnostic telemetry from the Azure Key Vault deployed in the environment.

I inspected those events using:

```kusto
AzureDiagnostics
| order by TimeGenerated desc
| take 50
```

The results confirmed telemetry from:

```text
Resource Group: RG-CONTOSO-PROD-001
Resource Provider: MICROSOFT.KEYVAULT
Resource: KV-CONTOSO-PROD-001
Resource Type: VAULTS
Operation: VaultGet
Result: Success
```

This provided the telemetry I later used for KQL analysis, threat hunting, and workbook validation.

 **Implementation Evidence**

<img width="947" height="908" alt="data table" src="https://github.com/user-attachments/assets/21dbb9ae-a3b0-4b0d-88dc-db419600a779" />

<img width="1600" height="757" alt="key vault diag" src="https://github.com/user-attachments/assets/ea2151e3-74cb-4a4a-8900-48590308df4e" />


## Custom KQL Analysis

After identifying the Key Vault telemetry, I developed KQL queries to investigate the activity further.

### Key Vault Activity Monitoring

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

This query returned **4 successful Key Vault events** during the selected seven-day investigation period.

### Failed Key Vault Operations

I then searched specifically for unsuccessful operations:

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

The query returned **0 matching events**.

I interpreted this result narrowly: no unsuccessful Key Vault operations matching the query were present in the available telemetry during the selected investigation period.

### Key Vault Activity Frequency

I also summarized the activity to establish a simple behavioral baseline:

```kusto
AzureDiagnostics
| where ResourceProvider =~ "MICROSOFT.KEYVAULT"
| summarize EventCount = count() by Resource, OperationName, ResultType
| order by EventCount desc
```

I stored the reusable KQL queries separately in the repository under [`queries/`](queries/).

> Full KQL documentation: [`docs/KQLQueries.md`](docs/KQLQueries.md)


## Analytics & Detection

I enabled the **Microsoft Defender Threat Intelligence Analytics** rule within Microsoft Sentinel.

The rule provides threat-intelligence-based detection by identifying matches between Microsoft Defender Threat Intelligence indicators and activity present in event logs.

### Rule Configuration

| Property | Configuration |
|---|---|
| Rule | Microsoft Defender Threat Intelligence Analytics |
| Severity | Medium |
| Status | Enabled |
| MITRE ATT&CK | Persistence, Lateral Movement |

This established an active detection capability within the Sentinel environment.

I also reviewed the Sentinel incident-management interface and its workflow for handling alerts promoted into security incidents.

At the time of validation, there were **0 active incidents**, so I did not manufacture an incident simply to populate the dashboard.

 **Implementation Evidence**

<img width="944" height="943" alt="threat intel analy" src="https://github.com/user-attachments/assets/98532954-2df2-45af-a14b-cb97ec318995" />


> Analytics documentation: [`docs/AnalyticsRules.md`](docs/AnalyticsRules.md)

> Incident workflow: [`docs/IncidentManagement.md`](docs/IncidentManagement.md)


## Threat Hunting

After validating the telemetry through Log Analytics, I moved from reactive detection into proactive threat hunting.

I created a structured Microsoft Sentinel Hunt named:

> **Azure Key Vault Suspicious Activity Investigation**

The hunt was designed to investigate whether unusual operations, unsuccessful requests, or unexpected activity patterns could indicate unauthorized or anomalous access to the production Key Vault.

I created three custom hunting queries inside the Hunt:

| Hunting Query | Purpose |
|---|---|
| **Key Vault Activity Monitoring** | Review operations performed against Azure Key Vault |
| **Failed Key Vault Operations** | Identify unsuccessful Key Vault operations |
| **Key Vault Activity Frequency Analysis** | Establish the frequency and outcome of observed operations |

### Investigation Results

The investigation identified:

- **4 successful Key Vault events**
- **0 unsuccessful operations matching the failure query**
- A consistent activity pattern within the telemetry available for the investigation period
- No observed result from these queries that warranted escalation to an incident

I therefore completed the hunt without manufacturing suspicious findings and updated the hunting hypothesis based on the evidence collected.

 **Implementation Evidence**

<img width="941" height="940" alt="threta hunting query" src="https://github.com/user-attachments/assets/f742d7ca-16d1-46da-8b0a-3e0d794f9170" />

<img width="939" height="945" alt="complete threat hunt query" src="https://github.com/user-attachments/assets/be253f3c-05f9-4f9e-8253-bb6f8e0c15bd" />


> Full investigation: [`docs/ThreatHunting.md`](docs/ThreatHunting.md)


## MITRE ATT&CK Mapping

I used Microsoft Sentinel's MITRE ATT&CK coverage matrix to review how the environment's detection capabilities mapped to attacker tactics and techniques.

The matrix allowed me to examine covered techniques and drill into individual ATT&CK entries to review information such as:

- Technique ID
- Description
- Tactic
- Associated alerts
- Associated incidents
- Detection coverage

I also validated the ATT&CK mappings associated with the enabled **Microsoft Defender Threat Intelligence Analytics** rule.

The rule mapped to:

```text
Persistence
Lateral Movement
```

This allowed me to connect individual detection rules to the wider ATT&CK coverage view rather than treating MITRE ATT&CK as an isolated framework.

 **Implementation Evidence**

<img width="935" height="940" alt="mitre att ck" src="https://github.com/user-attachments/assets/01ca8099-d56a-44ae-b5ea-6d7f277867ce" />


<img width="947" height="946" alt="analy-per-lateral" src="https://github.com/user-attachments/assets/b12e577b-c3b0-4314-a98a-36d7e79a321f" />


> Detailed ATT&CK analysis: [`docs/MITRE-ATTACK-Mapping.md`](docs/MITRE-ATTACK-Mapping.md)


## Security Visualization with Workbooks

I configured the **Azure Key Vault Security** workbook to visualize security and operational telemetry from the environment.

The template initially could not execute because the required parameters were unset.

I configured:

```text
Workspace: law-contoso-prod-001
Key Vault: kv-contoso-prod-001
```

After refreshing the workbook, the visualizations populated successfully.

I validated:

- Key Vault diagnostic-log coverage
- Key Vault activity analytics
- Activity baseline information
- Request activity
- Monitoring timelines
- Latency and service metrics

I then saved the configured template as a workbook within the Sentinel environment.

 **Implementation Evidence**

<img width="945" height="941" alt="workbook" src="https://github.com/user-attachments/assets/8bb864e7-5e15-4c6a-9b69-bf0716c9e46d" />


<img width="941" height="939" alt="key vault monitoring" src="https://github.com/user-attachments/assets/2e906329-c702-4b63-99c2-1fcc28c9680f" />


> Workbook implementation: [`docs/Workbooks.md`](docs/Workbooks.md)


## SOAR & Incident Automation

I implemented SOAR at two levels: native Microsoft Sentinel automation and an Azure Logic Apps playbook.

### Native Sentinel Automation

I created:

> **Contoso Sentinel Incident Triage**

The rule triggers:

```text
When incident is created
```

It automatically performs two initial triage actions:

1. Adds the `Automated-Triage` tag.
2. Creates the `Perform Initial SOC Investigation` task.

The task directs the analyst to review the incident alerts, affected entities, ATT&CK mappings, supporting telemetry, and related activity before deciding whether escalation, containment, or closure is appropriate.

I deliberately avoided automatically closing incidents or changing their severity without analyst validation.

### Azure Logic Apps Playbook

I then deployed:

> **MDTI-Automated-Triage**

The playbook was successfully deployed as an active Azure Logic App using the Consumption plan.

I validated:

- Microsoft Sentinel connection
- System-assigned managed identity
- Microsoft Sentinel Contributor RBAC assignment
- Logic App workflow structure
- Sentinel incident trigger
- Host entity extraction
- IP entity extraction
- Entity-processing workflow
- Threat-classification logic
- Successful execution through the visible workflow path

The automated workflow followed the pattern:

```text
Sentinel Incident
       │
       ▼
MDTI-Automated-Triage
       │
       ├── Get Hosts
       ├── Get IP Addresses
       ├── Process Entities
       ├── Initialize Classification
       └── Evaluate Malicious / Suspicious Logic
```

**Implementation Evidence**

<img width="1600" height="789" alt="automation rule" src="https://github.com/user-attachments/assets/f79f72dd-abd4-4b4e-b6da-3f16a9f9788c" />


<img width="944" height="940" alt="playbook" src="https://github.com/user-attachments/assets/df832274-9057-48b8-a31f-b416e069b318" />


<img width="1600" height="763" alt="logic app" src="https://github.com/user-attachments/assets/f2616d17-2501-4516-84c7-cc6bf1f3dd64" />


<img width="943" height="911" alt="logic run" src="https://github.com/user-attachments/assets/f3d87d3d-90f2-4001-a2b0-dc384e4f5697" />


> Complete SOAR implementation: [`docs/SOAR-Automation.md`](docs/SOAR-Automation.md)


## Implementation Validation

I validated each major component of the environment instead of considering deployment alone as proof that a feature was working.

| Component | Validation Performed | Status |
|---|---|---|
| Microsoft Sentinel | Log Analytics workspace successfully onboarded | Validated |
| Log Analytics | Telemetry confirmed through KQL queries | Validated |
| Azure Key Vault Diagnostics | Key Vault events identified in `AzureDiagnostics` | Validated |
| Data Connectors | Available Microsoft security integrations reviewed/configured | Configured |
| Content Hub | Relevant security solutions installed | Implemented |
| KQL | Queries executed against actual workspace telemetry | Validated |
| Analytics Rule | Microsoft Defender Threat Intelligence Analytics enabled | Implemented |
| Incident Management | Incident workflow and triage capabilities reviewed | Validated |
| Threat Hunting | Structured Hunt completed using three custom queries | Validated |
| MITRE ATT&CK | Detection coverage and rule mappings reviewed | Validated |
| Workbooks | Key Vault Security workbook populated with environment data | Validated |
| Automation Rule | Native incident-triage workflow created | Implemented |
| Logic App Playbook | `MDTI-Automated-Triage` deployed and active | Implemented |
| Managed Identity | System-assigned identity enabled | Validated |
| Sentinel RBAC | Microsoft Sentinel Contributor assigned | Implemented |
| SOAR Execution | Logic App run path executed successfully | Validated |
| UEBA | Administrative requirements investigated |  Limited |
| MDTI API Permission | `ThreatIntelligence.Read.All` not independently verified |  Limitation |

A detailed technical validation is available in:

[`docs/ValidationReport.md`](docs/ValidationReport.md)


## Key Results

The final implementation produced several measurable validation results.

### Log Ingestion

A seven-day workspace investigation identified:

```text
AzureMetrics       24 events
Usage               7 events
AzureDiagnostics    4 events
```

### Key Vault Investigation

The available diagnostic telemetry contained:

```text
Resource Provider: MICROSOFT.KEYVAULT
Resource:          KV-CONTOSO-PROD-001
Operation:         VaultGet
Result:            Success
Observed Events:   4
```

### Failed Operation Hunt

```text
Matching unsuccessful Key Vault operations: 0
```

This result was interpreted only within the available telemetry and selected investigation period.

### Threat Hunt

```text
Custom Hunting Queries: 3
Successful Key Vault Events: 4
Failed Operations Identified: 0
Escalated Findings: 0
```

### SOAR

```text
Native Automation Rule:     Implemented
Logic App Playbook:         Active
Managed Identity:           Enabled
Sentinel Contributor RBAC:  Assigned
Workflow Execution:         Successfully validated
```

## Troubleshooting & Challenges

Not every component worked on the first attempt. Troubleshooting became an important part of the implementation.

### 1. No Log Results in the Initial 24-Hour Window

My first workspace query returned no results when the investigation period was set to the last 24 hours.

I expanded the time range to **7 days**, after which the workspace returned telemetry from `AzureMetrics`, `Usage`, and `AzureDiagnostics`.

**Lesson:** A query returning zero results does not automatically mean data ingestion is broken. Time range should be validated before troubleshooting the ingestion pipeline.

### 2. KQL Resource Filter Returned No Results

While developing the Key Vault hunting query, an initial exact string comparison failed to return records that I knew existed.

I changed the comparison to the case-insensitive KQL operator:

```kusto
=~
```

For example:

```kusto
| where ResourceProvider =~ "MICROSOFT.KEYVAULT"
```

The expected Key Vault events were then returned.

**Lesson:** Understanding KQL comparison behavior is important when troubleshooting filters against real telemetry.

### 3. Key Vault Workbook Could Not Execute

When I initially opened the Azure Key Vault Security workbook, several panels displayed errors because required parameters were not configured.

I configured:

```text
Workspace = law-contoso-prod-001
KeyVault  = kv-contoso-prod-001
```

After refreshing the workbook, the visualizations populated successfully.

**Lesson:** Deploying or opening a workbook template does not mean it is automatically connected to the correct data sources.

### 4. UEBA Could Not Be Enabled

I investigated User and Entity Behavior Analytics (UEBA), but the tenant required administrative permissions that were not available in my environment.

I therefore documented UEBA as a limitation rather than claiming it as an implemented feature.

### 5. Logic App Deployment Required Additional Authorization

The `MDTI-Automated-Triage` playbook deployed successfully, but I did not treat deployment as proof that the automation was ready.

I separately validated:

- Microsoft Sentinel connector availability
- System-assigned managed identity
- Azure RBAC assignments
- Workflow execution

The managed identity initially had **0 Azure role assignments**.

I assigned:

```text
Microsoft Sentinel Contributor
```

at the `rg-contoso-prod-001` resource-group scope.

This followed least-privilege principles more closely than assigning the role across the entire subscription.

### 6. MDTI API Permission Verification

The playbook documentation referenced the `ThreatIntelligence.Read.All` permission.

I attempted to verify the playbook identity through Microsoft Entra Enterprise Applications but could not locate it through the interface available to me.

Although the visible Logic App workflow subsequently executed successfully, I did not independently verify this specific API permission.

I therefore recorded it as a project limitation rather than assuming it was configured.

> Full troubleshooting notes: [`docs/Troubleshooting.md`](docs/Troubleshooting.md)

## Project Limitations

This project represents a hands-on security operations environment, not a full enterprise SOC deployment.

The main limitations were:

- No virtual machines were deployed in the environment.
- No large endpoint fleet was onboarded.
- The available telemetry dataset was relatively small.
- The Sentinel incident queue contained no active incidents during the primary validation period.
- UEBA could not be enabled because of tenant administrative requirements.
- `ThreatIntelligence.Read.All` was not independently verified through the available Entra interface.
- The environment did not contain the volume and diversity of telemetry expected in a production SOC.
- The threat hunt did not identify evidence requiring escalation.

I intentionally retained these limitations in the documentation because they reflect the environment I actually implemented.

## What I Learned

This project changed how I think about SIEM and SOAR platforms.

Before building the environment, it was easy to view Sentinel as a collection of features: connectors, analytics rules, incidents, hunting, workbooks, and automation.

Working through the implementation showed me that those components are dependent on each other.

### Data Comes First

A detection or hunting query is only useful if the required telemetry is actually being collected.

A connector showing **Connected** is not enough. I learned to validate ingestion directly through the underlying logs.

### Zero Results Can Still Be a Result

My failed-operation hunt returned zero matches.

That did not mean the query failed. It meant the available telemetry did not contain events matching the hunting condition during the investigation period.

### Detection and Hunting Serve Different Purposes

Analytics rules continuously evaluate predefined conditions.

Threat hunting starts with an analyst hypothesis and proactively investigates telemetry.

Working with both helped me understand why mature SOCs need both capabilities.

### MITRE ATT&CK Is More Useful When Connected to Detection

Instead of only studying ATT&CK tactics and techniques theoretically, I used Sentinel to see how analytics rules contribute to ATT&CK detection coverage.

### Automation Requires Guardrails

I deliberately avoided automatically closing incidents or arbitrarily changing severity.

My native automation handled repetitive triage tasks while preserving analyst decision-making.

### Deployment Is Not Validation

This became one of the biggest lessons from the project.

I did not consider the Logic App complete simply because Azure reported a successful deployment.

I checked the connector, managed identity, RBAC configuration, workflow structure, and execution history.

## Skills Demonstrated

This project gave me hands-on experience across several security operations and cloud-security areas.

### Microsoft Sentinel

- Sentinel onboarding
- SIEM configuration
- Data connectors
- Content Hub
- Analytics rules
- Incident management
- Hunting
- Workbooks
- MITRE ATT&CK
- Automation rules
- Playbooks

### Kusto Query Language

- Log discovery
- Filtering
- Case-insensitive comparisons
- Field projection
- Result sorting
- Aggregation
- Activity-frequency analysis
- Security hunting

### Threat Hunting

- Hypothesis-driven investigation
- Custom hunting-query development
- Baseline analysis
- Failed-operation analysis
- Investigation closure
- Evidence-based escalation decisions

### Detection Engineering

- Analytics-rule deployment
- Threat-intelligence detection
- ATT&CK mapping
- Detection-coverage analysis

### SOAR

- Incident-triggered automation
- Automated tagging
- SOC investigation-task creation
- Azure Logic Apps
- Sentinel playbooks
- Entity extraction
- Automated triage
- Workflow validation

### Azure Security

- Log Analytics
- Azure Key Vault diagnostics
- Managed identities
- Azure RBAC
- Resource-group scoped permissions
- Azure Logic Apps

## Repository Structure

```text
microsoft-sentinel-siem-soar/
│
├── README.md
│
├── docs/
│   ├── Architecture.md
│   ├── DeploymentGuide.md
│   ├── DataConnectors.md
│   ├── KQLQueries.md
│   ├── AnalyticsRules.md
│   ├── IncidentManagement.md
│   ├── ThreatHunting.md
│   ├── MITRE-ATTACK-Mapping.md
│   ├── Workbooks.md
│   ├── SOAR-Automation.md
│   ├── ValidationReport.md
│   ├── Troubleshooting.md
│   └── LessonsLearned.md
│
├── queries/
│   ├── key-vault-activity.kql
│   ├── failed-key-vault-operations.kql
│   └── key-vault-activity-frequency.kql
│
└── images/
    ├── architecture/
    ├── deployment/
    ├── connectors/
    ├── analytics/
    ├── hunting/
    ├── mitre/
    ├── workbooks/
    └── automation/
```

## Technical Documentation

The README provides an overview of the project. Detailed implementation documentation is separated into the following guides:

| Document | Description |
|---|---|
| [`Architecture.md`](docs/Architecture.md) | Detailed architecture, components, and security data flow |
| [`DeploymentGuide.md`](docs/DeploymentGuide.md) | Sentinel deployment and configuration process |
| [`DataConnectors.md`](docs/DataConnectors.md) | Data integrations and Content Hub implementation |
| [`KQLQueries.md`](docs/KQLQueries.md) | KQL queries, results, and analysis |
| [`AnalyticsRules.md`](docs/AnalyticsRules.md) | Analytics and detection-rule configuration |
| [`IncidentManagement.md`](docs/IncidentManagement.md) | Sentinel incident-management workflow |
| [`ThreatHunting.md`](docs/ThreatHunting.md) | Complete Key Vault hunting investigation |
| [`MITRE-ATTACK-Mapping.md`](docs/MITRE-ATTACK-Mapping.md) | ATT&CK coverage analysis and detection mappings |
| [`Workbooks.md`](docs/Workbooks.md) | Azure Key Vault Security workbook implementation |
| [`SOAR-Automation.md`](docs/SOAR-Automation.md) | Native automation and Logic App playbook implementation |
| [`ValidationReport.md`](docs/ValidationReport.md) | Technical validation of implemented controls |
| [`Troubleshooting.md`](docs/Troubleshooting.md) | Issues encountered and resolutions |
| [`LessonsLearned.md`](docs/LessonsLearned.md) | Technical and operational lessons from the project |


## Future Improvements

If I extend this environment, the next improvements I would consider are:

- Onboard endpoint telemetry from Microsoft Defender for Endpoint.
- Add Microsoft Entra sign-in and audit telemetry where licensing permits.
- Expand the environment with additional Azure workloads.
- Develop custom scheduled analytics rules using KQL.
- Generate controlled test activity to validate alert-to-incident workflows.
- Develop additional hunting queries across identity, network, and endpoint telemetry.
- Implement entity mapping for users, hosts, and IP addresses.
- Expand MITRE ATT&CK coverage analysis.
- Enable UEBA where the required tenant permissions are available.
- Implement additional Logic App enrichment and notification workflows.
- Integrate a ticketing or case-management platform for SOC escalation.
- Build additional dashboards for incident trends and detection performance.

## Project Status

**Status: Completed **

The original project scope was:

```text
Microsoft Sentinel
├── SIEM
├── SOAR
├── Analytics Rules
├── Incidents
├── Workbooks
├── Threat Hunting
└── MITRE ATT&CK Mapping
```

All major areas were either **implemented and validated** or, where tenant permissions prevented implementation, explicitly investigated and documented.

The final environment demonstrates the complete security-operations relationship:

> **Collect → Analyze → Detect → Investigate → Hunt → Visualize → Automate → Validate**


## Final Thoughts

This project gave me a much clearer understanding of what operating Microsoft Sentinel involves beyond simply deploying a SIEM.

The most valuable part was connecting the individual capabilities together.

I collected telemetry and then verified it with KQL. I used that telemetry for threat hunting. I connected detections to MITRE ATT&CK. I visualized the same environment through Workbooks. I then moved into SOAR by creating native incident automation and deploying an Azure Logic Apps playbook.

Not everything worked immediately, and some capabilities were restricted by the tenant. Those limitations forced me to troubleshoot the environment and understand why something was not working instead of simply following a sequence of portal steps.

One principle guided the project from beginning to end:

> **If I didn't implement or validate it, I wouldn't claim that I did.**

That is also why the limitations and unsuccessful troubleshooting attempts are documented alongside the successful implementation.


## Connect

I'm continuing to build hands-on projects around cloud security, security operations, threat detection, SIEM/SOAR, and Microsoft security technologies.

If you found this project useful, feel free to explore the technical documentation and KQL queries included in this repository.

 **If this repository helped you or gave you ideas for your own Sentinel lab, consider starring it.**

### Technologies Used

`Microsoft Sentinel` • `Microsoft Defender` • `Microsoft Azure` • `Log Analytics` • `KQL` • `Azure Key Vault` • `MITRE ATT&CK` • `Azure Logic Apps` • `Microsoft Defender Threat Intelligence` • `Azure RBAC` • `Managed Identity` • `SIEM` • `SOAR`
