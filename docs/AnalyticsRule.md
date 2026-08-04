# Microsoft Sentinel Analytics Rules and Detection Engineering

## 1. Overview

This document describes the analytics and detection configuration implemented within the Microsoft Sentinel SIEM and SOAR project.

After validating that security telemetry was available in the Log Analytics workspace, I moved to the detection layer of Microsoft Sentinel.

The objective was not simply to enable as many analytics rules as possible.

Instead, I wanted to understand:

- How Microsoft Sentinel analytics rules work
- Which detection templates were relevant to the environment
- How analytics rules relate to alerts and incidents
- How detections map to MITRE ATT&CK
- How detection content should align with available telemetry
- How analytics rules fit into the wider SIEM and SOAR architecture

The final detection workflow was:

```text
Security Telemetry
        |
        v
Microsoft Sentinel
        |
        v
Analytics Rule
        |
        v
Detection Condition
        |
        v
Security Alert
        |
        v
Sentinel Incident
        |
        v
Investigation / Automation
```

# 2. Detection Architecture

Microsoft Sentinel uses analytics rules to continuously evaluate security data for conditions that may represent suspicious or malicious activity.

The detection layer sits between collected telemetry and incident management.

```text
Azure / Microsoft Security Data
              |
              v
       Log Analytics
              |
              v
      Microsoft Sentinel
              |
              v
       Analytics Rules
              |
              v
            Alert
              |
              v
           Incident
```

This architecture separates several important concepts.

An analytics rule defines detection logic.

An alert represents activity detected by that logic.

An incident provides the investigation object used by analysts to investigate and manage related security activity.

# 3. Initial Analytics State

When I opened the Microsoft Sentinel Analytics section, the environment initially showed:

```text
Active Rules: 0
```

This meant that although Sentinel had been deployed and data integrations were available, there was no active analytics rule in the environment at that point.

This was an important distinction.

Deploying Microsoft Sentinel does not automatically mean that the environment has meaningful detection coverage.

The next step was therefore to review the available analytics content.

# 4. Available Analytics Templates

The environment presented three analytics templates relevant to the available content:

```text
Anomalous RDP Login Detection

Anomalous SSH Login Detection

Microsoft Defender Threat Intelligence Analytics
```

Rather than enabling all three, I evaluated whether each template aligned with the resources actually deployed in the environment.

# 5. Evaluating the RDP Detection Template

One of the available templates was:

```text
Anomalous RDP Login Detection
```

Remote Desktop Protocol activity is typically associated with Windows systems and virtual machines.

However, this project did not include virtual machines being monitored for RDP activity.

Enabling the rule simply to increase the number of active analytics rules would therefore have provided little value to the environment.

I decided not to enable it.

This followed the principle:

```text
Detection Content
       |
       v
Should Match
       |
       v
Available Telemetry
```

A detection rule is only useful when the environment contains the data required to support it.

# 6. Evaluating the SSH Detection Template

The second available template was:

```text
Anomalous SSH Login Detection
```

SSH detections are particularly relevant to Linux systems and infrastructure where SSH authentication telemetry is available.

The project did not include virtual machines or Linux workloads being monitored for SSH activity.

For the same reason as the RDP rule, I did not enable this template.

The decision was based on the architecture rather than the desire to populate the Analytics dashboard.

# 7. Selecting the Threat Intelligence Analytics Rule

The third available template was:

```text
Microsoft Defender Threat Intelligence Analytics
```

This rule aligned more closely with the Microsoft security services and threat-intelligence capabilities available in the environment.

I therefore selected this template for implementation.

The rule became the first active analytics rule in the Sentinel environment.

# 8. Rule Configuration

The implemented analytics rule had the following observed configuration:

| Property | Configuration |
|---|---|
| Rule | Microsoft Defender Threat Intelligence Analytics |
| Severity | Medium |
| Status | Enabled |
| MITRE ATT&CK Tactic | Persistence |
| MITRE ATT&CK Tactic | Lateral Movement |

The rule template description indicated that it generates alerts when Microsoft Defender Threat Intelligence indicators match activity found in event logs.

This introduced threat-intelligence-based detection into the environment.


# 9. Detection Workflow

The implemented detection workflow can be represented as:

```text
Event Telemetry
       |
       v
Microsoft Sentinel
       |
       v
Microsoft Defender
Threat Intelligence Analytics
       |
       v
Threat Intelligence Match
       |
       v
Security Alert
       |
       v
Sentinel Incident
```

If the rule's detection conditions are satisfied, the resulting alert can contribute to the creation of an incident for analyst investigation.


