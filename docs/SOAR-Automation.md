# Microsoft Sentinel SOAR Automation and Automated Incident Triage

## 1. Overview

This document describes the Security Orchestration, Automation, and Response (SOAR) capabilities implemented within the Microsoft Sentinel environment.

The automation implementation consisted of two complementary layers:

```text
Layer 1:
Microsoft Sentinel Native Automation Rule

Layer 2:
Azure Logic Apps Security Playbook
```

The native automation rule was created as:

```text
Contoso Sentinel Incident Triage
```

and was designed to automatically prepare newly created incidents for analyst investigation.

I also deployed the Microsoft Defender Threat Intelligence playbook:

```text
MDTI-Automated-Triage
```

which introduced a more advanced Logic Apps workflow capable of receiving Sentinel incidents and processing associated entities.

The final SOAR architecture therefore combined:

```text
Microsoft Sentinel
        +
Native Automation
        +
Azure Logic Apps
        +
Managed Identity
        +
Azure RBAC
        +
Analyst Review
```

The objective was not to remove the analyst from the response process.

Instead, automation was used to reduce repetitive triage work while keeping security decisions under analyst control.


# 2. What Is SOAR?

SOAR stands for:

```text
Security Orchestration
Automation
and
Response
```

A SIEM primarily provides capabilities such as:

```text
Telemetry Collection

Log Analysis

Detection

Alerting

Investigation
```

SOAR extends this by introducing automated workflows.

```text
SIEM
 |
 v
Detect
 |
 v
Alert
 |
 v
Incident
 |
 v
SOAR
 |
 v
Automated Actions
 |
 v
Analyst Review / Response
```

Microsoft Sentinel provides SOAR capabilities primarily through:

```text
Automation Rules

Azure Logic Apps Playbooks
```

Both were explored and implemented during this project.


# 3. SOAR Architecture

The final automation architecture was:

```text
                    Security Telemetry
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
                           |
              +------------+------------+
              |                         |
              v                         v
      Native Automation          Logic App Playbook
              |                         |
              v                         v
     Automated-Triage Tag       MDTI-Automated-Triage
              |                         |
              v                         v
      Investigation Task          Entity Processing
              |                         |
              +------------+------------+
                           |
                           v
                     Analyst Review
                           |
                           v
                    Security Decision
```

This architecture provides both simple incident automation and more advanced workflow orchestration.


# 4. Initial Automation State

When I opened the Microsoft Sentinel Automation section, the environment initially showed:

```text
0 existing automation rules
```

The interface provided an option to create a new automation rule.

The rule creation interface included:

```text
Rule Type

Name

Trigger

Actions

Expiration

Order
```

The available rule types included:

```text
Enhanced Rule

Standard Rule
```

This provided the starting point for the native Sentinel automation implementation.


# 5. Automation Rule Trigger Options

During configuration, Microsoft Sentinel provided several trigger options:

```text
When incident is created

When incident is updated

When alert is created
```

The automation objective was to prepare new incidents immediately when they entered the SOC investigation queue.

I therefore selected:

```text
When incident is created
```

The resulting workflow was:

```text
New Sentinel Incident
         |
         v
Automation Rule Triggered
         |
         v
Initial Triage Actions
```


# 6. Available Automation Actions

The rule configuration exposed several possible actions:

```text
Run Logic App Playbook

Change Status

Change Severity

Assign Owner

Add Tags

Add Task
```

Additional actions could also be added to the same rule.

Rather than automatically changing security-sensitive incident properties, I selected actions that would prepare the incident for analyst investigation.


# 7. Contoso Sentinel Incident Triage

The native automation rule was created as:

```text
Contoso Sentinel Incident Triage
```

Its purpose was to standardize the first stage of incident handling.

The workflow was designed as:

```text
Incident Created
       |
       v
Contoso Sentinel Incident Triage
       |
       +---------------------------+
       |                           |
       v                           v
Add Automated-Triage Tag    Add Investigation Task
       |                           |
       +-------------+-------------+
                     |
                     v
               Analyst Review
```

