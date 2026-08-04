# Microsoft Sentinel Threat Hunting Investigation

## 1. Overview

This document describes the threat-hunting investigation I performed in Microsoft Sentinel using Azure Key Vault diagnostic telemetry.

The objective was to move beyond individual KQL queries and conduct a structured, hypothesis-driven investigation using Microsoft Sentinel Hunts.

The investigation focused on:

```text
Azure Key Vault:
kv-contoso-prod-001
```

with telemetry stored in:

```text
Log Analytics Workspace:
law-contoso-prod-001

Primary Table:
AzureDiagnostics
```

The Hunt was created as:

```text
Azure Key Vault Suspicious Activity Investigation
```

Three custom KQL queries were added to the Hunt to investigate:

1. General Key Vault activity
2. Unsuccessful Key Vault operations
3. Key Vault activity frequency

The investigation identified four successful Key Vault events and no unsuccessful operations matching the hunting condition.

Based on the telemetry available during the investigation period, I did not identify evidence that justified escalation to a security incident.


# 2. What Is Threat Hunting?

Threat hunting is a proactive security investigation process used to search telemetry for suspicious behavior that may not already have generated an automated detection.

Unlike traditional alert-driven investigation:

```text
Alert-Driven Investigation

Detection
   |
   v
Alert
   |
   v
Incident
   |
   v
Investigation
```

threat hunting can begin with a question or hypothesis:

```text
Threat Hunting

Hypothesis
   |
   v
Telemetry
   |
   v
KQL Investigation
   |
   v
Evidence Analysis
   |
   v
Conclusion
```

This makes hunting useful for exploring activity that may fall outside existing detection rules.

# 3. Hunting Objective

The objective of this Hunt was to determine whether the available Azure Key Vault telemetry contained activity that could warrant further security investigation.

The investigation focused on three questions:

```text
What Key Vault operations occurred?

Were any observed Key Vault operations unsuccessful?

What activity pattern was visible in the available telemetry?
```

These questions formed the basis for the three custom hunting queries.

# 4. Hunt Scope

The investigation scope was intentionally limited to the telemetry actually available in the environment.

| Scope Item | Value |
|---|---|
| Platform | Microsoft Sentinel |
| Resource | Azure Key Vault |
| Key Vault | `kv-contoso-prod-001` |
| Resource Group | `rg-contoso-prod-001` |
| Workspace | `law-contoso-prod-001` |
| Primary Table | `AzureDiagnostics` |
| Investigation Window | Last 7 days |
| Hunting Queries | 3 |
| Observed Key Vault Events | 4 |
| Matching Failed Operations | 0 |

The Hunt did not attempt to make conclusions about activity outside the available telemetry or selected time range.

# 5. Pre-Hunt Telemetry Validation

Before creating the structured Hunt, I first validated the underlying data.

I used:

```kusto
search *
| summarize EventCount = count() by $table
| order by EventCount desc
```

The initial 24-hour search returned no useful telemetry.

I expanded the investigation window to:

```text
Last 7 days
```

The workspace then returned:

```text
AzureMetrics       24
Usage               7
AzureDiagnostics    4
```

This confirmed that data was available for further investigation.


# 6. Identifying the Key Vault Events

I inspected `AzureDiagnostics` directly:

```kusto
AzureDiagnostics
| order by TimeGenerated desc
| take 50
```

The available telemetry included records associated with:

```text
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

This established Azure Key Vault as the telemetry source for the Hunt.

# 7. Hunt Creation

I opened Microsoft Sentinel Hunting and created a new Hunt.

The Hunt creation interface required:

```text
Hunt Name
Description
Owner
Status
Hypothesis
```

I created:

```text
Hunt Name:
Azure Key Vault Suspicious Activity Investigation
```

The available hypothesis states were:

```text
Unknown

Invalidated

Validated
```

At the beginning of the investigation, the evidence had not yet been reviewed.

The hypothesis therefore began as:

```text
Unknown
```

This avoided assuming the investigation outcome before running the queries.


# 8. Hunt Hypothesis

The working hypothesis was:

> The available Azure Key Vault telemetry may contain unsuccessful or unusual activity that warrants further security investigation.

The purpose of the Hunt was not to prove this hypothesis.

The purpose was to test it against the available evidence.

The investigation model was:

```text
Hypothesis
    |
    v
