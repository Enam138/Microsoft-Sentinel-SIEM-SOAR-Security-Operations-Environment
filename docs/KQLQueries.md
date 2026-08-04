# KQL Queries and Log Analysis

## 1. Overview

This document contains the Kusto Query Language (KQL) queries I used to validate telemetry, investigate Azure Key Vault activity, and support the threat-hunting phase of the Microsoft Sentinel project.

Rather than beginning with complex detection queries, I followed a progressive investigation process:

```text
Confirm Data Exists
        |
        v
Identify Available Tables
        |
        v
Inspect AzureDiagnostics
        |
        v
Identify Key Vault Telemetry
        |
        v
Filter Key Vault Activity
        |
        v
Search for Failed Operations
        |
        v
Analyze Activity Frequency
        |
        v
Reuse Queries in Sentinel Hunt
```

The queries were executed against:

```text
Log Analytics Workspace:
law-contoso-prod-001
```

The primary security telemetry used during the investigation was stored in:

```text
AzureDiagnostics
```

and originated from:

```text
Azure Key Vault:
kv-contoso-prod-001
```

# 2. Why KQL Was Important to the Project

Microsoft Sentinel provides dashboards, analytics rules, incidents, hunting capabilities, and workbooks.

However, all of those capabilities depend on telemetry.

KQL gave me a way to interact directly with that telemetry and answer questions such as:

```text
Is data actually reaching the workspace?

Which tables currently contain events?

Is Azure Key Vault generating diagnostic logs?

What Key Vault operations occurred?

Were any observed operations unsuccessful?

Which operations appeared most frequently?
```

This made KQL both a validation tool and an investigation tool throughout the project.


# 3. Investigation Environment

The relevant resources were:

| Component | Value |
|---|---|
| Log Analytics Workspace | `law-contoso-prod-001` |
| Resource Group | `rg-contoso-prod-001` |
| Monitored Resource | `kv-contoso-prod-001` |
| Resource Type | Azure Key Vault |
| Primary Table | `AzureDiagnostics` |
| Query Language | Kusto Query Language |
| Primary Investigation Window | Last 7 days |

The seven-day investigation period became important because my initial 24-hour search did not return the telemetry I expected.

# 4. Initial Workspace Data Discovery

Before writing resource-specific queries, I first needed to understand what data existed in the workspace.

I used:

```kusto
search *
| summarize EventCount = count() by $table
| order by EventCount desc
```

## Purpose

This query searches the available workspace data and summarizes the number of records by table.

The query allowed me to answer:

> Which Log Analytics tables currently contain data?

---

# 5. Understanding the Discovery Query

The query can be broken down into three stages.

### Search the Workspace

```kusto
search *
```

This searches across the available workspace data.

### Count Events by Table

```kusto
| summarize EventCount = count() by $table
```

This groups the records by their source table and calculates the number of events found in each table.

### Sort the Results

```kusto
| order by EventCount desc
```

This sorts the tables from the highest observed event count to the lowest.

# 6. Initial 24-Hour Result

My first attempt used the:

```text
Last 24 hours
```

time range.

The query returned no useful results.

At this stage, several explanations were possible:

```text
No telemetry was being generated

Telemetry ingestion was delayed

The connectors were not producing data

The selected time range did not contain the available events
```

Rather than immediately changing the Sentinel configuration, I first changed the investigation timeframe.

# 7. Expanding the Time Range

I changed the Log Analytics time range to:

```text
Last 7 days
```

and executed the same query again.

This time, the workspace returned data.

The observed results included:

| Table | Event Count |
|---|---:|
| `AzureMetrics` | 24 |
| `Usage` | 7 |
| `AzureDiagnostics` | 4 |

This was an important troubleshooting result.

The ingestion pipeline was not necessarily broken.

The initial investigation window simply did not contain the available telemetry.

# 8. Why AzureDiagnostics Became the Focus

Of the tables returned, `AzureDiagnostics` was particularly useful because it contained resource-level diagnostic telemetry.

I therefore moved from workspace discovery to table-specific investigation.

I used:

```kusto
AzureDiagnostics
| order by TimeGenerated desc
| take 50
```

## Purpose

This query retrieves recent records from `AzureDiagnostics` and sorts them from newest to oldest.

It allowed me to inspect the actual schema and determine which Azure resources were represented in the telemetry.

# 9. Key Vault Telemetry Discovery

The `AzureDiagnostics` results showed records associated with Azure Key Vault.

Relevant observed values included:

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

This confirmed that diagnostic telemetry from:

```text
kv-contoso-prod-001
```

was available for investigation.

The basic telemetry flow was therefore validated:

```text
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
KQL Investigation
```

# 10. Relevant AzureDiagnostics Fields

