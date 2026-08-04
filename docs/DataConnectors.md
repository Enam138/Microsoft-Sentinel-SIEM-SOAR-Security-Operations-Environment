# Microsoft Sentinel Data Connectors and Content Hub

## 1. Overview

This document describes the data integration and Microsoft Sentinel Content Hub configuration used in the project.

One of the main lessons from this stage of the implementation was the distinction between three different states:

```text
Connector Available
        |
        v
Connector Connected
        |
        v
Telemetry Actually Present
```

These are not necessarily the same thing.

During the project, several Microsoft security connectors showed as connected. However, I did not assume that a connected status meant that each service was actively generating searchable security telemetry.

I therefore validated available data separately through the underlying Log Analytics workspace using Kusto Query Language (KQL).

This approach allowed me to distinguish between:

- Integrations available in the tenant
- Integrations showing a connected state
- Content installed through Content Hub
- Telemetry actually observed in Log Analytics

# 2. Integration Architecture

The data integration layer sits between Azure/Microsoft security services and Microsoft Sentinel.

```text
Azure / Microsoft Security Services
                |
                v
        Data Connectors
                |
                v
       Log Analytics Workspace
       law-contoso-prod-001
                |
                v
        Microsoft Sentinel
                |
      +---------+---------+
      |         |         |
      v         v         v
  Analytics   Hunting   Workbooks
```

Microsoft Sentinel uses the telemetry available through the underlying workspace to support detection, investigation, hunting, visualization, and automation.

# 3. Log Analytics Workspace

The Microsoft Sentinel deployment used the existing Log Analytics workspace:

```text
law-contoso-prod-001
```

Resource group:

```text
rg-contoso-prod-001
```

I onboarded this existing workspace to Microsoft Sentinel rather than creating a separate workspace.

The workspace became the central telemetry repository supporting the Sentinel implementation.


# 4. Available Data Connectors

When I reviewed Microsoft Sentinel Data Connectors, the environment displayed eight available connectors.

These were:

| Data Connector | Observed Status |
|---|---|
| Azure Key Vault | Connected |
| Microsoft 365 Insider Risk Management | Connected |
| Microsoft Defender for Cloud | Connected |
| Microsoft Defender for Endpoint | Connected |
| Microsoft Defender for Identity | Connected |
| Microsoft Defender for Office | Connected |
| Microsoft Defender XDR | Connected |
| Microsoft Entra ID Protection | Connected |

The connector status confirmed that the integrations were available/configured within the Sentinel environment.

However, I treated this only as configuration evidence.

I separately queried Log Analytics to determine what telemetry was actually available for investigation.

# 5. Azure Key Vault Integration

Azure Key Vault became the most important telemetry source for the hands-on investigation.

The monitored Key Vault was:

```text
kv-contoso-prod-001
```

Relevant diagnostic telemetry was available in:

```text
AzureDiagnostics
```

The relationship was:

```text
Azure Key Vault
kv-contoso-prod-001
        |
        v
Diagnostic Telemetry
        |
        v
AzureDiagnostics
        |
        v
law-contoso-prod-001
        |
        v
Microsoft Sentinel
```

I validated this integration directly through KQL.


# 6. Validating Available Workspace Data

Instead of relying only on connector status, I queried the workspace to identify tables containing telemetry.

I used:

```kusto
search *
| summarize EventCount = count() by $table
| order by EventCount desc
```

The first query was executed using a 24-hour time range.

No useful results were returned.

I then expanded the investigation window to:

```text
Last 7 days
```

The workspace returned data from:

| Table | Observed Event Count |
|---|---:|
| `AzureMetrics` | 24 |
| `Usage` | 7 |
| `AzureDiagnostics` | 4 |

This confirmed that telemetry was available even though it had not appeared within the initial 24-hour investigation window.


# 7. Validating Key Vault Diagnostic Telemetry

I investigated the `AzureDiagnostics` table directly:

