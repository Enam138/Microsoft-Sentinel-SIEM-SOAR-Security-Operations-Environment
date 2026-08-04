# Microsoft Sentinel Incident Management

## 1. Overview

This document describes the incident-management component of the Microsoft Sentinel SIEM and SOAR environment.

Incident management sits between security detection and response.

Microsoft Sentinel analytics rules can generate alerts when detection conditions are satisfied. Those alerts can then become part of incidents that provide analysts with a centralized object for investigation, triage, ownership, evidence review, automation, and closure.

The incident-management workflow can be represented as:

```text
Security Telemetry
        |
        v
Analytics Rule
        |
        v
Security Alert
        |
        v
Microsoft Sentinel Incident
        |
        +----------------------+
        |                      |
        v                      v
Analyst Investigation     Automation
        |                      |
        +----------+-----------+
                   |
                   v
             Triage Decision
                   |
          +--------+--------+
          |        |        |
          v        v        v
       Close    Escalate   Respond
```

During the primary validation period, there were no active incidents requiring investigation.

Rather than generating artificial findings simply to populate the incident queue, I focused on understanding the incident workflow and configuring automation that would prepare future incidents for SOC investigation.


# 2. Role of Incidents in Microsoft Sentinel

An incident provides the investigation context surrounding potentially related security activity.

The basic relationship is:

```text
Telemetry
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

These components should not be treated as interchangeable.

An analytics rule contains detection logic.

An alert represents a detection generated when applicable conditions are satisfied.

An incident organizes security activity into an investigation object that can be handled by a SOC analyst.


# 3. Incident Management Architecture

Within the project, incident management connects the SIEM detection layer with the SOAR response layer.

```text
                   Microsoft Sentinel
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
             +------------+------------+
             |                         |
             v                         v
       Analyst Triage            Automation Rule
             |                         |
             |                  Add Triage Tag
             |                         |
             |                  Add Investigation Task
             |                         |
             +------------+------------+
                          |
                          v
                   Investigation
                          |
                          v
                    SOC Decision
```

This makes the incident the central object around which both analyst activity and automation can operate.

# 4. Reviewing the Incidents Interface

After enabling the Microsoft Defender Threat Intelligence Analytics rule, I opened the Microsoft Sentinel Incidents interface.

At the time of validation, the environment showed:

```text
0 active incidents
```

The incident-management interface still exposed the functionality that would be used when incidents were generated.

The interface included capabilities for reviewing and managing incident properties such as:

- Incident ID
- Alerts
- Description
- Severity
- Status
- Owner
- Tactics
- Entities
- Related activity
- Investigation information

This provided visibility into how a SOC analyst would work with an incident once one was created.


# 5. Incident Status

Incident status is used to track where an investigation currently sits in the SOC workflow.

Typical operational states include:

```text
New
 |
 v
In Progress
 |
 v
Closed
```

A newly generated incident normally enters the investigation queue before being assigned or investigated.

An analyst can then move the incident through the appropriate lifecycle as evidence is reviewed.

The incident status should reflect the actual investigation state rather than being changed solely for reporting purposes.

# 6. Incident Severity

Severity helps analysts prioritize the investigation queue.

Sentinel incidents can be categorized using severity levels such as:

```text
High
Medium
Low
Informational
```

Severity is useful for prioritization, but it should not be treated as proof that malicious activity has occurred.

For example:

```text
High Severity
      !=
Confirmed Compromise
```

Severity indicates the potential importance of the detection and should be considered alongside evidence, entities, context, and investigation results.


# 7. Incident Ownership

Incident ownership provides accountability during investigation.

In a larger SOC environment, incidents can be assigned to analysts or teams responsible for investigation.

A typical workflow might look like:

```text
Incident Created
       |
       v
SOC Queue
       |
       v
Assigned to Analyst
       |
       v
Investigation
       |
       v