Unknown
    |
    v
Run Hunting Queries
    |
    v
Analyze Evidence
    |
    +--------------------+
    |                    |
    v                    v
Evidence Supports     Evidence Does Not
Hypothesis            Support Hypothesis
    |                    |
    v                    v
Validated            Invalidated
```

# 9. Adding Queries to the Hunt

After creating the Hunt, the Queries section initially showed:

```text
No queries found
```

The interface provided options to:

```text
Create New Query

Add Queries to Hunt
```

I created three custom queries specifically for the investigation.

They were:

```text
Key Vault Activity Monitoring

Failed Key Vault Operations

Key Vault Activity Frequency Analysis
```

# 10. Hunting Query 1: Key Vault Activity Monitoring

The first query established the activity visible in the telemetry.

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

> What Azure Key Vault operations are present in the available telemetry?

The query returned:

```text
4 Key Vault events
```

The observed operations showed:

```text
OperationName:
VaultGet

ResultType:
Success
```

This established the initial activity baseline.

# 11. Analysis of Query 1

The first query confirmed that Key Vault activity existed.

However, the presence of activity alone did not indicate suspicious behavior.

The results showed successful operations.

Therefore, the next investigation step was to determine whether unsuccessful operations were also present.

The investigative progression became:

```text
Key Vault Activity Exists
          |
          v
Review Operation Results
          |
          v
Search Specifically for Failures
```

# 12. Hunting Query 2: Failed Key Vault Operations

The second query focused specifically on unsuccessful Key Vault activity.

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

## Investigation Question

> Are there unsuccessful Key Vault operations within the available telemetry that may require further investigation?

The query returned:

```text
0 matching results
```

# 13. Analysis of Query 2

The zero-result outcome was treated as evidence rather than as a failed query.

The correct conclusion was:

> No Key Vault events matching the unsuccessful-operation condition were identified within the available telemetry and selected investigation period.

This does not mean:

```text
The Key Vault has never experienced a failed operation.
```

It also does not mean:

```text
The Key Vault is guaranteed to be secure.
```

The result applies only to the data available to the query.

# 14. Hunting Query 3: Key Vault Activity Frequency Analysis

The third query summarized the observed activity.

```kusto
AzureDiagnostics
| where ResourceProvider =~ "MICROSOFT.KEYVAULT"
| summarize EventCount = count() by Resource, OperationName, ResultType
| order by EventCount desc
```

## Investigation Question

> What operation and result pattern is visible in the available Key Vault telemetry?

The query grouped activity by:

```text
Resource
OperationName
ResultType
EventCount
```

This provided a basic activity-frequency view of the small dataset.


# 15. Analysis of Query 3

The available telemetry was limited to four Key Vault events.

Because the dataset was small, I did not attempt to claim that the results represented a statistically meaningful long-term baseline.

Instead, the query demonstrated how activity could be summarized for further investigation.

In a larger environment, this type of analysis could help identify:

```text
Unexpected activity spikes

Repeated failures

Rare operations

Changes in normal access patterns

Unusual resource usage
```

For this project, it provided supporting context for the Hunt.

# 16. Executing the Hunt

After adding all three queries, I executed them from within the Hunt.

The queries produced the same results previously validated through Log Analytics.

The final Hunt query results were:

| Query | Result |
|---|---|
| Key Vault Activity Monitoring | 4 events |
| Failed Key Vault Operations | 0 matches |
| Key Vault Activity Frequency Analysis | Successfully summarized available activity |

This confirmed that the queries worked correctly within the structured Sentinel hunting workflow.

# 17. Evidence Correlation

The three queries were designed to complement one another.

```text
Query 1
What happened?
    |
    v
4 Key Vault events
    |
    v
Query 2
Did any fail?
    |
    v
0 matching failed operations
    |
    v
Query 3
What pattern is visible?
    |
    v
Successful activity summarized
```

Together, they provided more context than any single query alone.


# 18. Hunt Findings

The primary findings were:

```text
Finding 1:
Azure Key Vault telemetry was present.

Finding 2:
Four Key Vault events were observed.

Finding 3:
The observed operations were successful.

