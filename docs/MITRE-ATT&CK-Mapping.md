# MITRE ATT&CK Mapping and Detection Coverage

## 1. Overview

This document describes how I used the MITRE ATT&CK framework within Microsoft Sentinel to review and understand the behavioral coverage associated with the analytics capabilities implemented in this project.

The objective was not to claim broad MITRE ATT&CK coverage from a small lab environment.

Instead, I used the ATT&CK functionality available in Microsoft Sentinel to understand how analytics rules are mapped to attacker behaviors and how those mappings can help a SOC evaluate its detection posture.

The analytics rule I enabled during the project was:

```text
Microsoft Defender Threat Intelligence Analytics
```

The rule displayed mappings to the following ATT&CK tactics:

```text
Persistence

Lateral Movement
```

These mappings represent the attacker behaviors associated with the detection content.

They do not mean that Persistence or Lateral Movement activity was actually observed in the environment.


# 2. What Is MITRE ATT&CK?

MITRE ATT&CK is a knowledge base that organizes adversary behavior into tactics and techniques based on observed attacker activity.

At a high level:

```text
Tactic
   |
   v
Attacker Objective
   |
   v
Technique
   |
   v
Method Used to Achieve Objective
```

For example, an attacker may have an objective related to maintaining access to an environment.

That objective falls under a tactic such as:

```text
Persistence
```

The attacker may then use one or more techniques to achieve that objective.

ATT&CK gives security teams a common language for describing these behaviors.


# 3. Why ATT&CK Matters in a SOC

A SOC can use ATT&CK to understand more than individual alerts.

Instead of only asking:

```text
Which analytics rules are enabled?
```

the team can also ask:

```text
Which attacker behaviors can we detect?

Which tactics have detection coverage?

Which techniques have little or no coverage?

Where are our detection gaps?

Which telemetry sources support each detection?
```

This changes the perspective from:

```text
Number of Rules
```

to:

```text
Behavioral Detection Coverage
```

That distinction is important because a large number of analytics rules does not automatically mean that an environment has balanced security coverage.


# 4. ATT&CK in Microsoft Sentinel

Microsoft Sentinel integrates MITRE ATT&CK information into several security operations capabilities.

The relationship can be represented as:

```text
Security Telemetry
       |
       v
Analytics Rule
       |
       v
Detection Logic
       |
       v
MITRE ATT&CK Mapping
       |
       v
Detection Coverage
```

Sentinel allows analysts to review ATT&CK information associated with detection content and understand which attacker behaviors the available analytics capabilities are designed to detect.


# 5. ATT&CK Investigation in This Project

After configuring the analytics rule, I opened Microsoft Sentinel's MITRE ATT&CK coverage interface.

The interface provided a matrix-style view of attacker tactics and techniques.

I used this view to inspect the available detection coverage and understand how Sentinel connects analytics content to ATT&CK.

The investigation followed:

```text
Sentinel Analytics
       |
       v
Enabled Analytics Rule
       |
       v
Review ATT&CK Mapping
       |
       v
Open ATT&CK Coverage
       |
       v
Inspect Tactics and Techniques
       |
       v
Understand Detection Coverage
```


# 6. Enabled Analytics Rule

The rule used for the ATT&CK investigation was:

```text
Microsoft Defender Threat Intelligence Analytics
```

The observed configuration included:

| Property | Value |
|---|---|
| Rule | Microsoft Defender Threat Intelligence Analytics |
| Severity | Medium |
| Status | Enabled |
| ATT&CK Tactic | Persistence |
| ATT&CK Tactic | Lateral Movement |

The rule therefore contributed ATT&CK coverage associated with these two tactics.


# 7. ATT&CK Coverage Architecture

The relationship between the analytics rule and ATT&CK can be represented as:

```text
Microsoft Defender Threat
Intelligence Analytics
          |
          v
    Detection Logic
          |
      +---+---+
      |       |
      v       v
Persistence  Lateral Movement
```