# 10. Analytics Rule vs Template

During the implementation, it was important to distinguish between an analytics template and an active analytics rule.

An analytics template is reusable detection content provided through Microsoft Sentinel.

```text
Analytics Template
       |
       v
Review Configuration
       |
       v
Create / Enable
       |
       v
Active Analytics Rule
```

Simply seeing a template in Sentinel does not mean the detection is active.

The template must be configured and enabled before it becomes part of the active detection environment.


# 11. Analytics Rule vs Detection Rule

The term "detection rule" is commonly used in security operations to describe logic that identifies suspicious behavior.

Within Microsoft Sentinel, this functionality is primarily implemented through:

```text
Analytics Rules
```

Therefore, in the context of this project:

```text
Detection Rule
      ≈
Sentinel Analytics Rule
```

However, I use the Microsoft Sentinel term "analytics rule" throughout the technical documentation where possible.


# 12. Analytics Rule vs Alert

An analytics rule and an alert are not the same thing.

The rule is the detection logic.

The alert is the result generated when that logic identifies activity matching its conditions.

```text
Analytics Rule
      |
      | Detection condition satisfied
      v
Security Alert
```

A rule can remain enabled without continuously generating alerts.

No alert does not automatically mean the rule is broken.

It may simply mean that the detection condition has not been satisfied.


# 13. Alert vs Incident

A security alert represents a detected security condition.

An incident is a higher-level investigation object used by the SOC.

Conceptually:

```text
Analytics Rule
      |
      v
Security Alert
      |
      v
Microsoft Sentinel Incident
      |
      v
SOC Investigation
```

An incident may contain one or more related alerts depending on the detection and correlation logic involved.

The incident becomes the object analysts use for activities such as:

- Triage
- Investigation
- Entity analysis
- Severity review
- Ownership
- Status management
- Task management
- Automation
- Escalation
- Closure

# 14. Incident State During Validation

After enabling the analytics rule, I reviewed the Sentinel Incidents interface.

At the time of the main validation:

```text
Active Incidents: 0
```

This was not treated as a failure.

The analytics rule was enabled and ready to evaluate applicable telemetry, but there was no incident in the queue requiring investigation at that time.

I did not manufacture suspicious activity or classify benign Key Vault activity as malicious simply to produce an incident screenshot.

# 15. Detection Pipeline

The completed detection pipeline was therefore:

```text
Telemetry
    |
    v
Analytics Rule
    |
    v
Potential Detection
    |
    v
Alert
    |
    v
Incident
    |
    v
Triage
```

The first stages were implemented and configured.

The later stages depend on activity satisfying the relevant detection conditions.

# 16. MITRE ATT&CK Mapping

The Microsoft Defender Threat Intelligence Analytics rule was mapped to the following ATT&CK tactics:

```text
Persistence

Lateral Movement
```

This allowed the detection to contribute to the ATT&CK coverage represented within Microsoft Sentinel.

The relationship was:

```text
Analytics Rule
      |
      v
Detection Capability
      |
      v
MITRE ATT&CK Mapping
      |
      +-- Persistence
      |
      +-- Lateral Movement
```

I later reviewed these mappings through Sentinel's MITRE ATT&CK coverage interface.

# 17. Persistence

Persistence represents attacker techniques used to maintain access to systems or environments.

Within the Sentinel ATT&CK interface, I was able to explore techniques associated with the Persistence tactic and review information such as:

```text
Technique ID
Description
Alerts
Incidents
Tactic
```

One of the techniques visible during the ATT&CK investigation was:

```text
Account Manipulation
```

The purpose of reviewing this information was not to claim that Account Manipulation had occurred in the environment.

Instead, it demonstrated how Sentinel connects detection content with the ATT&CK framework.

# 18. Lateral Movement

Lateral Movement represents techniques used by attackers to move between systems, identities, or resources after gaining access to an environment.

The enabled Threat Intelligence Analytics rule also mapped to this tactic.

Again, the ATT&CK mapping represents detection coverage.

It does not mean that lateral movement was observed during the project.

This distinction is important:

```text
ATT&CK Mapping
       !=
Confirmed Attack
```

# 19. Detection Coverage vs Security Posture

An enabled analytics rule contributes to detection coverage, but it does not mean the environment is completely protected.

For example:

```text
One Active Rule
      |
      v
Some Detection Coverage
      |
      X
Does Not Equal
      |
      v
Complete Security Coverage
```

A mature Sentinel environment would typically contain many analytics rules aligned with:

- Identity telemetry
- Endpoint telemetry
- Cloud infrastructure
- Network telemetry
- SaaS activity
- Threat intelligence
- Application logs
- Organization-specific threats

The scope of this project was significantly smaller.

# 20. Why I Did Not Enable Every Available Rule

I deliberately avoided enabling analytics rules that did not align with the environment.

For example:

```text
Anomalous RDP Login Detection
```

would be difficult to validate meaningfully without relevant RDP telemetry.

Similarly:

```text
Anomalous SSH Login Detection
```

would not provide meaningful coverage without SSH-enabled workloads generating the required telemetry.

This reflects a basic detection-engineering principle:

> Detection logic should be aligned with the telemetry and attack surface that actually exist in the environment.


# 21. Detection Engineering Considerations

The analytics-rule phase highlighted several detection-engineering considerations.

## Telemetry Dependency

Every detection depends on data.

If the required telemetry is missing, the detection cannot operate effectively.

## Detection Relevance

Rules should correspond to technologies and risks present in the environment.

## Validation

An enabled rule should eventually be validated against expected test conditions before being considered production-ready.

## False Positives

Detection logic should be tuned to reduce unnecessary alerts while preserving meaningful coverage.

## ATT&CK Mapping

ATT&CK mappings can help understand which attacker behaviors are covered and where detection gaps exist.

## Response Integration

High-value detections should connect to an incident-management and response workflow.

# 22. Relationship with Threat Hunting

Analytics rules and threat hunting serve different purposes.

Analytics rules provide continuous automated detection.

Threat hunting provides proactive analyst-driven investigation.

```text
                Security Telemetry
                       |
          +------------+------------+
          |                         |
          v                         v
   Analytics Rules             Threat Hunting
          |                         |
          v                         v
Automated Detection        Analyst Hypothesis
          |                         |
          v                         v
        Alert                  KQL Investigation
          |                         |
          v                         v
      Incident                 Hunt Conclusion
```

Both approaches were implemented or explored within this project.

# 23. Relationship with SOAR

Analytics rules also form part of the SOAR architecture.

When detection activity results in an incident, Sentinel automation can respond to that incident.

The project implemented:

```text
Analytics Rule
      |
      v
Alert
      |
      v
Incident
      |
      v
Contoso Sentinel Incident Triage
      |
      +--------------------------+
      |                          |
      v                          v
Automated-Triage Tag      Investigation Task
```

I also deployed the `MDTI-Automated-Triage` Logic App playbook to provide a more advanced automated triage workflow.

This demonstrates how detection can connect to response:

```text
Detect
   |
   v
Create Investigation Context
   |
   v
Automate Initial Triage
   |
   v
Analyst Decision
```

# 24. Detection and Human Decision-Making

I deliberately kept analyst judgment within the response workflow.

The native automation did not automatically:

```text
Close incidents

Change incident severity

Declare activity malicious

Perform containment
```

Instead, the automation prepared the incident for investigation.

This reflects the principle:

```text
Automation
    +
Analyst Judgment
    =
Controlled Response
```

Automation can remove repetitive work without automatically replacing security decisions that require context.


# 25. Detection Validation

The following components were validated during the analytics phase:

| Component | Validation |
|---|---|
| Analytics interface | Accessible |
| Initial active-rule state | 0 rules observed |
| Available templates | Reviewed |
| RDP template | Reviewed but not enabled |
| SSH template | Reviewed but not enabled |
| Threat Intelligence template | Selected |
| Threat Intelligence analytics rule | Created/enabled |
| Rule severity | Medium |
| Rule status | Enabled |
| Persistence mapping | Confirmed |
| Lateral Movement mapping | Confirmed |
| Incident dashboard | Reviewed |
| Active incidents during primary validation | 0 |

# 26. Evidence

The detection implementation is supported by screenshots showing:

```text
Analytics rule interface

Microsoft Defender Threat Intelligence Analytics rule

Enabled status

Medium severity

MITRE ATT&CK mappings

Incident dashboard
```

Recommended repository structure:

```text
images/
|
+-- analytics/
|   |
|   +-- analytics-rules-overview.png
|   +-- threat-intelligence-analytics-rule.png
|   +-- analytics-rule-details.png
|
+-- mitre/
|   |
|   +-- analytics-rule-mitre-mapping.png
|
+-- incidents/
    |
    +-- sentinel-incidents-overview.png
```

If some screenshots contain the same information, they do not need to be duplicated simply to populate every filename.

# 27. Limitations

The detection implementation had several limitations.

