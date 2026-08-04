# Microsoft Sentinel SIEM & SOAR Architecture

## 1. Overview

This document describes the architecture of the Microsoft Sentinel SIEM and SOAR environment I implemented on Microsoft Azure.

The architecture was designed to demonstrate the core security operations lifecycle using Microsoft Sentinel as the central platform for security monitoring, detection, investigation, threat hunting, visualization, and automated response.

Rather than designing an architecture around resources that were not available in my Azure environment, I based the implementation on the services and telemetry I could actually deploy, configure, query, and validate.

The architecture therefore focuses on:

- Azure security telemetry
- Azure Key Vault diagnostic logging
- Azure Log Analytics
- Microsoft Sentinel
- Microsoft security integrations
- Kusto Query Language (KQL)
- Analytics and detection rules
- Incident management
- Threat hunting
- MITRE ATT&CK
- Security workbooks
- Microsoft Sentinel automation rules
- Azure Logic Apps
- Managed identities
- Azure Role-Based Access Control (RBAC)

The resulting architecture supports the following end-to-end security operations flow:

```text
Azure Resources
      |
      v
Diagnostic Telemetry
      |
      v
Azure Log Analytics
      |
      v
Microsoft Sentinel
      |
      +--------------------+
      |                    |
      v                    v
Log Analysis          Analytics Rules
      |                    |
      v                    v
Threat Hunting          Alerts
      |                    |
      v                    v
MITRE ATT&CK          Incidents
                           |
                           v
                    Automation Rules
                           |
                           v
                     Azure Logic Apps
                           |
                           v
                    Automated Triage
```


## 2. Architecture Diagram

The following diagram represents the implemented environment.

<img width="1536" height="1024" alt="ChatGPT Image Jul 31, 2026, 01_53_28 AM" src="https://github.com/user-attachments/assets/a3eaa5a8-1a84-4e65-bcb1-67974cfd217c" />


The diagram intentionally represents the environment I actually implemented.

I did not include virtual machines, endpoint fleets, firewalls, or other infrastructure that was not deployed as part of this project.


## 3. Core Architecture Components

The environment consists of several interconnected security and Azure components.

| Component | Purpose |
|---|---|
| Microsoft Azure | Cloud platform hosting the environment |
| Azure Key Vault | Monitored Azure resource and source of diagnostic telemetry |
| Azure Log Analytics | Centralized log storage and query platform |
| Microsoft Sentinel | SIEM and SOAR platform |
| Microsoft Defender Portal | Security operations interface for Sentinel capabilities |
| Data Connectors | Integration between Sentinel and supported Microsoft security services |
| Content Hub | Source of Sentinel solutions, analytics content, workbooks, and integrations |
| KQL | Query language used for log analysis and threat hunting |
| Analytics Rules | Automated security detection capability |
| Incidents | Centralized security investigation objects |
| Threat Hunting | Proactive investigation capability |
| MITRE ATT&CK | Detection coverage and attacker-behavior mapping |
| Workbooks | Security telemetry visualization |
| Automation Rules | Native Sentinel incident automation |
| Azure Logic Apps | Advanced SOAR workflow platform |
| Managed Identity | Identity used by the Logic App for Azure authorization |
| Azure RBAC | Authorization mechanism for the playbook |


# 4. Azure Resource Layer

The first layer of the architecture contains the Azure resources generating security and operational telemetry.

The primary resource used during the project was:

```text
Azure Key Vault
kv-contoso-prod-001
```

The Key Vault was particularly useful because diagnostic telemetry from the resource was available in the Log Analytics workspace.

This allowed me to investigate actual Azure resource activity instead of relying entirely on synthetic or sample security logs.

The Key Vault telemetry included information such as:

```text
TimeGenerated
ResourceGroup
ResourceProvider
Resource
ResourceType
OperationName
ResultType
```

During validation, I identified events associated with:

```text
Resource Provider: MICROSOFT.KEYVAULT
Resource:          KV-CONTOSO-PROD-001
Operation:         VaultGet
Result:            Success
```

These events later became the basis for the KQL and threat-hunting portions of the project.

# 5. Telemetry Collection Layer

Azure resources generate operational and security telemetry that can be forwarded to monitoring platforms through diagnostic settings and supported integrations.

For this environment, relevant telemetry was available within the Log Analytics workspace:

```text
law-contoso-prod-001
```

The workspace was located within:

```text
rg-contoso-prod-001
```

The telemetry pipeline can be represented as:

```text
Azure Key Vault
      |
      | Diagnostic Telemetry
      v
Log Analytics Workspace
law-contoso-prod-001
```

I validated the presence of telemetry directly using KQL rather than relying solely on the status of data connectors.

A workspace-wide query identified data within:

```text
AzureMetrics
Usage
AzureDiagnostics
```

`AzureDiagnostics` contained the Key Vault activity used throughout the investigation.


# 6. Log Analytics Layer

Azure Log Analytics provides the centralized telemetry store supporting Microsoft Sentinel.

Microsoft Sentinel does not replace Log Analytics as the underlying log repository. Instead, Sentinel adds security analytics, detection, investigation, threat hunting, visualization, and automation capabilities on top of the telemetry stored within the workspace.

The relationship in this environment was:

```text
Azure Resources
      |
      v
Diagnostic Logs
      |
      v
law-contoso-prod-001
      |
      v
Microsoft Sentinel
```

I onboarded the existing `law-contoso-prod-001` workspace to Microsoft Sentinel rather than creating a separate workspace solely for the project.

This allowed the existing Azure telemetry to become available for Sentinel security operations.


# 7. Microsoft Sentinel Layer

Microsoft Sentinel acts as the central security operations component of the architecture.

Within this environment, Sentinel provided several capabilities:

```text
Microsoft Sentinel
|
+-- Data Connectors
|
+-- Content Hub
|
+-- Logs / KQL
|
+-- Analytics
|
+-- Incidents
|
+-- Hunting
|
+-- MITRE ATT&CK
|
+-- Workbooks
|
+-- Automation
|
+-- Playbooks
```

These capabilities are interconnected rather than independent.

For example:

```text
Telemetry
    |
    v
KQL / Analytics
    |
    v
Alert
    |
    v
Incident
    |
    v
Automation
```

At the same time, analysts can investigate the same underlying telemetry independently through:

```text
Logs
 |
 v
Threat Hunting
 |
 v
Investigation
```


# 8. Data Connector Architecture

Microsoft Sentinel Data Connectors provide integrations between Sentinel and supported Azure and Microsoft security services.

The available connector set in the environment included services such as:

- Azure Key Vault
- Microsoft Defender for Cloud
- Microsoft Defender for Endpoint
- Microsoft Defender for Identity
- Microsoft Defender for Office
- Microsoft Defender XDR
- Microsoft Entra ID Protection
- Microsoft 365 Insider Risk Management

The connector layer can be represented as:

```text
Microsoft Security Services
           |
           v
    Data Connectors
           |
           v
   Microsoft Sentinel
```

A connector reporting a connected state was not treated as sufficient evidence of log ingestion.

I independently queried the Log Analytics workspace to determine whether telemetry was actually present.

This separation between connector configuration and ingestion validation was an important architecture and operational consideration throughout the project.


# 9. Content Hub Architecture

Microsoft Sentinel Content Hub provides packaged security content for specific Microsoft services, technologies, and security scenarios.

I installed content relevant to the environment, including:

```text
Microsoft Defender for Cloud
Azure Key Vault
Azure Activity
Threat Intelligence
```

Content Hub contributed components such as:

- Analytics templates
- Workbook templates
- Threat-intelligence integrations
- Security content
- Automation content

The architecture relationship is:

```text
Content Hub
     |
     v
Installed Solutions
     |
     +-- Analytics Content
     |
     +-- Workbooks
     |
     +-- Connectors
     |
     +-- Automation Content
     |
     v
Microsoft Sentinel
```

I avoided installing large amounts of unrelated content simply to increase the number of available templates.


# 10. KQL Analysis Architecture

Kusto Query Language was the primary mechanism I used to interact directly with the collected telemetry.

The basic query architecture was:

```text
Azure Resource
     |
     v
Diagnostic Telemetry
     |
     v
Log Analytics
     |
     v
KQL Query
     |
     v
Security Analysis
```

I used KQL for three primary purposes:

### Activity Monitoring

Identify Key Vault operations within the available telemetry.

### Failure Analysis

Search for unsuccessful Key Vault operations that could warrant further investigation.

### Activity Frequency Analysis

Summarize observed operations to establish a basic activity pattern.

These queries later became reusable components of the structured Sentinel Hunt.

# 11. Detection Architecture

The detection layer converts collected telemetry into security detections.

I enabled:

```text
Microsoft Defender Threat Intelligence Analytics
```

The rule was configured with:

```text
Severity: Medium
Status: Enabled

MITRE ATT&CK:
- Persistence
- Lateral Movement
```

The detection architecture follows:

```text
Security Telemetry
       |
       v
Microsoft Sentinel Analytics
       |
       v
Analytics Rule
       |
       v
Alert
       |
       v
Incident
```

This separation is important.

An analytics rule defines the detection logic.

An alert represents a detection generated when relevant conditions are met.

An incident provides the investigation object used by security analysts to manage related security activity.


# 12. Incident Management Architecture

Microsoft Sentinel incidents provide the central investigation layer for detected security activity.

The incident lifecycle can be represented as:

```text
Analytics Rule
      |
      v
Security Alert
      |
      v
Sentinel Incident
      |
      +--------------------+
      |                    |
      v                    v
SOC Investigation     Automation Rule
                           |
                           v
                    Automated Actions
```

During the main validation period, the environment contained no active incidents requiring investigation.

I did not artificially classify benign activity as malicious simply to populate the incident dashboard.

Instead, I configured the architecture so that future incidents can enter the automated triage workflow.

# 13. Threat Hunting Architecture

Threat hunting provides a proactive investigation path independent of automated detection.

The hunting architecture follows:

```text
Security Hypothesis
       |
       v
Custom KQL Queries
       |
       v
Telemetry Investigation
       |
       v
Result Analysis
       |
       +--------------------+
       |                    |
       v                    v
Evidence Found        No Supporting Evidence
       |                    |
       v                    v
Escalate             Close / Invalidate
```

I created the following Hunt:

```text
Azure Key Vault Suspicious Activity Investigation
```

The Hunt contained three custom queries:

```text
Key Vault Activity Monitoring

Failed Key Vault Operations

Key Vault Activity Frequency Analysis
```

The available telemetry showed four successful Key Vault events and no unsuccessful operations matching the failure query.

The investigation did not produce evidence requiring escalation.

This demonstrated the complete hypothesis-driven hunting process without artificially manufacturing a malicious result.


# 14. MITRE ATT&CK Architecture

MITRE ATT&CK provides a behavioral framework for understanding detection coverage.

Within Microsoft Sentinel, I used the ATT&CK matrix to review how available detections mapped to attacker tactics and techniques.

The architecture relationship was:

```text
Analytics Rule
      |
      v
Detection Logic
      |
      v
ATT&CK Mapping
      |
      +-- Persistence
      |
      +-- Lateral Movement
```

The enabled Microsoft Defender Threat Intelligence Analytics rule contributed coverage associated with:

```text
Persistence
Lateral Movement
```

The ATT&CK matrix therefore provided a higher-level view of how individual detections contribute to security coverage.

# 15. Security Visualization Architecture