Escalation / Closure
```

The project did not have an active incident requiring assignment during the primary validation period, but I reviewed the ownership capability as part of the incident-management workflow.


# 8. Alerts Within an Incident

Alerts provide the detection information that contributes to an incident.

The relationship can be represented as:

```text
Analytics Rule
      |
      v
Security Alert
      |
      v
Incident
```

Depending on the detection and correlation logic, an incident may contain one or multiple alerts.

Analysts can use the alerts to understand:

- What triggered the detection
- When the activity occurred
- Which detection rule generated it
- Which entities were involved
- Which tactics or techniques may be relevant
- What additional evidence should be investigated


# 9. Entities

Entities provide investigation context around the objects associated with security activity.

Examples can include:

```text
Users

Hosts

IP Addresses

Accounts

Azure Resources

URLs

Domains
```

Entity context becomes particularly important when moving from an alert to an investigation.

Instead of only asking:

```text
What alert fired?
```

the analyst can ask:

```text
Which user was involved?

Which host was involved?

Which IP address was involved?

What other activity is associated with the entity?

Has the entity appeared in other incidents?
```

Entity processing also became important later in the SOAR implementation.

# 10. Entities and the MDTI Playbook

The `MDTI-Automated-Triage` Logic App deployed later in the project was designed to process entities associated with Sentinel incidents.

The visible workflow included actions for:

```text
Get Hosts

Get IPs

Process Hosts

Process IP Addresses

Evaluate Threat Classification
```

This demonstrates why incident entities are important beyond the analyst interface.

They can also become inputs into automated enrichment and triage workflows.

```text
Sentinel Incident
       |
       v
Incident Entities
       |
       +----------------+
       |                |
       v                v
     Hosts          IP Addresses
       |                |
       v                v
Automated Processing / Enrichment
       |
       v
Triage Context
```


# 11. MITRE ATT&CK Context

Sentinel incidents can also contain ATT&CK-related information associated with the underlying detections.

The Microsoft Defender Threat Intelligence Analytics rule used in this project was mapped to:

```text
Persistence

Lateral Movement
```

If an applicable detection generated an incident, these mappings could provide additional behavioral context for investigation.

However:

```text
ATT&CK Mapping
       !=
Confirmed ATT&CK Technique Execution
```

The mapping helps analysts understand the behavior the detection is designed to identify.


# 12. Primary Validation Result

When I reviewed the Incidents page, there were:

```text
0 active incidents
```

This result was retained in the documentation rather than hidden.

It meant that during the selected validation period, there was no active Sentinel incident requiring investigation.

This did not prevent me from configuring and validating the architecture surrounding incident management.


# 13. Why I Did Not Manufacture an Incident

It would have been possible to attempt to create activity solely for the purpose of obtaining an incident screenshot.

I chose not to treat that as necessary for the project.

The Key Vault telemetry available during the investigation showed successful operations, and the custom failed-operation hunting query returned:

```text
0 matching events
```

The threat hunt also did not produce evidence that justified escalation.

Creating a security narrative unsupported by the available telemetry would have weakened the project rather than improved it.

I therefore documented the actual state:

```text
Analytics capability: Enabled

Incident management interface: Reviewed

Active incidents during primary validation: 0

Artificial malicious finding: Not created
```


# 14. Incident Triage Design

Although there were no active incidents requiring manual investigation, I designed the automation architecture around how future incidents should be triaged.

The objective was to automate repetitive preparation without allowing automation to make unsupported security conclusions.

The desired workflow was:

```text
Incident Created
       |
       v
Initial Automation
       |
       v
Prepare Investigation Context
       |
       v
SOC Analyst Review
       |
       v