The investigation primarily used the following fields:

| Field | Purpose |
|---|---|
| `TimeGenerated` | Time the event was recorded |
| `ResourceGroup` | Azure resource group associated with the event |
| `ResourceProvider` | Azure service that generated the telemetry |
| `Resource` | Resource associated with the event |
| `ResourceType` | Type of Azure resource |
| `OperationName` | Operation represented by the event |
| `ResultType` | Outcome associated with the operation |

These fields provided enough information for the initial Key Vault investigation without displaying every column available in `AzureDiagnostics`.

# 11. Key Vault Activity Monitoring Query

After confirming that Key Vault records existed, I created a more focused query.

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

## Investigation Question

> What Azure Key Vault activity is available in the workspace?

# 12. Query Breakdown

### Select the Table

```kusto
AzureDiagnostics
```

The investigation begins with the Azure diagnostic telemetry table.

### Filter for Key Vault

```kusto
| where ResourceProvider =~ "MICROSOFT.KEYVAULT"
```

This restricts the results to records generated by the Azure Key Vault resource provider.

### Select Relevant Fields

```kusto
| project
    TimeGenerated,
    ResourceGroup,
    ResourceProvider,
    Resource,
    ResourceType,
    OperationName,
    ResultType
```

Rather than displaying the full schema, `project` limits the output to fields useful for the investigation.

### Sort by Time

```kusto
| order by TimeGenerated desc
```

This places the most recent observed activity first.

# 13. KQL String Comparison Issue

During development of the Key Vault query, one version of my filter returned:

```text
0 results
```

even though I had already confirmed that Key Vault records existed in `AzureDiagnostics`.

This meant the problem was likely with the query rather than the ingestion pipeline.

I reviewed the value stored in:

```text
ResourceProvider
```

and adjusted the comparison to use:

```kusto
=~
```

instead of relying on a case-sensitive comparison.

The working condition became:

```kusto
| where ResourceProvider =~ "MICROSOFT.KEYVAULT"
```

The expected records were then returned.

# 14. Why `=~` Was Useful

In KQL:

```kusto
==
```

performs a case-sensitive equality comparison.

By comparison:

```kusto
=~
```

performs a case-insensitive equality comparison.

For example:

```kusto
ResourceProvider == "Microsoft.KeyVault"
```

may not match a stored value represented with different capitalization.

Using:

```kusto
ResourceProvider =~ "MICROSOFT.KEYVAULT"
```

allowed the query to match the resource-provider value without depending on capitalization.

This was a small query change, but it was an important troubleshooting lesson.

# 15. Key Vault Activity Results

After correcting the query and using the seven-day investigation window, the activity-monitoring query returned:

```text
4 events
```

The observed events showed:

```text
OperationName:
VaultGet

ResultType:
Success
```

These events became the baseline for the remaining investigation.

I did not interpret the existence of four Key Vault events as evidence of malicious activity.

At this stage, the query only established that Key Vault operations had occurred.

# 16. Failed Key Vault Operations Query

The next investigation question was:

> Did any of the available Key Vault events represent unsuccessful operations?

I used:

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

# 17. Understanding the Failure Filter

The important condition is:

```kusto
| where ResultType !in~ ("Success", "Succeeded")
```

This removes events whose result type matches either:

```text
Success
Succeeded
```

using case-insensitive comparison.

The remaining records, if any, can then be reviewed as potentially unsuccessful operations.

This does not automatically mean every returned event would be malicious.

It simply creates a focused dataset for further investigation.

# 18. Failed Operations Result

The query returned:

```text
0 results
```

I treated this as a valid investigation result.

The correct conclusion was:

> No Key Vault operations matching the unsuccessful-operation condition were identified within the available telemetry and selected investigation period.

I did not interpret this as:

```text
The Key Vault has never experienced a failed operation.
```

Nor did I interpret it as:

```text
The Key Vault is completely secure.
```

The result applied only to the telemetry available to the query.

# 19. Why Zero Results Matter in Threat Hunting

A threat-hunting query does not need to discover malicious activity to be useful.

A query can:

```text
Confirm a hypothesis

Reject a hypothesis

Establish a baseline

Identify missing telemetry

Reveal a query problem

Show that a condition was not observed
```

In this case, the failed-operation query helped establish that the four observed Key Vault events did not match the unsuccessful-operation condition.

That information later contributed to the outcome of the structured Sentinel Hunt.


# 20. Key Vault Activity Frequency Analysis

I next wanted to summarize the observed activity rather than inspect each event individually.

I used:

```kusto
AzureDiagnostics
| where ResourceProvider =~ "MICROSOFT.KEYVAULT"
| summarize EventCount = count() by Resource, OperationName, ResultType
| order by EventCount desc
```