```kusto
AzureDiagnostics
| order by TimeGenerated desc
| take 50
```

The results contained telemetry associated with Azure Key Vault.

Observed fields included:

```text
TimeGenerated
ResourceGroup
ResourceProvider
Resource
ResourceType
OperationName
ResultType
```

The relevant records showed:

```text
ResourceGroup:
RG-CONTOSO-PROD-001

ResourceProvider:
MICROSOFT.KEYVAULT

Resource:
KV-CONTOSO-PROD-001

ResourceType:
VAULTS

OperationName:
VaultGet

ResultType:
Success
```

Four Key Vault events were available during the selected investigation period.

This provided direct evidence that Key Vault telemetry was reaching the workspace.


# 8. Connected Does Not Automatically Mean Data Is Present

This distinction became important during the project.

The Data Connectors page showed several Microsoft security integrations as connected.

However, I did not independently observe or validate active telemetry from every one of those eight services.

For example, the project did not contain a large endpoint fleet generating Microsoft Defender for Endpoint telemetry.

It would therefore be inaccurate to state that every connected integration produced security events used during the investigation.

The accurate implementation state was:

```text
Connector configuration:
Validated through Sentinel interface

Key Vault telemetry:
Validated through Log Analytics

Telemetry from every connected service:
Not independently validated
```

This distinction is retained throughout the project documentation.


# 9. Microsoft Defender Integrations

Several Microsoft Defender integrations were available in the Sentinel environment.

These included:

```text
Microsoft Defender for Cloud

Microsoft Defender for Endpoint

Microsoft Defender for Identity

Microsoft Defender for Office

Microsoft Defender XDR
```

These connectors provide the architecture for bringing alerts and security information from the broader Microsoft Defender ecosystem into Sentinel.

Conceptually:

```text
Microsoft Defender Services
           |
           v
Sentinel Data Connectors
           |
           v
Microsoft Sentinel
           |
           v
Centralized Security Operations
```

The presence of these integrations allowed me to explore how Sentinel fits into Microsoft's wider security ecosystem.

However, the project did not attempt to generate or validate telemetry across every Defender product.


# 10. Microsoft Entra ID Protection

Microsoft Entra ID Protection was also present among the available connectors.

Identity telemetry is particularly valuable in a SIEM because many security investigations involve:

- Authentication activity
- Identity risk
- Privileged access
- Suspicious sign-ins
- Account compromise
- Credential misuse

The connector was available within the environment, but identity-focused investigation was not the primary telemetry source used in this project.

The hands-on investigation remained centered on Azure Key Vault diagnostic activity.


# 11. Microsoft 365 Insider Risk Management

Microsoft 365 Insider Risk Management was also included among the eight available integrations.

This type of connector can contribute information related to insider-risk scenarios within Microsoft 365 environments.

It was present within the Sentinel integration layer but was not used as a primary telemetry source for the custom KQL threat hunt.


# 12. Content Hub

After reviewing the available connectors, I moved to Microsoft Sentinel Content Hub.

Content Hub contained approximately:

```text
490 solutions
```

The large catalog includes security content for many technologies and security scenarios.

I did not install every available package.

Instead, I selected content relevant to the Azure environment and the security operations objectives of the project.


# 13. Installed Content Hub Solutions

The solutions I installed were:

```text
Microsoft Defender for Cloud

Azure Key Vault

Azure Activity

Threat Intelligence
```

These solutions aligned with the resources and Sentinel capabilities I intended to use.

The Content Hub relationship can be represented as:

```text
Microsoft Sentinel Content Hub
             |
             v
       Installed Solutions
             |
     +-------+-------+
     |       |       |
     v       v       v
Analytics  Workbooks Integrations
     |
     v
Security Operations
```


# 14. Microsoft Defender for Cloud Solution

I installed the Microsoft Defender for Cloud solution because Defender for Cloud was part of the existing Azure security environment.