Evidence-Based Decision
```

This led to the creation of a native Microsoft Sentinel automation rule.


# 15. Contoso Sentinel Incident Triage

I created the automation rule:

```text
Contoso Sentinel Incident Triage
```

The rule was configured to trigger:

```text
When incident is created
```

This means the workflow is designed to execute automatically when a new Sentinel incident enters the environment.


# 16. Automation Rule Actions

The rule performs two initial triage actions.

## Action 1: Add Tag

The first action adds:

```text
Automated-Triage
```

to the incident.

This provides a simple way to identify incidents that have entered the automated triage process.

The workflow becomes:

```text
New Incident
     |
     v
Automation Rule
     |
     v
Add Automated-Triage Tag
```


# 17. Investigation Task

The second automation action creates:

```text
Perform Initial SOC Investigation
```

The task is intended to guide the analyst through the initial review.

The investigation should consider areas such as:

- Associated alerts
- Affected entities
- MITRE ATT&CK mappings
- Supporting telemetry
- Related activity
- Detection context
- Whether escalation is justified
- Whether containment is required
- Whether the incident can be closed

The resulting workflow becomes:

```text
New Incident
      |
      v
Contoso Sentinel Incident Triage
      |
      +---------------------------+
      |                           |
      v                           v
Automated-Triage Tag       Investigation Task
      |                           |
      +-------------+-------------+
                    |
                    v
              Analyst Review
```


# 18. Why I Did Not Automatically Change Severity

Sentinel automation rules can perform actions such as changing incident severity.

I deliberately avoided configuring automatic severity modification in the initial workflow.

Changing severity without sufficient context could incorrectly prioritize or deprioritize an incident.

Instead:

```text
Automation
    |
    v
Prepare Incident
    |
    v
Analyst Reviews Evidence
    |
    v
Severity Decision
```

This keeps the decision tied to investigation context.


# 19. Why I Did Not Automatically Close Incidents

The automation-rule interface also supports status changes.

I did not configure newly created incidents to be automatically closed.

Automatic closure can be useful in mature environments where a detection has been extensively tuned and the closure logic is well understood.

That was not the purpose of this project.

The workflow therefore preserves analyst judgment:

```text
Incident
    |
    v
Automated Initial Triage
    |
    v
Analyst Investigation
    |
    +----------------------+
    |                      |
    v                      v
Close                  Escalate
```


# 20. Why I Did Not Automatically Assign an Owner

Owner assignment was available as an automation action.

However, the project was not operating a multi-analyst SOC with an assignment queue.

Automatically assigning every incident to an arbitrary owner would not have added meaningful operational value.

In a larger environment, ownership automation could be based on:

- SOC team
- Severity
- Detection type
- Business unit
- Geography
- On-call schedule
- Incident category

For this project, the focus remained on automated triage preparation.

# 21. Available Native Automation Actions

During configuration, the Sentinel automation-rule interface exposed actions including:

```text
Run Logic App Playbook

Change Status

Change Severity

Assign Owner

Add Tags

Add Task
```

It also supported adding multiple actions to the same automation rule.

I selected:

```text
Add Tags

Add Task
```

because they aligned with the initial triage objective while preserving analyst control over the investigation outcome.


# 22. Rule Trigger Options

The interface provided trigger options including:

```text
When incident is created

When incident is updated

When alert is created
```

For the triage workflow, I selected:

```text
When incident is created
```

because I wanted the initial preparation to occur as soon as a new incident entered the Sentinel investigation queue.


# 23. Rule Order and Expiration

The automation configuration also exposed:

```text
Order

Expiration
```

These settings are important when multiple automation rules exist.

Order can influence the sequence in which applicable rules are processed.

Expiration can be used when an automation rule should only remain active for a defined period.

The project contained a limited automation-rule set, so complex ordering logic was not required.


# 24. Incident Management and SOAR

The native automation rule established the first SOAR layer.

The incident-management architecture became:

```text
Detection
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
    +------------------------+
    |                        |
    v                        v
Add Tag                 Add Task
    |                        |
    +-----------+------------+
                |
                v
          Analyst Review