This gives analysts a behavioral interpretation of what the detection content is intended to address.


# 8. Persistence

One of the tactics mapped to the analytics rule was:

```text
Persistence
```

Persistence refers to attacker behavior intended to maintain access to an environment after initial access has been achieved.

From a security-monitoring perspective, persistence-related detection coverage can help identify behaviors associated with attempts to maintain continued access.

The important distinction in this project is:

```text
Rule mapped to Persistence
            |
            X
            |
Does not mean Persistence
was observed
```

The mapping represents the behavior associated with the detection logic.


# 9. Persistence Technique Investigation

While reviewing the Sentinel ATT&CK interface, I could inspect technique-level information.

One of the techniques visible during the investigation was:

```text
Account Manipulation
```

The interface exposed information such as:

```text
Technique ID

Incidents

Alerts

Description

Tactic
```

This allowed me to move from a high-level tactic into more detailed technique information.

I did not interpret the presence of the technique in the ATT&CK interface as evidence that Account Manipulation had occurred in the environment.

# 10. Account Manipulation

Account Manipulation describes attacker behavior involving changes to accounts that may help maintain or expand access.

Within a security operations environment, detections related to account manipulation can be important because identity changes may be associated with attempts to:

```text
Maintain access

Modify permissions

Change account configuration

Alter authentication-related settings
```

The technique was reviewed as part of understanding ATT&CK coverage within Sentinel.

No claim is made that the technique was actually executed against the project environment.


# 11. Lateral Movement

The second tactic mapped to the enabled analytics rule was:

```text
Lateral Movement
```

Lateral Movement refers to attacker behavior intended to move through an environment after obtaining access.

An attacker may attempt to reach:

```text
Additional systems

Privileged accounts

Cloud resources

Applications

Sensitive services
```

Detection coverage associated with Lateral Movement can therefore help identify activity that suggests an attacker is expanding access beyond the original point of compromise.

Again:

```text
Lateral Movement Mapping
          !=
Observed Lateral Movement
```

The mapping describes the behavior the detection content is designed to address.


# 12. Detection Coverage vs Attack Evidence

This distinction was one of the most important parts of the ATT&CK investigation.

Suppose an analytics rule displays:

```text
Persistence
Lateral Movement
```

That means the rule contributes detection content associated with those tactics.

It does not mean:

```text
An attacker established persistence.

An attacker moved laterally.

The environment was compromised.
```

ATT&CK coverage should therefore be interpreted as:

```text
Detection Capability
```

rather than:

```text
Proof of Attack
```

# 13. ATT&CK Matrix Interpretation

The ATT&CK matrix provides a way to organize security coverage across attacker objectives.

A simplified conceptual representation is:

```text
Reconnaissance
      |
Resource Development
      |
Initial Access
      |
Execution
      |
Persistence
      |
Privilege Escalation
      |
Defense Evasion
      |
Credential Access
      |
Discovery
      |
Lateral Movement
      |
Collection
      |
Command and Control
      |
Exfiltration
      |
Impact
```

A mature SOC attempts to understand how its detection capabilities map across these behaviors.

The objective is not necessarily to have an equal number of rules everywhere.

Instead, coverage should reflect:

```text
Organizational Risk

Attack Surface

Available Telemetry

Threat Model

Technology Stack
```


# 14. ATT&CK and Detection Engineering

ATT&CK becomes particularly useful during detection engineering.

A detection engineer can start with:

```text
ATT&CK Technique
       |
       v
Required Telemetry
       |
       v
Detection Hypothesis
       |
       v
KQL Logic
       |
       v
Analytics Rule
       |
       v
Validation
```

Alternatively, an existing analytics rule can be evaluated in the opposite direction:

```text
Analytics Rule
       |
       v
What Does It Detect?
       |
       v
Which ATT&CK Behavior?
       |
       v
Which Tactic / Technique?
       |
       v
Where Does It Contribute Coverage?
```