This creates a consistent starting point for future Sentinel incidents.


# 8. Automated-Triage Tag

The first action configured in the automation rule was:

```text
Add Tags
```

with the tag:

```text
Automated-Triage
```

The tag provides a simple visual indication that the incident has passed through the automated initial triage workflow.

Conceptually:

```text
Incident
   |
   v
Automation Rule
   |
   v
Automated-Triage
```

In a larger SOC, tags could also support filtering, reporting, queue management, and additional automation conditions.


# 9. Investigation Task

The second action configured was:

```text
Add Task
```

The task was created as:

```text
Perform Initial SOC Investigation
```

This provides the analyst with an explicit investigation action when the incident is opened.

The investigation can include reviewing:

```text
Associated Alerts

Affected Entities

MITRE ATT&CK Context

Supporting Telemetry

Related Activity

Automation Results

Potential Escalation Requirements
```

This combines automation with human decision-making.


# 10. Why the Native Workflow Was Intentionally Conservative

Microsoft Sentinel supported more aggressive actions, including:

```text
Change Status

Change Severity

Assign Owner

Run Playbook
```

I deliberately kept the first native automation rule conservative.

The workflow did not automatically:

```text
Close incidents

Declare activity malicious

Change severity

Perform containment

Assign an arbitrary analyst
```

Instead:

```text
Automation
    |
    v
Prepare Investigation
    |
    v
Analyst Reviews Evidence
    |
    v
Security Decision
```

This reduced the risk of automation making unsupported decisions.


# 11. Automation Rule Order

The configuration also included an:

```text
Order
```

field.

Order becomes particularly important when multiple automation rules can respond to the same incident.

For example:

```text
Incident Created
      |
      v
Rule 1 - Initial Classification
      |
      v
Rule 2 - Enrichment
      |
      v
Rule 3 - Notification
      |
      v
Rule 4 - Response
```

The current project contained a limited automation-rule set, so complex ordering was not required.


# 12. Automation Rule Expiration

The rule interface also included:

```text
Expiration
```

This can be useful when automation is intended to operate only for a defined period.

Examples could include:

```text
Temporary monitoring campaigns

Incident-specific automation

Short-term threat response

Testing periods
```

The feature was reviewed as part of the automation-rule configuration.


# 13. Moving Beyond Native Automation

The native Sentinel rule provided useful incident preparation, but more advanced SOAR workflows require greater orchestration capability.

Microsoft Sentinel integrates with:

```text
Azure Logic Apps
```

through security playbooks.

Logic Apps can provide:

```text
Conditional logic

Loops

Entity processing

External API integration

Threat-intelligence enrichment

Complex branching

Automated response actions
```

I therefore moved from native automation to the Playbooks environment.


# 14. Initial Playbook State

When I opened Microsoft Sentinel Playbooks, the environment initially showed:

```text
0 active playbooks
```

The interface provided options including:

```text
Create Playbook

Playbook Templates
```

I reviewed the available templates to identify an appropriate security automation workflow.

# 15. Available MDTI Playbook Templates

The environment provided several Microsoft Defender Threat Intelligence-related playbook templates.

The visible templates included:

```text
MDTI-Automated-Triage

MDTI-Intel-Reputation

MDTI-Data-Cookies

MDTI-Data-Trackers

MDTI-Data-ReverseDns

MDTI-Data-PassiveDns

MDTI-Data-WebComp...
```

Rather than deploying every template, I selected the workflow most closely aligned with automated SOC incident triage.

That template was:

```text
MDTI-Automated-Triage
```

# 16. Why MDTI-Automated-Triage Was Selected

The selected playbook was designed around automated threat-intelligence triage.

This aligned with the broader project architecture:

```text
Microsoft Sentinel Incident
          |
          v
Automated Triage
          |
          v
Entity Processing
          |
          v
Threat Intelligence Context
          |
          v
Analyst Investigation
```

It also complemented the native:

```text
Contoso Sentinel Incident Triage
```

rule.

The native automation handled simple incident preparation.

The Logic App demonstrated more advanced SOAR orchestration.