```

I then extended the SOAR architecture using Azure Logic Apps.


# 25. Logic App Integration

Microsoft Sentinel automation rules can execute Logic App playbooks.

The project later deployed:

```text
MDTI-Automated-Triage
```

The playbook was designed to receive:

```text
Sentinel Incident
```

as its input.

This created a second automation path:

```text
Sentinel Incident
       |
       v
Azure Logic App
       |
       v
MDTI-Automated-Triage
       |
       v
Entity Processing
       |
       v
Threat Classification
```

This demonstrates how Sentinel incidents can act as the bridge between SIEM detections and SOAR workflows.


# 26. Native Automation vs Logic App Automation

The project used two different automation approaches.

| Capability | Native Automation Rule | Logic App Playbook |
|---|---|---|
| Trigger on incident | Yes | Yes |
| Add tag | Yes | Possible |
| Add investigation task | Yes | Possible |
| Change incident fields | Yes | Yes |
| Entity processing | Limited | Advanced |
| External integrations | Limited | Extensive |
| Complex branching | Limited | Yes |
| Threat-intelligence workflow | Limited | Yes |
| Workflow orchestration | Basic | Advanced |

The two approaches are complementary rather than mutually exclusive.


# 27. Incident Triage Model

The final triage model can be represented as:

```text
                   New Incident
                        |
                        v
             Native Automation Rule
                        |
              +---------+---------+
              |                   |
              v                   v
          Add Tag             Add Task
              |                   |
              +---------+---------+
                        |
                        v
                Investigation Ready
                        |
              +---------+---------+
              |                   |
              v                   v
         Analyst Review      Logic App SOAR
              |                   |
              |                   v
              |             Entity Processing
              |                   |
              |                   v
              |              Enrichment /
              |              Classification
              |                   |
              +---------+---------+
                        |
                        v
                  SOC Decision
```

This preserves human decision-making while allowing automation to handle repeatable tasks.


# 28. Recommended Analyst Investigation Workflow

For a future incident entering the environment, the initial investigation process would be:

### Step 1: Review Incident Metadata

Review:

```text
Incident ID
Title
Severity
Status
Creation Time
Analytics Rule
```

### Step 2: Review Alerts

Determine:

```text
What triggered the incident?

Which rule generated the alert?

How many alerts are associated with it?
```

### Step 3: Review Entities

Investigate:

```text
Accounts
Hosts
IP Addresses
Azure Resources
Domains
URLs
```

where applicable.

### Step 4: Review ATT&CK Context

Determine which tactics or techniques are associated with the detection.

### Step 5: Review Supporting Telemetry

Use KQL to inspect the underlying events.

### Step 6: Check Related Activity

Determine whether the same entities appear elsewhere in the environment.

### Step 7: Review Automation Results

Check:

```text
Tags

Tasks

Playbook output

Enrichment information
```

### Step 8: Make a Security Decision

Possible outcomes include:

```text
Continue Investigation

Escalate

Contain

Close as Benign

Close as False Positive

Close after Remediation
```

The final decision should be based on evidence rather than automation alone.

# 29. Incident Evidence

The incident-management portion of the project can be supported with screenshots showing:

```text
Sentinel Incidents interface

0 active incidents state

Available incident filters

Automation rule configuration

Incident-created trigger

Automated-Triage tag action

Perform Initial SOC Investigation task action
```

Recommended repository structure:

```text
images/
|
+-- incidents/
|   |
|   +-- sentinel-incidents-overview.png
|
+-- automation/
    |
    +-- sentinel-automation-rule.png
    +-- incident-created-trigger.png
    +-- automated-triage-actions.png