The second approach was used during this project.


# 15. ATT&CK and Telemetry

ATT&CK mapping alone is not enough.

Detection coverage also depends on whether the environment collects the telemetry required by the detection.

For example:

```text
ATT&CK Technique
       |
       v
Analytics Rule
       |
       v
Required Data Source
       |
       v
Telemetry Available?
       |
       +-------------+
       |             |
      Yes            No
       |             |
       v             v
Detection       Detection Gap
Possible
```

This is one reason I did not enable the available RDP and SSH analytics templates.

The project did not contain the corresponding workloads and telemetry required to meaningfully support those detections.


# 16. ATT&CK and the RDP Template

The available analytics content included:

```text
Anomalous RDP Login Detection
```

RDP-related detection could contribute useful behavioral coverage in an environment containing Windows systems with relevant authentication telemetry.

However, the project did not include virtual machines being monitored for RDP activity.

Enabling the rule would therefore have created a detection configuration without meaningful supporting telemetry.

I chose not to enable it.


# 17. ATT&CK and the SSH Template

The environment also provided:

```text
Anomalous SSH Login Detection
```

SSH-focused analytics can be useful for detecting suspicious authentication activity involving Linux systems and other SSH-enabled infrastructure.

The project did not include SSH workloads generating the required telemetry.

I therefore did not enable the template solely to increase apparent ATT&CK coverage.


# 18. Meaningful Coverage vs Artificial Coverage

This resulted in an important design principle:

```text
More ATT&CK Mappings
        |
        X
        |
Do Not Automatically Mean
Better Security
```

Meaningful detection coverage requires:

```text
Relevant Attack Surface
        +
Required Telemetry
        +
Appropriate Detection Logic
        +
Validation
```

Simply enabling templates without the supporting data can create the appearance of coverage without providing meaningful detection capability.


# 19. ATT&CK and Threat Hunting

ATT&CK can also support threat hunting.

A hunter can begin with an attacker behavior and develop a hypothesis around it.

For example:

```text
ATT&CK Behavior
      |
      v
Security Hypothesis
      |
      v
Required Telemetry
      |
      v
KQL Query
      |
      v
Hunt
      |
      v
Evidence Analysis
```

The Key Vault Hunt in this project was primarily resource-focused rather than ATT&CK-technique-driven.

However, the same hunting methodology could be extended to specific ATT&CK techniques in a larger environment.


# 20. ATT&CK and Incidents

When analytics rules generate alerts and incidents, ATT&CK mappings can provide behavioral context to analysts.

The relationship becomes:

```text
Analytics Rule
      |
      v
ATT&CK Mapping
      |
      v
Alert
      |
      v
Incident
      |
      v
Analyst Investigation
```

An analyst reviewing an incident can use the ATT&CK context to understand:

```text
What attacker objective may be relevant?

Which technique is associated with the detection?

What other behaviors should I search for?

Which telemetry should I pivot into next?
```

This makes ATT&CK useful during investigation rather than only during detection design.

# 21. ATT&CK and SOAR

ATT&CK context can also influence automated response.

For example, a mature SOAR implementation could apply different workflows based on:

```text
Tactic

Technique

Severity

Affected Entity

Detection Source
```

A conceptual workflow could be:

```text
Incident
   |
   v
Read ATT&CK Context
   |
   v
Determine Tactic
   |
   +---------------------+
   |                     |
   v                     v
Credential Access    Lateral Movement
   |                     |
   v                     v
Identity Workflow    Host / Network Workflow
```

The project did not implement ATT&CK-specific automation branching, but the architecture provides a foundation for that type of future enhancement.

# 22. ATT&CK Coverage Observed in the Project

The validated ATT&CK coverage from the enabled analytics rule was:

| Analytics Rule | Tactic |
|---|---|
| Microsoft Defender Threat Intelligence Analytics | Persistence |
| Microsoft Defender Threat Intelligence Analytics | Lateral Movement |