This allowed Sentinel to incorporate relevant security content associated with Azure cloud-security monitoring.

The integration contributes to the broader architecture:

```text
Azure Resources
      |
      v
Microsoft Defender for Cloud
      |
      v
Microsoft Sentinel
```

This supports centralized visibility across cloud security and security operations.


# 15. Azure Key Vault Solution

The Azure Key Vault solution was directly relevant because:

```text
kv-contoso-prod-001
```

was already deployed and generating diagnostic telemetry.

The solution also provided access to the Azure Key Vault Security workbook used later in the project.

This created a direct relationship between:

```text
Azure Key Vault
      |
      v
Diagnostic Telemetry
      |
      v
Log Analytics
      |
      v
Sentinel Content
      |
      v
Key Vault Security Workbook
```

This was one of the strongest integrations in the project because both the raw telemetry and the resulting visualization were validated.


# 16. Azure Activity Solution

I originally looked for Microsoft Entra ID content during Content Hub configuration.

The required Entra solution was not available in the environment in the way I expected.

Rather than documenting a solution I had not installed, I selected:

```text
Azure Activity
```

instead.

Azure Activity content was relevant because the project focused on monitoring and investigating Azure resources.

This decision reflects the actual tenant capabilities available during implementation.

# 17. Threat Intelligence Solution

I installed the Threat Intelligence solution to support the detection and automation portions of the project.

Threat intelligence became relevant in two major areas.

First, I enabled:

```text
Microsoft Defender Threat Intelligence Analytics
```

as an active Sentinel analytics rule.

Later, I deployed:

```text
MDTI-Automated-Triage
```

as an Azure Logic Apps-based Sentinel playbook.

The relationship was:

```text
Threat Intelligence Content
          |
          +-----------------------+
          |                       |
          v                       v
   Analytics Rule          MDTI Playbook
          |                       |
          v                       v
      Detection             Automated Triage
```

This connected Content Hub installation to both SIEM detection and SOAR automation.

# 18. Content Hub and Hunting

When I initially opened Sentinel Hunting, the environment showed:

```text
0 hunting queries
```

There was no visible built-in hunting-query catalog in the Hunting interface at that point.

This prompted me to investigate Content Hub and install relevant solutions.

Rather than depending entirely on prebuilt hunting content, I later created my own Hunt and three custom KQL queries based on the Key Vault telemetry available in the environment.

This resulted in:

```text
Content Hub
     |
     v
Relevant Sentinel Content
     |
     +
     |
Custom KQL Development
     |
     v
Structured Threat Hunt
```

# 19. Content Hub and Workbooks

Content Hub also played an important role in the visualization portion of the project.

After installing the relevant solutions, the following workbook templates were available:

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

because it directly corresponded to a resource generating telemetry in the environment.

After configuring the required workspace and Key Vault parameters, the workbook successfully populated with monitoring information.

# 20. Content Hub and Analytics

Content Hub also contributed analytics content.

The environment provided templates including:

```text
Anomalous RDP Login Detection

Anomalous SSH Login Detection

Microsoft Defender Threat Intelligence Analytics
```

Because the environment did not contain virtual machines being monitored for RDP or SSH activity, I selected:

```text
Microsoft Defender Threat Intelligence Analytics
```

rather than enabling unrelated rules.

This maintained alignment between:

```text
Installed Content
      |
      v
Available Detection
      |
      v
Actual Environment
```

# 21. Integration Validation Strategy

I used several levels of validation during the data-integration phase.

## Level 1: Connector Status

I confirmed that the available Sentinel connectors showed a connected state.

This validated the configuration layer.

## Level 2: Content Installation

I verified that relevant Content Hub solutions were installed.

This validated the content layer.

## Level 3: Log Analytics Query

I queried the underlying workspace directly.

This validated the telemetry layer.

## Level 4: Resource Identification

I confirmed that `AzureDiagnostics` contained events from:

```text
kv-contoso-prod-001
```

This validated the resource telemetry.

## Level 5: Security Use

I reused the telemetry for:

```text
KQL Analysis
Threat Hunting
Workbooks
```

This validated that the data was usable for security operations rather than merely present in storage.

# 22. Validation Model

The final validation model can be summarized as:

```text
Connector Shows Connected
          |
          v
Content Installed
          |
          v
Workspace Queried
          |
          v
Telemetry Found
          |
          v
Resource Identified
          |
          v
Security Analysis Performed
```

For Azure Key Vault, this complete validation chain was achieved.

# 23. Evidence Collected

The evidence retained for the integration portion of the project includes screenshots showing:

```text
Microsoft Sentinel data connectors

Connector status

Content Hub

Installed Sentinel solutions

Log Analytics query results

AzureDiagnostics telemetry

Azure Key Vault diagnostic events
```

Recommended repository locations:

```text
images/
|
+-- connectors/
|   |
|   +-- sentinel-data-connectors.png
|   +-- content-hub-solutions.png
|
+-- hunting/
    |
    +-- log-analytics-data-tables.png
    +-- key-vault-diagnostic-events.png
```

The exact filenames can be adjusted to match the screenshots stored in the repository.


# 24. Data Integration Limitations

Several limitations should be considered when interpreting the connector configuration.

## Not Every Connector Was Independently Tested

Although eight connectors showed as connected, I did not generate and validate telemetry from every associated service.

## Small Telemetry Dataset

The workspace contained a relatively small amount of telemetry compared with a production SOC.

## No Endpoint Fleet

The project did not include a large Defender for Endpoint deployment generating continuous endpoint-security events.

## Limited Identity Investigation

Identity-focused telemetry was not the primary source used for the custom threat hunt.

## Tenant Capabilities

The available integrations and Content Hub options were influenced by the permissions, services, and licensing available in the Azure tenant.


# 25. Security Considerations

The data-integration architecture highlighted several security considerations.

## Verify Data Rather Than Trusting Status

A connected connector should not be considered sufficient proof that the expected security telemetry is available.

## Monitor Data Coverage

A SOC should understand which data sources are actively producing logs and which integrations are configured but inactive.

## Install Relevant Content

Content Hub should be aligned with the technologies actually deployed in the environment.

Installing large quantities of unused detection content can create unnecessary complexity.

## Validate Time Ranges

A zero-result query should be investigated carefully.

In this project, expanding the time range from 24 hours to seven days revealed telemetry that was already present in the workspace.

# 26. Final Data Integration State

At the end of this phase, the environment had:

```text
8 available/connected Microsoft security integrations

4 relevant Content Hub solutions installed

3 Log Analytics tables observed with data

4 Azure Key Vault diagnostic events identified

1 validated Azure Key Vault telemetry source

Reusable telemetry for:
- KQL analysis
- Threat hunting
- Security workbooks
```

The most important result was not simply that Sentinel showed connected services.

It was that I could trace actual Azure telemetry from the monitored resource into Log Analytics and then use that data within Microsoft Sentinel.

The validated flow was:

```text
Azure Key Vault
      |
      v
Diagnostic Telemetry
      |
      v
AzureDiagnostics
      |
      v
Log Analytics
      |
      v
Microsoft Sentinel
      |
      +----------------+
      |                |
      v                v
KQL Analysis      Threat Hunting
      |
      v
Workbooks
```

## Related Documentation

- [Architecture](Architecture.md)
- [Deployment Guide](DeploymentGuide.md)
- [KQL Queries](KQLQueries.md)
- [Analytics Rules](AnalyticsRules.md)
- [Threat Hunting](ThreatHunting.md)
- [Workbooks](Workbooks.md)
- [SOAR Automation](SOAR-Automation.md)
- [Validation Report](ValidationReport.md)
- [Troubleshooting](Troubleshooting.md)