# 17. Playbook Deployment

I selected:

```text
Create Playbook
```

from the MDTI automated triage template.

The Azure deployment completed successfully.

This created the Logic App:

```text
MDTI-Automated-Triage
```

The successful deployment confirmed that the playbook infrastructure itself could be provisioned in the environment.


# 18. Logic App Workflow

After deployment, I opened the Logic App workflow.

The workflow contained a Microsoft Sentinel incident trigger and several downstream actions.

At a high level:

```text
Microsoft Sentinel Incident
          |
          v
Get Incident Information
          |
          v
Extract Entities
          |
     +----+----+
     |         |
     v         v
   Hosts       IPs
     |         |
     v         v
Process     Process
Hosts       IP Addresses
     |         |
     +----+----+
          |
          v
Threat Intelligence Logic
          |
          v
Classification / Triage
```

This demonstrated significantly more advanced automation than the native Sentinel automation rule.


# 19. Microsoft Sentinel Connection

When reviewing the Logic App connections, I confirmed that the workflow contained a:

```text
Microsoft Sentinel
```

connection.

This connection is important because the Logic App requires access to Sentinel data and incident context.

The relationship is:

```text
Microsoft Sentinel
       |
       v
Logic App Connection
       |
       v
MDTI-Automated-Triage
```

Without the appropriate connection and authorization, the playbook would not be able to interact correctly with Sentinel.


# 20. Incident Trigger

The workflow was configured to receive:

```text
Microsoft Sentinel Incident
```

as its trigger.

Conceptually:

```text
Sentinel Incident
      |
      v
Logic App Trigger
      |
      v
Workflow Execution
```

This makes the playbook incident-oriented rather than simply running as an unrelated scheduled workflow.


# 21. Entity Extraction

The playbook included actions for retrieving incident entities.

Visible entity-processing stages included:

```text
Get Hosts

Get IPs
```

This is important because threat-intelligence triage often requires extracting indicators and entities from the incident before they can be enriched.

For example:

```text
Incident
   |
   v
Entities
   |
   +----------------+
   |                |
   v                v
Hosts          IP Addresses
   |                |
   v                v
Enrichment / Investigation
```


# 22. Host Processing

The workflow included logic for processing hosts associated with the Sentinel incident.

Host information can provide useful context during an investigation, including:

```text
Affected system

Related endpoint

Potential infrastructure relationships

Associated threat intelligence
```

The workflow could therefore process host entities separately from other entity types.


# 23. IP Address Processing

The workflow also included logic for processing IP addresses.

IP addresses are particularly useful for threat-intelligence enrichment because they can be evaluated for reputation or known malicious associations.

The conceptual process is:

```text
Incident
   |
   v
Extract IP Address
   |
   v
Threat Intelligence Lookup
   |
   v
Evaluate Context
   |
   v
Support Triage Decision
```


# 24. Threat Classification

The playbook contained additional logic for evaluating threat information and classification.

This demonstrates a more advanced SOAR pattern:

```text
Collect Entity
      |
      v
Enrich Entity
      |
      v
Evaluate Intelligence
      |
      v
Classify Result
      |
      v
Provide Investigation Context
```

This can reduce the amount of repetitive enrichment work an analyst performs manually.


# 25. Managed Identity

After deploying the Logic App, I reviewed:

```text
Identity
```

within the Azure resource.

The Logic App's:

```text
System Assigned Managed Identity
```

was:

```text
On
```

This meant Azure had created an identity specifically for the Logic App resource.

The identity could then be granted Azure permissions through Role-Based Access Control.


# 26. Why Managed Identity Matters

A managed identity allows an Azure resource to authenticate to other Azure services without embedding long-lived credentials directly into application code.

The security model becomes:

```text
Logic App
    |
    v
System Assigned Identity
    |
    v
Azure RBAC
    |
    v
Authorized Resource Access
```

This is preferable to storing credentials directly inside workflow logic.


# 27. Initial Role Assignment State

When I reviewed the managed identity's Azure role assignments, the initial state showed:

```text
0 role assignments
```