Finding 4:
No operations matched the unsuccessful-operation query.

Finding 5:
The available dataset was small.

Finding 6:
The available evidence did not justify escalation.
```

No evidence requiring incident creation was identified from the Hunt.

# 19. Hypothesis Decision

The Hunt began with:

```text
Hypothesis:
Unknown
```

After executing the three queries and reviewing the evidence, the available telemetry did not support the working hypothesis that unsuccessful or unusual Key Vault activity requiring escalation was present.

The hypothesis was therefore concluded as:

```text
Invalidated
```

within the scope of the available evidence.

This conclusion should be interpreted as:

> The investigation did not identify evidence supporting the Hunt hypothesis within the available telemetry and selected investigation period.

It should not be interpreted as a permanent statement that suspicious activity could never occur against the Key Vault.


# 20. Why the Hypothesis Was Not Marked Validated

Marking the hypothesis:

```text
Validated
```

would have implied that the available evidence supported the suspicious-activity hypothesis.

The investigation did not produce that evidence.

The observed events were successful, and the unsuccessful-operation query returned no matches.

Therefore, marking the hypothesis as validated would not have been supported by the results.

# 21. Why I Did Not Create an Incident

Threat hunting can lead to incident escalation when sufficient evidence is discovered.

The escalation path would be:

```text
Hunting Query
      |
      v
Suspicious Result
      |
      v
Evidence Review
      |
      v
Create Bookmark / Investigation Context
      |
      v
Escalate
      |
      v
Incident
```

In this Hunt, the available evidence did not support that escalation.

I therefore did not create an incident solely to demonstrate the feature.

This preserved the distinction between:

```text
Investigated Activity
```

and:

```text
Confirmed or Escalated Security Finding
```

# 22. Bookmarks

Microsoft Sentinel hunting can use bookmarks to preserve interesting query results for deeper investigation.

Bookmarks are useful when an analyst identifies a result that should be:

- Preserved
- Annotated
- Investigated further
- Correlated with other evidence
- Used during escalation

The Hunt did not identify a result that required preservation as a suspicious finding.

I therefore did not create a bookmark merely to populate the Bookmarks section.

# 23. Entities

The Hunt also included an Entities area.

Entities can help analysts pivot from a hunting result into related security context.

Examples can include:

```text
Accounts
Hosts
IP Addresses
Azure Resources
Domains
URLs
```

The available Key Vault dataset did not provide enough suspicious entity context to justify an entity-driven escalation.

In a larger investigation, entity analysis would become an important next step.

# 24. Hunting vs Analytics Rules

This investigation also demonstrated the difference between automated detection and proactive hunting.

## Analytics

```text
Telemetry
    |
    v
Detection Rule
    |
    v
Condition Match
    |
    v
Alert
```

## Hunting

```text
Hypothesis
    |
    v
Analyst Query
    |
    v
Telemetry
    |
    v
Evidence Analysis
    |
    v
Conclusion
```

Analytics continuously evaluates predefined conditions.

Threat hunting allows analysts to proactively investigate questions that may not already have dedicated detection logic.

# 25. Hunting vs Incident Investigation

Threat hunting and incident investigation also begin differently.

Incident investigation begins with something already detected:

```text
Alert
   |
   v
Incident
   |
   v
Investigation
```

Threat hunting can begin without an alert:

```text
Hypothesis
   |
   v
Query
   |
   v
Investigation
```

A successful Hunt may eventually lead to an incident if sufficient evidence is discovered.

In this case, escalation was not required.

# 26. Query Troubleshooting During the Investigation

The Key Vault investigation also required query troubleshooting.

An early resource-provider filter did not return the records I expected.

I adjusted the comparison to:

```kusto
=~
```

resulting in:

```kusto
| where ResourceProvider =~ "MICROSOFT.KEYVAULT"
```

The expected events were then returned.

This demonstrated that threat hunting involves more than running prewritten queries.

The analyst must also understand:

```text
Telemetry schema

Field values

Query operators

Time ranges

Result interpretation
```


# 27. Time Range Troubleshooting

The initial workspace investigation used:

```text
Last 24 hours
```

and returned no useful telemetry.

I expanded the period to:

```text
Last 7 days
```

and identified the available Key Vault records.

This became an important hunting lesson:

> A zero-result query should be validated against the investigation timeframe before concluding that the telemetry source is unavailable.


# 28. Hunt Investigation Workflow

The complete Hunt followed this process:

```text
Identify Available Telemetry
          |
          v