Microsoft Sentinel Workbooks provide visualization capabilities over collected security telemetry.

I configured the:

```text
Azure Key Vault Security Workbook
```

against:

```text
Workspace:
law-contoso-prod-001

Key Vault:
kv-contoso-prod-001
```

The data flow was:

```text
Azure Key Vault
      |
      v
Diagnostic Logs
      |
      v
Log Analytics
      |
      v
Sentinel Workbook
      |
      v
Security Visualization
```

The workbook provided views covering areas such as:

- Diagnostic-log coverage
- Key Vault activity
- Request metrics
- Monitoring timelines
- Latency
- Service behavior

This provided a visual monitoring layer over the same telemetry I had previously investigated using KQL.


# 16. Native SOAR Architecture

The first SOAR layer was implemented using a native Microsoft Sentinel automation rule.

I created:

```text
Contoso Sentinel Incident Triage
```

The trigger was:

```text
When incident is created
```

The rule performs two initial actions:

```text
1. Add tag:
   Automated-Triage

2. Add task:
   Perform Initial SOC Investigation
```

The architecture is:

```text
New Sentinel Incident
         |
         v
Contoso Sentinel Incident Triage
         |
         +-------------------------+
         |                         |
         v                         v
Add Automated-Triage       Create Investigation
Tag                        Task
         |                         |
         +------------+------------+
                      |
                      v
               SOC Analyst Review
```

I deliberately avoided automatically closing incidents or changing their severity.

The automation handles repetitive triage preparation while retaining human decision-making for security conclusions.

# 17. Azure Logic Apps SOAR Architecture

The second automation layer uses Azure Logic Apps.

I deployed:

```text
MDTI-Automated-Triage
```

as a Microsoft Sentinel playbook.

The playbook processes Sentinel incident entities through an automated threat-intelligence triage workflow.

The high-level architecture is:

```text
Microsoft Sentinel Incident
           |
           v
MDTI-Automated-Triage
           |
           v
Sentinel Incident Trigger
           |
           +-------------------+
           |                   |
           v                   v
       Get Hosts            Get IPs
           |                   |
           v                   v
     Process Hosts        Process IPs
           |                   |
           +---------+---------+
                     |
                     v
             Threat Classification
                     |
                     v
           Malicious / Suspicious
                 Evaluation
                     |
                     v
              Sentinel Actions
```

This extended the project beyond native automation into a full Azure workflow orchestration platform.


# 18. Managed Identity Architecture

The `MDTI-Automated-Triage` Logic App uses a system-assigned managed identity.

The identity architecture follows:

```text
MDTI-Automated-Triage
       |
       v
System-Assigned
Managed Identity
       |
       v
Azure RBAC
       |
       v
Microsoft Sentinel
```

The system-assigned identity was enabled successfully.

When I initially reviewed the identity's Azure access, it had:

```text
0 role assignments
```

I assigned:

```text
Microsoft Sentinel Contributor
```

at the following scope:

```text
rg-contoso-prod-001
```

I selected resource-group scope rather than subscription-wide scope to avoid granting broader access than required for the project.


# 19. SOAR Execution Flow

The final automated security operations flow can be represented as:

```text
                         SECURITY TELEMETRY
                                |
                                v
                     +----------------------+
                     |    Log Analytics     |
                     +----------+-----------+
                                |
                                v
                     +----------------------+
                     | Microsoft Sentinel   |
                     +----------+-----------+
                                |
                         Detection / Alert
                                |
                                v
                     +----------------------+
                     |      Incident        |
                     +----------+-----------+
                                |
                  +-------------+-------------+
                  |                           |
                  v                           v
        Native Automation             Logic App Playbook
                  |                           |
                  v                           v
       Automated-Triage Tag          Extract Host Entities
                  |                           |
                  v                           v
       Investigation Task            Extract IP Entities
                                              |
                                              v
                                      Process Entities
                                              |
                                              v
                                      Classification Logic
                                              |
                                              v
                                       Sentinel Response
```