This table intentionally lists only the mappings I verified from the enabled rule.

I do not claim comprehensive ATT&CK coverage across the entire matrix.


# 23. What the ATT&CK Review Demonstrated

The ATT&CK portion of the project demonstrated that I could:

```text
Navigate Sentinel ATT&CK coverage

Inspect tactic-level information

Inspect technique-level information

Connect analytics rules to ATT&CK

Interpret detection coverage correctly

Distinguish mappings from confirmed attacks

Consider telemetry requirements

Identify potential detection gaps
```

This moved the project beyond simply enabling an analytics rule.


# 24. ATT&CK Coverage Validation

The following components were validated:

| Component | Result |
|---|---|
| Sentinel ATT&CK interface | Reviewed |
| ATT&CK matrix | Accessible |
| Technique details | Reviewed |
| Analytics-rule mappings | Reviewed |
| Persistence mapping | Confirmed |
| Lateral Movement mapping | Confirmed |
| Account Manipulation information | Reviewed |
| ATT&CK-based incident claim | Not made |
| Full ATT&CK coverage claim | Not made |


# 25. Evidence

The ATT&CK portion of the project can be supported by screenshots showing:

```text
MITRE ATT&CK matrix

Persistence tactic

Lateral Movement tactic

Technique details

Account Manipulation information

Analytics rule ATT&CK mappings
```

Recommended repository structure:

```text
images/
└── mitre/
    ├── sentinel-mitre-attack-matrix.png
    ├── persistence-coverage.png
    ├── lateral-movement-coverage.png
    ├── account-manipulation-technique.png
    └── analytics-rule-mitre-mapping.png
```

Only screenshots actually captured during implementation should be added.

# 26. ATT&CK Coverage Limitations

The ATT&CK implementation had several limitations.

## Limited Analytics Rule Set

Only a small number of analytics templates were available in the environment, and one relevant rule was enabled.

## Limited Telemetry Sources

The project did not contain the diversity of telemetry expected in a production SOC.

## No Endpoint Fleet

Endpoint-focused ATT&CK coverage was limited because a large Defender for Endpoint environment was not part of the implementation.

## No RDP/SSH Workloads

Available RDP and SSH detection templates were not enabled because the supporting workloads were not present.

## No Full ATT&CK Gap Assessment

The project reviewed the available Sentinel coverage but did not perform a formal organization-wide ATT&CK gap analysis.

## No ATT&CK-Based Detection Testing

The project did not simulate every mapped attacker technique to validate detections.

# 27. Future ATT&CK Improvements

A future version of the project could expand ATT&CK coverage significantly.

Potential improvements include:

### Build an ATT&CK Coverage Inventory

Document every active analytics rule and its associated tactics and techniques.

### Add Identity Telemetry

Identity logs could support detections related to:

```text
Credential Access

Persistence

Privilege Escalation

Defense Evasion
```

depending on the available data and detection logic.

### Add Endpoint Telemetry

Microsoft Defender for Endpoint could significantly expand behavioral visibility across endpoints.

### Add Network Telemetry

Network security telemetry could support additional investigation and detection scenarios.

### Develop Custom Analytics Rules

Custom KQL detections could be mapped to appropriate ATT&CK techniques after validation.

### ATT&CK-Based Hunting

Create Hunts starting from specific techniques rather than only from individual resources.

### Detection Gap Analysis

Identify techniques relevant to the environment that currently lack meaningful detection coverage.

### Detection Validation

Safely test detection logic against controlled activity where appropriate.

# 28. Example ATT&CK Detection Gap Process

A future detection-gap assessment could follow:

```text
Identify Critical Assets
        |
        v
Define Relevant Threat Scenarios
        |
        v
Map Scenarios to ATT&CK
        |
        v
Review Existing Analytics Rules
        |
        v
Map Current Coverage
        |
        v
Identify Gaps
        |
        v
Determine Required Telemetry
        |
        v
Develop Detection Logic
        |
        v
Test
        |
        v
Deploy
        |
        v
Continuously Tune
```