Select Key Vault as Investigation Target
          |
          v
Define Security Hypothesis
          |
          v
Create Sentinel Hunt
          |
          v
Set Hypothesis to Unknown
          |
          v
Add Three Custom Queries
          |
          v
Run Queries
          |
          v
Review Key Vault Activity
          |
          v
Search for Failed Operations
          |
          v
Analyze Activity Frequency
          |
          v
Correlate Results
          |
          v
Determine Whether Evidence Supports Hypothesis
          |
          v
No Supporting Evidence Identified
          |
          v
Invalidate Hypothesis
          |
          v
Close Investigation Without Escalation
```

# 29. Hunt Result Summary

| Investigation Area | Result |
|---|---|
| Hunt Created | Yes |
| Custom Queries | 3 |
| Key Vault Activity Found | Yes |
| Key Vault Events | 4 |
| Successful Operations | 4 |
| Failed Operations Matching Query | 0 |
| Suspicious Finding Requiring Escalation | No |
| Incident Created | No |
| Bookmark Required | No |
| Hypothesis Outcome | Invalidated |
| Hunt Completed | Yes |


# 30. Evidence-Based Decision Making

One of the most important principles applied during this Hunt was:

```text
Do not force the evidence to fit the hypothesis.
```

The objective of threat hunting is not to discover a threat every time.

The objective is to investigate a security question using available evidence.

Possible outcomes include:

```text
Hypothesis validated

Hypothesis invalidated

Insufficient evidence

Additional telemetry required

Escalation required
```

For this investigation, the available evidence did not support escalation.

# 31. Limitations of the Hunt

The Hunt had several limitations.

## Small Dataset

Only four Key Vault diagnostic events were available during the selected period.

This is significantly smaller than the telemetry volume expected in a production environment.

## Limited Entity Context

The available dataset did not support a rich investigation across users, source IP addresses, hosts, and related entities.

## Limited Historical Baseline

The available data was insufficient to establish a meaningful long-term behavioral baseline.

## No Failed Operations

The failed-operation query did not return any events for deeper investigation.

## No Active Security Incident

There was no related active Sentinel incident available for correlation.

These limitations affect how broadly the Hunt findings can be interpreted.


# 32. What Would Trigger Escalation?

In a production environment, the investigation could be escalated if the queries identified activity such as:

```text
Repeated unsuccessful Key Vault operations

Unexpected access to secrets or keys

Access from an unusual identity

Access from an unexpected IP address

Unusual operation frequency

Activity outside expected operational periods

Unexpected administrative changes

Activity correlated with threat-intelligence indicators

Related suspicious activity across other Azure resources
```

A suspicious result would then require additional evidence before being classified as malicious.


# 33. Production Investigation Expansion

If suspicious activity were discovered, I would expand the Hunt beyond the original three queries.

The next investigation stages could include:

```text
Suspicious Key Vault Event
          |
          v
Identify Identity
          |
          v
Identify Source IP
          |
          v
Review Related Azure Activity
          |
          v
Review Authentication Activity
          |
          v
Check Threat Intelligence
          |
          v
Search Related Resources
          |
          v
Build Timeline
          |
          v
Determine Scope
          |
          v
Escalate if Required
```

This would turn the initial Key Vault Hunt into a broader cloud-security investigation.


# 34. Potential Future Hunting Queries

The Hunt could be expanded with additional queries targeting:

### Repeated Failures

Detect multiple unsuccessful operations within a short period.

### High-Frequency Access

Identify unusually large numbers of Key Vault operations.

### Rare Operations

Find operations that rarely occur in the historical dataset.

### Identity Analysis

Group activity by user, application, or service principal where the relevant fields are available.

### Source IP Analysis

Identify unusual or previously unseen source IP addresses.

### Time-Based Analysis

Search for access outside normal operational periods.

### Administrative Changes

Investigate changes to Key Vault configuration or access controls.

### Cross-Service Correlation

Correlate Key Vault activity with Azure Activity, identity, endpoint, or other security telemetry.


# 35. Hunting to Detection Engineering

A mature hunting query can eventually become a detection.

The progression can be:

```text
Security Question
      |
      v