This architecture provides both:

```text
Native Sentinel Automation
```

and:

```text
Azure Logic Apps-based SOAR
```

within the same security operations environment.

# 20. Security Design Decisions

Several design decisions were made deliberately during implementation.

## Existing Log Analytics Workspace

I used the existing `law-contoso-prod-001` workspace rather than creating an unnecessary second workspace.

This allowed Sentinel to operate directly against telemetry already associated with the Azure environment.

## Resource-Group Scoped RBAC

I assigned Microsoft Sentinel Contributor to the Logic App managed identity at the resource-group level rather than across the entire subscription.

This reduced unnecessary permission scope.

## Analyst-Controlled Incident Decisions

The native automation rule does not automatically:

```text
Close incidents
Change incident severity
Declare activity malicious
```

These decisions remain with the analyst.

## Evidence-Based Hunting

The threat hunt was concluded based on the available telemetry.

Because the queries did not identify evidence requiring escalation, I did not manufacture an incident or suspicious finding.

## Architecture Reflects Actual Resources

I intentionally excluded components that were not deployed.

For example, the architecture does not include a virtual machine simply because VMs commonly appear in Sentinel reference architectures.

The documentation represents the environment that was actually built.

# 21. Architecture Limitations

The architecture has several limitations that should be considered when interpreting the implementation.

### Limited Telemetry Volume

The environment contained significantly less telemetry than a production SOC.

### No Endpoint Fleet

No large endpoint environment was onboarded into Microsoft Defender for Endpoint as part of this implementation.

### No Virtual Machine Workloads

The Azure environment did not contain virtual machines used as telemetry sources for the project.

### UEBA

User and Entity Behavior Analytics could not be enabled because of tenant administrative requirements.

### MDTI Permission Verification

The MDTI playbook documentation referenced:

```text
ThreatIntelligence.Read.All
```

I was unable to independently verify this permission through the available Microsoft Entra interface.

The limitation is therefore retained in the documentation even though the visible Logic App workflow path executed successfully.


# 22. Architecture Validation

I validated the architecture at multiple layers.

| Layer | Validation |
|---|---|
| Azure Resource | Key Vault available and generating telemetry |
| Log Collection | Key Vault events identified in `AzureDiagnostics` |
| Log Analytics | Workspace queries returned telemetry |
| Sentinel | Workspace successfully onboarded |
| Detection | Threat Intelligence analytics rule enabled |
| Hunting | Three custom queries executed |
| ATT&CK | Detection mappings reviewed |
| Visualization | Key Vault workbook populated |
| Native SOAR | Incident automation rule configured |
| Logic Apps | MDTI playbook deployed and active |
| Identity | System-assigned managed identity enabled |
| RBAC | Sentinel Contributor assigned |
| Execution | Visible automated-triage workflow executed successfully |


# 23. Final Architecture

The completed implementation can be summarized as:

```text
Azure Resources
      |
      v
Diagnostic Telemetry
      |
      v
Azure Log Analytics
      |
      v
Microsoft Sentinel
      |
      +-----------------------+
      |                       |
      v                       v
Detection                Threat Hunting
      |                       |
      v                       v
MITRE ATT&CK              KQL Analysis
      |
      v
Incident
      |
      +---------------------------+
      |                           |
      v                           v
Native Automation          Azure Logic Apps
      |                           |
      v                           v
Initial SOC Triage         MDTI Automated Triage
      |                           |
      +-------------+-------------+
                    |
                    v
             Analyst Investigation
```

The architecture demonstrates how Azure telemetry can move from raw diagnostic data into a security operations workflow involving collection, detection, investigation, threat hunting, visualization, ATT&CK mapping, and automated response.


## Related Documentation

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
- [Troubleshooting](Troubleshooting.md)
- [Lessons Learned](LessonsLearned.md)