## Small Telemetry Volume

The environment contained significantly less telemetry than a production SOC.

## Single Active Analytics Rule

The project did not attempt to build a large detection-rule library.

## No RDP/SSH Workloads

The environment did not contain virtual machines generating the telemetry required to meaningfully validate the available RDP and SSH templates.

## No Active Incident During Primary Validation

There was no active security incident requiring investigation when I reviewed the incident queue.

## No Full Detection-Tuning Exercise

The available telemetry volume was not sufficient for meaningful long-term false-positive analysis or rule tuning.

These limitations are documented rather than hidden.


# 28. Potential Detection Improvements

A future version of the environment could expand the detection layer considerably.

Potential improvements include:

### Custom Scheduled Analytics Rules

Convert validated KQL hunting logic into scheduled analytics rules where appropriate.

### Identity Detection

Add detection scenarios using Microsoft Entra sign-in and audit telemetry.

### Endpoint Detection

Onboard Microsoft Defender for Endpoint telemetry and enable endpoint-focused analytics.

### Key Vault Detection

Develop custom rules for activity such as:

```text
Repeated failed Key Vault operations

Unexpected secret access

Unusual access times

Unexpected source identities

Unusual source IP addresses

Rare Key Vault operations

High-frequency access patterns
```

### Entity Mapping

Map relevant query fields to Sentinel entities such as:

```text
Account

IP Address

Host

Azure Resource
```

### ATT&CK Gap Analysis

Review which tactics and techniques lack detection coverage.

### Detection Testing

Generate controlled test activity to validate that custom rules create the expected alerts and incidents.

# 29. Detection Maturity Model

The implementation demonstrated the beginning of a detection-engineering lifecycle.

```text
Telemetry
    |
    v
Exploratory KQL
    |
    v
Threat Hunting
    |
    v
Validated Logic
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
Automation
    |
    v
Analyst Review
    |
    v
Detection Tuning
```

Not every stage was completed for every custom query.

For example, the Key Vault hunting queries remained hunting and analysis queries rather than being presented as production-ready analytics rules.

That distinction prevents experimental investigation logic from being misrepresented as a fully validated detection.

# 30. Final Detection State

The final detection configuration can be summarized as:

```text
Initial Active Rules:
0

Available Templates Reviewed:
3

Rules Enabled:
1

Enabled Rule:
Microsoft Defender Threat Intelligence Analytics

Severity:
Medium

MITRE ATT&CK:
Persistence
Lateral Movement

Active Incidents During Primary Review:
0
```

The project therefore established an active detection layer while remaining aligned with the telemetry and resources actually available in the environment.

# 31. Key Lessons

The analytics phase reinforced several important lessons.

### Sentinel Deployment Is Not the Same as Detection Coverage

A Sentinel workspace can exist without active analytics rules.

### More Rules Do Not Automatically Mean Better Security

Rules should align with the actual attack surface and telemetry.

### Alerts and Incidents Are Different Objects

The analytics rule provides the detection logic, an alert represents a detection, and an incident provides the investigation context.

### ATT&CK Mapping Represents Coverage, Not Confirmed Attacks

Seeing Persistence or Lateral Movement in a rule configuration does not mean those behaviors occurred.

### Detection Should Lead Somewhere

Useful detection architecture needs an investigation and response process behind it.

# 32. Final Detection Workflow

The detection component of the project ultimately fit into the larger security operations architecture as follows:

```text
Azure / Microsoft Telemetry
            |
            v
       Log Analytics
            |
            v
    Microsoft Sentinel
            |
            v
      Analytics Rule
            |
            v
Microsoft Defender Threat
Intelligence Analytics
            |
            v
     Potential Alert
            |
            v
     Sentinel Incident
            |
            +----------------------+
            |                      |
            v                      v
     Analyst Triage           SOAR Automation
            |                      |
            +----------+-----------+
                       |
                       v
                Security Decision
```

This established the detection layer required to connect telemetry collection with incident investigation and automated response.

## Related Documentation

- [Architecture](Architecture.md)
- [Deployment Guide](DeploymentGuide.md)
- [Data Connectors](DataConnectors.md)
- [KQL Queries](KQLQueries.md)
- [Incident Management](IncidentManagement.md)
- [Threat Hunting](ThreatHunting.md)
- [MITRE ATT&CK Mapping](MITRE-ATTACK-Mapping.md)
- [SOAR Automation](SOAR-Automation.md)
- [Validation Report](ValidationReport.md)
- [Troubleshooting](Troubleshooting.md)