Threat Hunt
      |
      v
Custom KQL
      |
      v
Repeated Useful Finding
      |
      v
Query Refinement
      |
      v
False Positive Analysis
      |
      v
Analytics Rule
      |
      v
Automated Detection
```

I did not convert the three Key Vault hunting queries directly into production analytics rules because the available dataset was too small to validate detection quality sufficiently.

This keeps exploratory hunting logic separate from production-ready detection engineering.


# 36. Evidence Captured

The threat-hunting implementation can be supported with screenshots showing:

```text
Hunting interface

New Hunt creation

Hunt details

Hypothesis field

Queries section

Three custom hunting queries

Query execution results

Completed Hunt
```

Recommended repository structure:

```text
images/
└── hunting/
    ├── sentinel-hunting-overview.png
    ├── create-key-vault-hunt.png
    ├── key-vault-hunt-details.png
    ├── hunting-queries.png
    ├── key-vault-activity-results.png
    ├── failed-key-vault-results.png
    ├── key-vault-frequency-results.png
    └── completed-key-vault-hunt.png
```

Only screenshots actually captured during implementation should be included.

# 37. Skills Demonstrated

This Hunt demonstrated practical experience with:

```text
Microsoft Sentinel Hunting

Hypothesis-driven investigation

Kusto Query Language

AzureDiagnostics

Azure Key Vault telemetry

Telemetry validation

Query troubleshooting

Time-range analysis

Activity baselining

Failed-operation analysis

Evidence correlation

Investigation scoping

Security escalation decisions
```

More importantly, it demonstrated the ability to distinguish between:

```text
An interesting event

A suspicious event

A confirmed security finding
```

These are not automatically the same thing.

# 38. Key Lessons

Several important lessons came from the threat-hunting phase.

## Hunting Starts with a Question

A Hunt should have a clear security question rather than simply running random queries.

## The Hypothesis Can Be Wrong

Invalidating a hypothesis is a legitimate investigation result.

## Zero Results Can Be Useful

The absence of matching failed operations provided information about the available telemetry.

## Telemetry Determines What Can Be Investigated

The depth of a Hunt depends heavily on the quality and volume of available data.

## Time Range Matters

The initial 24-hour investigation would have missed the telemetry later found within the seven-day window.

## KQL Requires Troubleshooting

Query logic must be validated against the actual schema and field values.

## Hunting Does Not Require Manufacturing an Incident

An investigation should only be escalated when the evidence supports escalation.


# 39. Final Hunt Outcome

The final investigation can be summarized as:

```text
Hunt:
Azure Key Vault Suspicious Activity Investigation

Resource:
kv-contoso-prod-001

Workspace:
law-contoso-prod-001

Telemetry:
AzureDiagnostics

Queries:
3

Observed Key Vault Events:
4

Observed Results:
Success

Failed Operations Matching Query:
0

Escalation:
Not Required

Incident:
Not Created

Hypothesis:
Invalidated
```

The Hunt demonstrated an end-to-end proactive investigation process:

```text
Hypothesis
    |
    v
Telemetry Validation
    |
    v
KQL Development
    |
    v
Structured Sentinel Hunt
    |
    v
Query Execution
    |
    v
Evidence Analysis
    |
    v
Hypothesis Decision
    |
    v
No Escalation Required
```

The value of the exercise was not finding a threat where none was supported by the evidence.

The value was demonstrating a repeatable process for asking a security question, investigating it using real telemetry, interpreting the results correctly, and making an evidence-based escalation decision.


## Related Documentation

- [Architecture](Architecture.md)
- [Deployment Guide](DeploymentGuide.md)
- [Data Connectors](DataConnectors.md)
- [KQL Queries](KQLQueries.md)
- [Analytics Rules](AnalyticsRules.md)
- [Incident Management](IncidentManagement.md)
- [MITRE ATT&CK Mapping](MITRE-ATTACK-Mapping.md)
- [Workbooks](Workbooks.md)
- [SOAR Automation](SOAR-Automation.md)
- [Validation Report](ValidationReport.md)
- [Troubleshooting](Troubleshooting.md)