This would turn the ATT&CK matrix into a practical detection-engineering planning tool.


# 29. Example Detection Coverage Table

A mature version of the environment could maintain a table similar to:

| ATT&CK Tactic | Technique | Data Source | Detection | Validation | Status |
|---|---|---|---|---|---|
| Persistence | Example technique | Identity telemetry | Analytics rule | Test activity | Covered |
| Lateral Movement | Example technique | Endpoint telemetry | Analytics rule | Test activity | Covered |
| Credential Access | Example technique | Endpoint telemetry | Hunting query | Not tested | Partial |
| Discovery | Example technique | Endpoint telemetry | None | N/A | Gap |

This type of table can provide more useful information than simply counting the number of enabled analytics rules.


# 30. ATT&CK Maturity Progression

The ATT&CK component can be viewed as a maturity progression:

```text
Understand ATT&CK
       |
       v
View Sentinel ATT&CK Matrix
       |
       v
Review Analytics Mappings
       |
       v
Understand Current Coverage
       |
       v
Identify Missing Coverage
       |
       v
Collect Required Telemetry
       |
       v
Develop New Detections
       |
       v
Validate Detections
       |
       v
Continuously Improve Coverage
```

This project covered the early and intermediate stages of that progression.


# 31. Key Lessons

Several lessons came from the ATT&CK phase.

## ATT&CK Provides Context

It helps explain the attacker behavior that detection content is intended to identify.

## Mapping Does Not Mean Detection Occurred

A rule mapped to Persistence does not mean persistence activity happened.

## Coverage Depends on Telemetry

A detection cannot provide meaningful coverage without the required data.

## More Rules Are Not Automatically Better

Relevant and validated rules are more valuable than unrelated templates enabled for appearance.

## ATT&CK Can Reveal Detection Gaps

The matrix can help identify areas where additional telemetry and detection engineering may be required.

## ATT&CK Connects Multiple SOC Functions

It can support:

```text
Detection Engineering

Threat Hunting

Incident Investigation

Threat Modeling

Detection Gap Analysis

SOAR Design
```

# 32. Final ATT&CK Outcome

The final ATT&CK implementation can be summarized as:

```text
Platform:
Microsoft Sentinel

ATT&CK Interface:
Reviewed

Enabled Analytics Rule:
Microsoft Defender Threat Intelligence Analytics

Verified Tactic Mappings:
Persistence
Lateral Movement

Technique Information Reviewed:
Yes

Account Manipulation:
Reviewed within ATT&CK interface

Confirmed Attack:
No claim made

Comprehensive ATT&CK Coverage:
No claim made
```

The final relationship was:

```text
Microsoft Sentinel
       |
       v
Analytics Rule
       |
       v
Microsoft Defender Threat
Intelligence Analytics
       |
       +-----------------------+
       |                       |
       v                       v
Persistence              Lateral Movement
       |                       |
       +-----------+-----------+
                   |
                   v
          ATT&CK Coverage View
                   |
                   v
          Detection Context
```

The value of the ATT&CK implementation was not simply displaying a matrix.

It was understanding how Sentinel connects detection content to attacker behavior and how that information can be used to evaluate detection coverage without confusing coverage with evidence of compromise.

## Related Documentation

- [Architecture](Architecture.md)
- [Deployment Guide](DeploymentGuide.md)
- [Data Connectors](DataConnectors.md)
- [KQL Queries](KQLQueries.md)
- [Analytics Rules](AnalyticsRules.md)
- [Incident Management](IncidentManagement.md)
- [Threat Hunting](ThreatHunting.md)
- [Workbooks](Workbooks.md)
- [SOAR Automation](SOAR-Automation.md)
- [Validation Report](ValidationReport.md)
- [Troubleshooting](Troubleshooting.md)