This meant the identity existed, but it had not yet been granted an Azure RBAC role.

An identity without the required authorization may be unable to perform Sentinel operations even if authentication succeeds.


# 28. Role Eligibility Limitation

The interface also displayed a message indicating that role eligibility required:

```text
Microsoft Entra ID P2

or

Microsoft Entra ID Governance
```

The project environment did not have the required licensing for that capability.

This limitation was documented rather than treated as a deployment failure.

Role eligibility and standard Azure RBAC role assignment are different capabilities.

I therefore proceeded using the role-assignment functionality available in the environment.


# 29. Microsoft Sentinel Contributor Role

To provide the Logic App identity with the required Sentinel permissions, I added:

```text
Microsoft Sentinel Contributor
```

to the system-assigned managed identity.

The role assignment completed successfully.

The authorization model became:

```text
MDTI-Automated-Triage
        |
        v
System Assigned Managed Identity
        |
        v
Microsoft Sentinel Contributor
        |
        v
Microsoft Sentinel Resources
```

This established Azure RBAC authorization for the playbook.


# 30. Why RBAC Was Required

The Logic App needed more than a trigger and connection.

It also required appropriate authorization to perform operations against Sentinel resources.

This demonstrates the difference between:

```text
Authentication
```

and:

```text
Authorization
```

The managed identity establishes an Azure identity.

RBAC determines what that identity is permitted to do.

```text
Identity
   |
   v
Who are you?

RBAC
   |
   v
What are you allowed to do?
```

Both are necessary for secure automation.


# 31. Least Privilege Consideration

Permissions should be scoped to what the workflow actually requires.

The objective was not to assign broad Azure permissions simply to eliminate authorization errors.

Instead, the playbook was given a Sentinel-specific role appropriate to the security automation scenario.

This follows the principle:

```text
Grant Required Access
        |
        v
Avoid Unnecessary Privilege
```

In a production deployment, the exact required actions and scope should be reviewed carefully and reduced further where possible.


# 32. ThreatIntelligence.Read.All Limitation

During the playbook configuration and validation process, I encountered an additional permission requirement related to:

```text
ThreatIntelligence.Read.All
```

The required permission could not be located/configured through the available environment in the way expected during the exercise.

Rather than hiding this limitation, I documented it as part of the implementation.

This meant that although the playbook was successfully deployed and the Sentinel trigger could be tested, some Microsoft Defender Threat Intelligence enrichment functionality could remain dependent on permissions or licensing beyond the available environment.


# 33. Why This Limitation Matters

A deployed Logic App is not automatically equivalent to every downstream integration being fully authorized.

The complete chain can involve:

```text
Playbook Deployment
       |
       v
Sentinel Connection
       |
       v
Managed Identity
       |
       v
Azure RBAC
       |
       v
API Permissions
       |
       v
Licensing / Service Availability
       |
       v
Full Workflow Capability
```

A failure or limitation at one stage can affect specific workflow actions without necessarily invalidating the entire deployment.


# 34. Testing the Trigger

After configuring the Logic App and permissions available to the environment, I tested the workflow.

The execution feedback showed:

```text
Successfully ran the trigger
```

This was an important validation milestone.

It demonstrated that the Microsoft Sentinel trigger itself could execute successfully.

I captured this result as implementation evidence.


# 35. Interpreting the Successful Trigger Correctly

The successful trigger should be interpreted precisely.

It demonstrates that:

```text
The Logic App was deployed

The workflow existed

The Sentinel connection existed

The trigger could execute successfully
```

It does not automatically prove that:

```text
Every downstream MDTI enrichment action completed

Every external permission was available

A real malicious incident was triaged end-to-end
```

This distinction is maintained throughout the documentation.


# 36. SOAR Validation Model

I treated SOAR validation as several separate layers.

```text
Layer 1
Playbook Deployment
       |
       v
Successful

Layer 2
Sentinel Connection
       |
       v
Confirmed

Layer 3
Managed Identity
       |
       v
Enabled

Layer 4
Azure RBAC
       |
       v
Sentinel Contributor Assigned

Layer 5
Sentinel Trigger
       |
       v
Successfully Executed

Layer 6
Full MDTI Enrichment
       |
       v
Subject to Additional Permission /
Environment Limitations
```