```

Only screenshots that were actually captured should be added.

# 30. Incident Management Validation

The following areas were validated:

| Component | Result |
|---|---|
| Sentinel Incidents interface | Reviewed |
| Incident fields and controls | Reviewed |
| Active incidents during primary validation | 0 |
| Incident-created automation trigger | Configured |
| Automated triage tag | Configured |
| Investigation task | Configured |
| Automatic severity change | Not configured |
| Automatic incident closure | Not configured |
| Automatic owner assignment | Not configured |
| Logic App incident integration | Deployed separately |
| Sentinel incident trigger in playbook | Successfully executed |


# 31. Limitations

The incident-management portion of the project has several limitations.

## No Active Security Incident During Primary Validation

The environment did not contain an active incident requiring a complete analyst-led investigation.

## No Multi-Analyst SOC

Ownership, queue management, escalation between SOC tiers, and analyst workload distribution were not tested.

## No Production SLA

The environment did not implement formal incident response service-level agreements.

## No Ticketing Integration

Incidents were not integrated with an external ticketing or case-management system.

## Limited Detection Volume

The environment did not generate the volume of detections expected in a production SOC.

These limitations should be considered when evaluating the scope of the implementation.


# 32. Future Improvements

A future version of the project could expand incident management by implementing:

### Controlled Detection Testing

Generate safe, controlled test telemetry that triggers a custom analytics rule and creates a real Sentinel incident.

### Full Incident Investigation

Document:

```text
Alert review
Entity investigation
KQL pivoting
Timeline analysis
Evidence collection
Classification
Closure
```

### Automated Ownership

Assign incidents based on severity or detection category.

### Severity-Based Automation

Run different workflows for High, Medium, and Low severity incidents.

### Notification Integration

Send incident notifications through approved communication platforms.

### Ticketing Integration

Create external SOC tickets automatically for selected incidents.

### SLA Tracking

Measure:

```text
Mean Time to Acknowledge

Mean Time to Investigate

Mean Time to Respond

Mean Time to Resolve
```

### Automated Enrichment

Use Logic Apps to enrich entities with additional threat-intelligence context before analyst review.


# 33. Key Lessons

The incident-management phase reinforced several important concepts.

### An Alert Is Not an Incident

Alerts represent detections. Incidents provide the investigation context.

### Severity Is Not Proof

A high-severity incident still requires investigation.

### Zero Incidents Is a Valid Environment State

The absence of active incidents should not be replaced with fabricated findings.

### Automation Should Prepare, Not Automatically Decide

The native triage workflow handles repetitive preparation while leaving security conclusions to the analyst.

### Entities Connect SIEM and SOAR

Incident entities provide context for analysts and inputs for automated playbooks.

### Incident Management Connects Detection to Response

Without an investigation process, detection alone does not complete the SOC workflow.


# 34. Final Incident Management Architecture

The completed incident-management design can be summarized as:

```text
Security Telemetry
       |
       v
Analytics Rule
       |
       v
Alert
       |
       v
Sentinel Incident
       |
       +--------------------------------+
       |                                |
       v                                v
Native Automation                Logic App SOAR
       |                                |
       +-- Automated-Triage Tag         +-- Get Hosts
       |                                |
       +-- Investigation Task           +-- Get IPs
       |                                |
       v                                +-- Process Entities
Analyst Investigation                   |
       |                                +-- Classification
       |                                |
       +---------------+----------------+
                       |
                       v
                Evidence Review
                       |
                       v
                  SOC Decision
```

This provides the incident-management foundation connecting Microsoft Sentinel detection, analyst investigation, and automated response.


## Related Documentation

- [Architecture](Architecture.md)
- [Deployment Guide](DeploymentGuide.md)
- [KQL Queries](KQLQueries.md)
- [Analytics Rules](AnalyticsRules.md)
- [Threat Hunting](ThreatHunting.md)
- [MITRE ATT&CK Mapping](MITRE-ATTACK-Mapping.md)
- [Workbooks](Workbooks.md)
- [SOAR Automation](SOAR-Automation.md)
- [Validation Report](ValidationReport.md)
- [Troubleshooting](Troubleshooting.md)