## Investigation Question

> Which Key Vault operations and outcomes appear most frequently in the available telemetry?


# 21. Understanding the Frequency Query

The query begins with the same Key Vault filter:

```kusto
| where ResourceProvider =~ "MICROSOFT.KEYVAULT"
```

It then groups the records using:

```kusto
| summarize EventCount = count() by Resource, OperationName, ResultType
```

This produces a summarized result based on:

```text
Resource
Operation
Result
Event Count
```

Finally:

```kusto
| order by EventCount desc
```

places the most frequently observed combination first.


# 22. Purpose of Frequency Analysis

In a larger environment, activity-frequency analysis can help identify:

- Frequently accessed resources
- Unusual spikes in operations
- Changes in expected behavior
- Repeated failed operations
- Unexpected operation types
- Potential investigation baselines

The dataset in this project was small, so I did not attempt to build a statistical anomaly-detection model from four events.

Instead, the query demonstrated how the available activity could be summarized into a basic behavioral baseline.


# 23. Final Custom Query Set

The investigation ultimately produced three reusable Key Vault queries.

| Query | Purpose |
|---|---|
| Key Vault Activity Monitoring | Review Key Vault operations |
| Failed Key Vault Operations | Search for unsuccessful operations |
| Key Vault Activity Frequency Analysis | Summarize activity patterns |

The progression was:

```text
What happened?
      |
      v
Activity Monitoring

Did anything fail?
      |
      v
Failed Operations

What pattern is visible?
      |
      v
Frequency Analysis
```

# 24. Reusing the Queries in Microsoft Sentinel Hunting

After validating the queries through Logs, I reused them within a structured Microsoft Sentinel Hunt.

The Hunt was named:

```text
Azure Key Vault Suspicious Activity Investigation
```

I added the three queries as Hunt queries:

```text
Key Vault Activity Monitoring

Failed Key Vault Operations

Key Vault Activity Frequency Analysis
```

I then executed all three from within the Hunt.

The results matched the earlier investigation:

```text
Observed Key Vault events: 4

Successful events: 4

Failed operations matching query: 0
```

This allowed the KQL development work to become part of a repeatable threat-hunting workflow.

# 25. Hunt Decision

The Hunt began without assuming that the activity was malicious.

The hypothesis state was initially:

```text
Unknown
```

After running the queries and reviewing the available telemetry, I did not identify evidence supporting suspicious Key Vault activity.

The investigation therefore did not warrant escalation based on the available results.

This demonstrated an important distinction:

```text
Threat Hunting
      !=
Finding a Threat Every Time
```

A hunt is an investigation process, not a requirement to produce a positive detection.

# 26. KQL Query Files

The reusable queries are stored separately in the repository.

```text
queries/
|
+-- key-vault-activity.kql
|
+-- failed-key-vault-operations.kql
|
+-- key-vault-activity-frequency.kql
```

This separates executable query content from the explanatory documentation.

# 27. `key-vault-activity.kql`

The file should contain:

```kusto
// Key Vault Activity Monitoring
// Purpose: Display Azure Key Vault diagnostic activity.

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


# 28. `failed-key-vault-operations.kql`

The file should contain:

```kusto
// Failed Key Vault Operations
// Purpose: Identify Key Vault operations that do not have a successful result.

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


# 29. `key-vault-activity-frequency.kql`

The file should contain:

```kusto
// Key Vault Activity Frequency Analysis
// Purpose: Summarize observed Key Vault activity by resource, operation, and result.

AzureDiagnostics
| where ResourceProvider =~ "MICROSOFT.KEYVAULT"
| summarize EventCount = count() by Resource, OperationName, ResultType
| order by EventCount desc
```

# 30. Investigation Results Summary

| Investigation | Result |
|---|---|
| Workspace discovery | Telemetry identified |
| Initial 24-hour search | No useful results |
| Seven-day search | Telemetry found |
| `AzureDiagnostics` inspection | Key Vault telemetry identified |
| Key Vault activity query | 4 events |
| Observed result | Successful |
| Failed-operation query | 0 matching events |
| Frequency analysis | Successfully executed |
| Queries added to Hunt | 3 |
| Hunt escalation required | No |

# 31. Query Validation Method

I validated the KQL queries using the following process:

```text
Write Query
    |
    v
Execute Against Workspace
    |
    v
Review Result
    |
    +----------------------+
    |                      |
    v                      v
Expected Result       Unexpected Result
    |                      |
    v                      v
Document             Troubleshoot Query
                           |
                           v
                     Modify Filter
                           |
                           v
                       Re-run
```

This process was particularly useful when the Key Vault resource-provider filter initially returned zero results.

# 32. Query Design Decisions