This provides a more accurate representation of the implementation than simply stating that everything was fully operational.


# 37. Native Automation vs Logic App Playbook

The project ultimately implemented two different automation models.

| Capability | Native Sentinel Automation | MDTI Logic App |
|---|---|---|
| Incident trigger | Yes | Yes |
| Add tag | Yes | Possible |
| Add investigation task | Yes | Possible |
| Change severity | Available | Possible |
| Change status | Available | Possible |
| Assign owner | Available | Possible |
| Entity extraction | Limited | Yes |
| Host processing | No | Yes |
| IP processing | No | Yes |
| Threat-intelligence enrichment | Limited | Designed for it |
| Conditional workflow | Basic | Advanced |
| Looping | Limited | Yes |
| External integrations | Limited | Yes |
| Managed identity | Not required for simple actions | Used |
| Azure RBAC | Not required for simple actions | Required for relevant operations |

The two layers complement one another.


# 38. Final SOAR Workflow

The combined automation architecture can be represented as:

```text
                         Sentinel Incident
                                |
                 +--------------+--------------+
                 |                             |
                 v                             v
      Contoso Sentinel                MDTI-Automated-Triage
       Incident Triage                    Logic App
                 |                             |
          +------+-------+                     v
          |              |              Sentinel Trigger
          v              v                     |
   Add Triage Tag    Add SOC Task               v
          |              |              Extract Entities
          +------+-------+                     |
                 |                       +-----+-----+
                 |                       |           |
                 |                       v           v
                 |                     Hosts        IPs
                 |                       |           |
                 |                       v           v
                 |                   Processing   Processing
                 |                       |           |
                 |                       +-----+-----+
                 |                             |
                 |                             v
                 |                      Threat Intelligence
                 |                             |
                 +--------------+--------------+
                                |
                                v
                         Analyst Investigation
                                |
                                v
                          Security Decision
```


# 39. Automation and Human Oversight

The automation architecture was intentionally designed around:

```text
Human-in-the-Loop Security Operations
```

Automation performs repeatable tasks.

Analysts retain responsibility for security decisions.

```text
Automation:
Prepare
Enrich
Organize
Assist

Analyst:
Interpret
Validate
Escalate
Contain
Close
```

This distinction is particularly important when automation interacts with security incidents.


# 40. Why Full Automatic Containment Was Not Implemented

The project did not automatically perform actions such as:

```text
Disable user accounts

Block IP addresses

Delete resources

Revoke sessions

Isolate endpoints

Close incidents
```

These actions can have significant operational impact.

Implementing them safely requires:

```text
High-confidence detections

Validated automation

Exception handling

Rollback planning

Appropriate approvals

Production testing
```

The current environment focused on automated triage rather than autonomous containment.


# 41. Automation Failure Handling

A production SOAR workflow should assume that automation can fail.

Possible failure points include:

```text
Missing Permissions

Expired Connections

API Failures

Rate Limits

Missing Entities

Unexpected Data Formats

Licensing Restrictions

Service Availability

RBAC Misconfiguration
```

A mature workflow should therefore include:

```text
Error Handling

Retry Logic

Logging

Failure Notifications

Manual Fallback
```

These would be important enhancements for a production deployment.


# 42. SOAR Security Considerations

Security automation itself must be secured.

Important controls include:

### Least Privilege

Only grant the workflow permissions it actually requires.

### Managed Identity

Prefer managed identity over embedded Azure credentials where supported.

### RBAC Scope

Avoid assigning unnecessarily broad roles at subscription scope.

### Connection Security

Review Logic App API connections and their authorization.

### Workflow Change Control

Changes to automated response logic should be reviewed before production deployment.

### Auditability

Workflow executions should be logged and reviewable.

### Human Approval

High-impact response actions may require analyst approval.


# 43. Validation Results

The SOAR implementation produced the following validation state:

| Component | Result |
|---|---|
| Initial Sentinel automation rules | 0 |
| Native automation rule | Created |
| Rule name | `Contoso Sentinel Incident Triage` |
| Trigger | When incident is created |
| Automated-Triage tag | Configured |
| Initial SOC investigation task | Configured |
| Initial active playbooks | 0 |
| MDTI templates reviewed | 7 visible |
| `MDTI-Automated-Triage` | Selected |
| Logic App deployment | Successful |
| Microsoft Sentinel connection | Confirmed |
| Sentinel incident trigger | Present |
| Host processing | Present |
| IP processing | Present |
| System-assigned identity | Enabled |
| Initial RBAC assignments | 0 |
| Sentinel RBAC role | Added successfully |
| Role | Microsoft Sentinel Contributor |
| Entra role eligibility | Limited by licensing |
| MDTI permission requirement | Limitation encountered |
| Sentinel trigger test | Successful |
| Full real-world malicious incident triage | Not claimed

# 44. Evidence Captured

The SOAR implementation can be supported with screenshots showing:

```text
Sentinel Automation interface

Contoso Sentinel Incident Triage rule

Incident-created trigger

Automation actions

Playbooks interface

MDTI playbook templates

MDTI-Automated-Triage deployment

Logic App designer

Microsoft Sentinel connection

System-assigned managed identity

Initial role assignment state

Microsoft Sentinel Contributor assignment

Workflow trigger execution

Successful trigger feedback
```

Recommended repository structure:

```text
images/
└── soar/
    ├── sentinel-automation-overview.png
    ├── contoso-incident-triage-rule.png
    ├── incident-created-trigger.png
    ├── automated-triage-actions.png
    ├── sentinel-playbooks-overview.png
    ├── mdti-playbook-templates.png
    ├── mdti-automated-triage-deployment.png
    ├── mdti-logic-app-workflow.png
    ├── sentinel-connection.png
    ├── system-assigned-identity.png
    ├── initial-role-assignments.png
    ├── sentinel-contributor-role.png
    └── successful-trigger-run.png
```

Only evidence actually captured during implementation should be added.


# 45. Limitations

The SOAR implementation had several limitations.

## No Active Incident During Primary Validation

There was no real active Sentinel incident available for a complete end-to-end incident triage demonstration.

## MDTI Permission Limitation

Some Defender Threat Intelligence functionality depended on permissions that could not be fully configured within the available environment.

## Entra Licensing Limitation

Role eligibility functionality required Microsoft Entra ID P2 or Microsoft Entra ID Governance.

## No Automated Containment

The workflow focused on triage rather than high-impact remediation.

## No External Ticketing Integration

The playbook was not connected to an external ITSM or case-management platform.

## No Notification Integration

Automated email, Teams, or other notification workflows were outside the implemented scope.

These limitations are explicitly documented so that the project does not overstate its capabilities.


# 46. Future Improvements

The SOAR architecture could be expanded in several ways.

### Automated Incident Enrichment

Enrich:

```text
IP addresses

Domains

URLs

Hosts

Accounts
```

using approved intelligence sources.

### Severity-Based Automation

Create different workflows for:

```text
High

Medium

Low
```

severity incidents.

### Conditional Playbook Execution

Only execute expensive enrichment workflows when relevant entities exist.

### SOC Notifications

Send notifications for high-priority incidents.

### Ticketing Integration

Automatically create cases in an ITSM or SOC case-management platform.

### Approval-Based Containment

Introduce analyst approval before actions such as:

```text
Account disablement

Session revocation

Endpoint isolation

IP blocking
```

### Automation Metrics

Track:

```text
Playbook executions

Success rate

Failure rate

Average execution time

Incidents enriched

Manual actions avoided
```

# 47. Example Future Response Architecture

A more mature implementation could evolve into:

```text
Incident Created
       |
       v
Initial Triage
       |
       v
Extract Entities
       |
       v
Threat Intelligence Enrichment
       |
       v
Risk Evaluation
       |
       +-----------------------------+
       |                             |
       v                             v
Low Confidence                 High Confidence
       |                             |
       v                             v
Analyst Review              Request Approval
                                     |
                                     v
                              Approved Response
                                     |
                         +-----------+-----------+
                         |                       |
                         v                       v
                  Identity Action         Endpoint Action
                         |                       |
                         +-----------+-----------+
                                     |
                                     v
                                Update Incident
```

Such a workflow would require considerably more validation before production use.


# 48. Skills Demonstrated

The SOAR phase demonstrated practical experience with:

```text
Microsoft Sentinel Automation Rules

Microsoft Sentinel Playbooks

Azure Logic Apps

Microsoft Defender Threat Intelligence

Incident Triggers

Entity Processing

Managed Identities

Azure RBAC

Microsoft Sentinel Contributor

Security Workflow Troubleshooting

Human-in-the-Loop Automation

SOAR Architecture

Security Permission Analysis
```

It also demonstrated an understanding that automation is not only about building a workflow.

A secure SOAR implementation also requires:

```text
Identity

Authorization

Permissions

Connections

Telemetry

Error Handling

Analyst Oversight
```


# 49. Key Lessons

Several important lessons came from the SOAR implementation.

## Deployment Is Only the First Step

A successfully deployed Logic App still requires connections, identity, authorization, and validation.

## Authentication and Authorization Are Different

Managed identity establishes identity, while RBAC determines what the workflow can do.

## Native Automation and Logic Apps Serve Different Needs

Simple incident actions can remain within Sentinel, while complex orchestration benefits from Logic Apps.

## Least Privilege Matters for Automation

A security playbook should not receive excessive permissions simply to make configuration easier.

## Licensing Can Affect Security Automation

Some identity governance and threat-intelligence capabilities depend on licensing or additional permissions.

## A Successful Trigger Is Not the Same as Full End-to-End Enrichment

Validation should state exactly what was tested.

## Automation Should Not Outrun Detection Confidence

High-impact response actions require stronger validation than basic triage actions.


# 50. Final SOAR Outcome

The final SOAR implementation can be summarized as:

```text
Native Automation:
Contoso Sentinel Incident Triage

Trigger:
When incident is created

Native Actions:
Add Automated-Triage tag
Add Perform Initial SOC Investigation task


Advanced Playbook:
MDTI-Automated-Triage

Platform:
Azure Logic Apps

Sentinel Connection:
Confirmed

System Assigned Managed Identity:
Enabled

Initial Role Assignments:
0

RBAC Remediation:
Microsoft Sentinel Contributor assigned successfully

Sentinel Trigger:
Successfully executed

MDTI Permission:
Additional limitation encountered

Automatic Containment:
Not implemented

Human Analyst:
Retained in decision loop
```

The final security automation model was therefore:

```text
                 DETECT
                    |
                    v
              Sentinel Alert
                    |
                    v
                 Incident
                    |
          +---------+---------+
          |                   |
          v                   v
Native Automation       Logic App SOAR
          |                   |
          v                   v
Tag + Task          Entity Processing
          |                   |
          |                   v
          |            Threat Intelligence
          |                   |
          +---------+---------+
                    |
                    v
                  TRIAGE
                    |
                    v
              Analyst Review
                    |
                    v
                 DECIDE
                    |
                    v
                 RESPOND
```

This completed the SOAR layer of the Microsoft Sentinel implementation and connected security monitoring, detection, investigation, automation, identity, and access control into a single security operations workflow.


## Related Documentation

- [Architecture](Architecture.md)
- [Deployment Guide](DeploymentGuide.md)
- [Data Connectors](DataConnectors.md)
- [KQL Queries](KQLQueries.md)
- [Analytics Rules](AnalyticsRules.md)
- [Incident Management](IncidentManagement.md)
- [Threat Hunting](ThreatHunting.md)
- [MITRE ATT&CK Mapping](MITRE-ATTACK-Mapping.md)
- [Workbooks](Workbooks.md)
- [Validation Report](ValidationReport.md)
- [Troubleshooting](Troubleshooting.md)