Several decisions were made intentionally.

## Use the Available Schema

I developed the queries based on fields actually observed in the environment rather than assuming every Key Vault deployment would expose identical fields.

## Keep the Initial Queries Explainable

The telemetry volume was small.

I therefore preferred queries I could validate and explain rather than introducing complex joins, machine-learning functions, or anomaly logic that the available dataset could not meaningfully support.

## Separate Detection from Interpretation

A failed operation is not automatically malicious.

Likewise, a successful Key Vault operation is not automatically benign.

The queries identify activity that can support an investigation; analyst context is still required.

## Preserve Investigation Scope

Statements about the results are limited to:

```text
The available telemetry
+
The selected investigation period
+
The conditions defined by the query
```

This prevents conclusions that exceed the evidence.


# 33. Potential Production Enhancements

The queries used in this project provide a foundation that could be extended significantly in a larger environment.

Potential improvements include:

### Identity Correlation

Correlate Key Vault operations with identity information to determine which account or service principal performed the activity.

### Source IP Analysis

Extract and analyze client IP addresses where available.

### Time-Based Baselines

Compare current activity with historical patterns.

### Failure Thresholds

Identify repeated unsuccessful operations within a defined period.

### Rare Operation Detection

Identify operations rarely observed for a particular Key Vault.

### Azure Activity Correlation

Correlate Key Vault diagnostic activity with Azure control-plane operations.

### Sentinel Watchlists

Compare observed entities against approved or monitored entity lists.

### Threat Intelligence

Correlate relevant IP addresses or domains with threat-intelligence indicators where appropriate.

### Analytics Rules

Convert mature hunting logic into scheduled detection rules once the query has been sufficiently tested.

# 34. Example Detection Evolution

A useful security engineering progression would be:

```text
Raw Telemetry
     |
     v
Exploratory KQL
     |
     v
Threat Hunting Query
     |
     v
Validated Detection Logic
     |
     v
Analytics Rule
     |
     v
Alert
     |
     v
Incident
     |
     v
SOAR Response
```

The queries developed in this project currently sit primarily in the exploratory analysis and threat-hunting stages.

I did not automatically convert them into production analytics rules without sufficient telemetry and validation.


# 35. Evidence

The KQL portion of the project is supported by screenshots showing:

```text
Workspace table discovery

AzureDiagnostics records

Key Vault diagnostic telemetry

Four successful Key Vault events

Failed-operation query with zero matches

Frequency-analysis results

Queries added to the Sentinel Hunt
```

Recommended repository structure:

```text
images/
└── hunting/
    ├── log-analytics-data-tables.png
    ├── key-vault-diagnostic-events.png
    ├── key-vault-activity-results.png
    ├── failed-key-vault-query.png
    ├── key-vault-frequency-analysis.png
    └── key-vault-hunting-queries.png
```

# 36. Key Lessons

The KQL phase produced several practical lessons.

### Validate the Time Range First

The initial 24-hour search returned nothing, while the seven-day search exposed the available telemetry.

### Inspect Raw Data Before Writing Complex Queries

Looking directly at `AzureDiagnostics` helped me understand the fields and values actually available.

### Zero Results Do Not Automatically Mean Query Failure

The failed-operation query correctly returned zero matching records based on the available telemetry.

### Query Syntax Matters

The `=~` operator resolved the resource-provider matching problem encountered during the investigation.

### Hunting Should Follow Evidence

I did not alter the queries or interpretation simply to produce a suspicious result.


# 37. Final KQL Outcome

The KQL implementation established a direct investigation path from raw Azure telemetry to structured threat hunting:

```text
Log Analytics
      |
      v
Workspace Discovery
      |
      v
AzureDiagnostics
      |
      v
Key Vault Identification
      |
      v
Custom KQL
      |
      +----------------------+
      |          |           |
      v          v           v
Activity      Failed      Frequency
Monitoring   Operations   Analysis
      |          |           |
      +----------+-----------+
                 |
                 v
         Microsoft Sentinel Hunt
                 |
                 v
          Evidence Review
                 |
                 v
        Investigation Conclusion
```

This provided the KQL foundation for the threat-hunting portion of the Microsoft Sentinel project.

## Related Documentation

- [Architecture](Architecture.md)
- [Deployment Guide](DeploymentGuide.md)
- [Data Connectors](DataConnectors.md)
- [Analytics Rules](AnalyticsRules.md)
- [Incident Management](IncidentManagement.md)
- [Threat Hunting](ThreatHunting.md)
- [MITRE ATT&CK Mapping](MITRE-ATTACK-Mapping.md)
- [Workbooks](Workbooks.md)
- [Troubleshooting](Troubleshooting.md)
- [Validation Report](ValidationReport.md)
