# Lessons Learned

## 1. Overview

Building this Microsoft Sentinel SIEM/SOAR environment provided lessons that went beyond configuring individual Azure services.

The project required working across:

```text
Telemetry Collection
KQL Investigation
Detection Engineering
Incident Management
Threat Hunting
MITRE ATT&CK
Security Visualization
SOAR
Managed Identity
Azure RBAC
Troubleshooting
```

The most important lesson was that a security operations platform is not defined by the number of features that have been enabled.

Its effectiveness depends on how well those capabilities work together.

The project reinforced a security operations model based on:

```text
Visibility
    |
    v
Detection
    |
    v
Investigation
    |
    v
Decision
    |
    v
Response
    |
    v
Continuous Improvement
```


# 2. Telemetry Comes Before Detection

One of the strongest lessons from the project was:

> Detection engineering begins with telemetry, not analytics rules.

Before creating hunting queries, enabling analytics rules, or building dashboards, I first had to determine whether useful security data actually existed.

The initial workspace search returned no useful data within:

```text
Last 24 hours
```

After expanding the investigation window to:

```text
Last 7 days
```

I identified:

```text
AzureMetrics       24
Usage               7
AzureDiagnostics    4
```

This demonstrated a fundamental SOC principle:

```text
No Telemetry
     |
     v
No Visibility
     |
     v
No Reliable Detection
     |
     v
No Effective Investigation
```

Before asking what should be detected, an analyst must understand what can actually be observed.


# 3. Understand the Data Before Writing Detection Logic

After finding `AzureDiagnostics`, I inspected the raw records before developing more targeted queries.

This revealed values including:

```text
ResourceProvider:
MICROSOFT.KEYVAULT

Resource:
KV-CONTOSO-PROD-001

OperationName:
VaultGet

ResultType:
Success
```

That inspection directly influenced the KQL logic that followed.

This reinforced the workflow:

```text
Inspect Raw Telemetry
        |
        v
Understand Schema
        |
        v
Understand Field Values
        |
        v
Develop Query
        |
        v
Validate Results
```

Writing detection logic without understanding the underlying data can easily produce queries that look correct but do not actually detect anything.


# 4. Time Range Is Part of the Investigation

The difference between:

```text
Last 24 hours
```

and:

```text
Last 7 days
```

completely changed the initial investigation.

The first timeframe suggested that useful telemetry might not be available.

The second revealed the records needed for the rest of the project.

This demonstrated that time range is not simply a UI preference.

It is part of the investigative logic.

A SOC analyst should consider:

```text
When did the activity occur?

How frequently does the resource generate events?

How long is telemetry retained?

Could the selected timeframe exclude the evidence?
```

A correct query can still produce an incomplete investigation when the wrong time window is selected.


# 5. Zero Results Can Be Evidence

The failed Key Vault operations query returned:

```text
0 results
```

Initially, it would have been easy to view that as an unsuccessful query.

However, the broader activity query had already confirmed that Key Vault telemetry existed.

The zero-result query therefore provided information:

> No Key Vault events matching the unsuccessful-operation condition were identified within the available telemetry and investigation period.

This reinforced an important threat-hunting principle:

```text
No Matches
     !=
Query Failure
```

Zero results can help invalidate a hypothesis.


# 6. Threat Hunting Is Not About Finding a Threat Every Time

The Hunt was created as:

```text
Azure Key Vault Suspicious Activity Investigation
```

The hypothesis began as:

```text
Unknown
```

The investigation identified:

```text
4 Key Vault events
0 unsuccessful operations matching the query
```

The evidence did not support the suspicious-activity hypothesis.

The final hypothesis was therefore:

```text
Invalidated
```

This was an important lesson.

Threat hunting should not operate like:

```text
Start Investigation
      |
      v
Must Find Something Malicious
```

Instead:

```text
Develop Hypothesis
      |
      v
Collect Evidence
      |
      v
Evaluate Evidence
      |
      +-------------------+
      |                   |
      v                   v
Supported            Not Supported
      |                   |
      v                   v
Validate             Invalidate
```

Both outcomes are legitimate.


# 7. Do Not Force an Incident

The Hunt did not produce evidence that justified creating a Sentinel incident.

I therefore did not manufacture an incident merely to make the project appear more complete.

This reinforced the distinction between:

```text
Interesting Activity

Suspicious Activity

Confirmed Security Finding
```

They are not automatically the same thing.

A SOC should escalate based on evidence, not because an analyst expects every investigation to become an incident.


# 8. Detection Coverage Is More Important Than Rule Count

The Analytics interface initially contained no active rules.

Available templates included:

```text
Anomalous RDP Login Detection

Anomalous SSH Login Detection

Microsoft Defender Threat Intelligence Analytics
```

It would have been easy to enable all available rules and report a larger detection-rule count.

However, the environment did not contain the workloads or telemetry required to meaningfully support RDP and SSH detections.

I therefore did not enable those templates.

This reinforced:

```text
More Rules
    !=
Better Detection
```

A better model is:

```text
Relevant Workload
       +
Required Telemetry
       +
Validated Detection Logic
       =
Meaningful Detection Coverage
```

# 9. MITRE ATT&CK Is a Coverage Framework, Not Proof of Compromise

The enabled analytics rule displayed mappings to:

```text
Persistence

Lateral Movement
```

The Sentinel ATT&CK interface also allowed technique-level investigation.

One of the technique areas reviewed was:

```text
Account Manipulation
```

The important lesson was that these mappings represent:

```text
Detection Context
```

not:

```text
Confirmed Attacker Activity
```

A rule being mapped to Persistence does not mean persistence occurred.

Similarly, a technique appearing in the ATT&CK coverage interface does not mean an attacker used that technique against the environment.


# 10. ATT&CK Becomes More Useful When Connected to Telemetry

MITRE ATT&CK is most useful when connected to actual detection engineering.

A practical model is:

```text
ATT&CK Technique
       |
       v
What Telemetry Would Reveal It?
       |
       v
Do We Collect That Telemetry?
       |
       v
Can We Detect the Behavior?
       |
       v
Can We Validate the Detection?
```

This makes ATT&CK a practical engineering framework rather than simply a matrix displayed in the SOC.


# 11. Dashboards Are Only as Good as Their Data

The Azure Key Vault Security workbook initially failed to populate correctly.

The issue was not the workbook itself.

The required parameters had not been configured.

After setting:

```text
Workspace:
law-contoso-prod-001

KeyVault:
kv-contoso-prod-001
```

the visualizations populated successfully.

This reinforced:

```text
Dashboard
   |
   v
Queries
   |
   v
Telemetry
```

A visually impressive workbook cannot compensate for missing, incorrect, or improperly scoped telemetry.


# 12. Templates Are Starting Points, Not Finished Implementations

This applied to several parts of the project:

```text
Analytics Templates

Workbook Templates

Playbook Templates
```

A template may provide useful logic, but it still requires:

```text
Environment Configuration

Data Sources

Permissions

Parameters

Validation
```

The Key Vault workbook demonstrated this directly.

The MDTI playbook demonstrated it even more strongly.


# 13. Deployment Does Not Mean Operational

The `MDTI-Automated-Triage` Logic App deployed successfully.

However, successful deployment was only the beginning.

The workflow still required investigation of:

```text
Connections

Managed Identity

Azure RBAC

Additional Permissions

Trigger Execution
```

This produced one of the most important engineering lessons from the project:

```text
Deployed
   !=
Configured
   !=
Authorized
   !=
Tested
   !=
Production Ready
```

Each stage requires separate validation.


# 14. Authentication and Authorization Are Different

The Logic App had:

```text
System Assigned Identity:
On
```

but initially showed:

```text
0 role assignments
```

The identity existed, but that did not automatically mean it had permission to perform Sentinel operations.

I later assigned:

```text
Microsoft Sentinel Contributor
```

successfully.

The distinction became:

```text
Managed Identity
      |
      v
Authentication
      |
      v
Who is this workload?
```

versus:

```text
Azure RBAC
      |
      v
Authorization
      |
      v
What may this workload do?
```

Understanding this difference is essential when securing cloud automation.

# 15. Least Privilege Still Applies to Automation

It can be tempting to solve permission problems by granting very broad Azure access.

That would make troubleshooting easier in the short term but weaken the security architecture.

Instead, the playbook identity was given a Sentinel-specific role appropriate to the workflow.

The lesson was:

```text
Automation
     |
     v
Is Still an Identity
     |
     v
Must Follow Least Privilege
```

SOAR workflows can perform powerful actions.

Their permissions therefore require the same scrutiny as human administrator accounts.

# 16. RBAC Does Not Solve Every Permission Problem

Assigning:

```text
Microsoft Sentinel Contributor
```

addressed the Sentinel RBAC requirement.

However, the MDTI workflow also encountered an additional requirement related to:

```text
ThreatIntelligence.Read.All
```

This demonstrated that cloud security automation may involve several authorization systems simultaneously.

For example:

```text
Azure RBAC
      +
Managed Identity
      +
API Connections
      +
Service-Specific Permissions
      +
Licensing
```

Successfully configuring one layer does not automatically configure the others.

# 17. Licensing Is Part of Security Architecture

During role configuration, the environment indicated that role eligibility required:

```text
Microsoft Entra ID P2

or

Microsoft Entra ID Governance
```

This demonstrated that security architecture is influenced not only by technical configuration but also by licensing.

A theoretically valid design may depend on capabilities unavailable in a particular tenant.

The correct response is to determine:

```text
What feature is affected?

Is the limitation critical?

Is an alternative available?

Can unaffected components still be implemented?

How should the limitation be documented?
```

In this case, standard Azure RBAC assignment remained available.


# 18. SOAR Should Assist Analysts, Not Blindly Replace Them

The native Sentinel automation rule was intentionally conservative.

It was configured to:

```text
Add Automated-Triage tag

Add Perform Initial SOC Investigation task
```

It did not automatically:

```text
Close incidents

Change severity

Block infrastructure

Disable accounts

Declare activity malicious
```

This reinforced the value of:

```text
Human-in-the-Loop Automation
```

A useful model is:

```text
Automation
    |
    v
Prepare
Enrich
Organize
Prioritize
    |
    v
Analyst
    |
    v
Interpret
Validate
Decide
Respond
```

# 19. Automation Confidence Should Match Detection Confidence

The more disruptive an automated action is, the more confidence should be required before execution.

For example:

```text
Add Tag
```

has relatively low operational impact.

But:

```text
Disable Account
```

or:

```text
Isolate Endpoint
```

can have significant business consequences.

A mature SOAR architecture should therefore consider:

```text
Detection Confidence
        |
        v
Response Impact
        |
        v
Approval Requirement
```

High-impact actions may require analyst approval.


# 20. Test Automation in Layers

The Logic App trigger successfully returned:

```text
Successfully ran the trigger
```

That was useful evidence.

However, it did not prove that every downstream MDTI action completed successfully.

This reinforced a layered testing approach:

```text
1. Deployment

2. Connection

3. Identity

4. RBAC

5. Trigger

6. Individual Actions

7. End-to-End Workflow

8. Failure Handling
```

Testing each layer separately makes troubleshooting more precise.


# 21. Successful Trigger Does Not Mean Successful Workflow

This distinction deserves specific emphasis.

The successful trigger confirmed:

```text
Sentinel Trigger Operational
```

It did not automatically confirm:

```text
Full MDTI Enrichment Operational
```

This lesson applies broadly to automation.

For example:

```text
HTTP Request Received
        !=
Entire Application Successful
```

and:

```text
Logic App Triggered
        !=
Every Action Completed
```

Run history and downstream actions must be inspected separately.


# 22. Small Datasets Require Careful Language

The environment contained:

```text
4 Key Vault events
```

This was enough to demonstrate:

```text
Telemetry ingestion

KQL investigation

Hunting

Workbook visualization
```

but not enough to establish a strong production behavioral baseline.

Therefore, I avoided claims such as:

```text
Established normal enterprise Key Vault behavior
```

and instead described the results as:

```text
Available telemetry

Observed activity

Basic activity summary
```

The lesson was simple:

> Analytical confidence should match the amount and quality of evidence available.


# 23. Security Documentation Is Part of the Engineering

The project did not end when the Azure configuration worked.

Each component also needed to be documented:

```text
What was implemented?

Why was it implemented?

How was it validated?

What evidence exists?

What failed?

What was fixed?

What remains limited?
```

This produced documentation covering:

```text
Architecture

Deployment

Data Connectors

KQL

Analytics

Incident Management

Threat Hunting

MITRE ATT&CK

Workbooks

SOAR

Validation

Troubleshooting
```

The process reinforced that good security engineering includes making systems understandable to other people.


# 24. Evidence Is More Valuable Than Claims

Throughout the project, screenshots were captured for major implementation stages.

Examples included:

```text
KQL query results

Threat Hunt

Analytics rule

MITRE ATT&CK coverage

Workbook

Automation rule

Logic App

Managed identity

RBAC

Successful trigger execution
```

This supports a useful portfolio principle:

```text
Do Not Just Say:
"I configured Sentinel."

Show:
What was configured
How it was configured
What data it received
How it was tested
What the results were
```

Evidence makes the project technically defensible.


# 25. Failed or Limited Features Should Not Be Hidden

Some project capabilities were constrained.

Examples included:

```text
Entra role eligibility

Additional MDTI permissions

Full live incident testing

Large-scale telemetry

Full MDTI enrichment
```

These were documented instead of removed from the project story.

This matters because real security engineering includes limitations.

A perfectly smooth project with no troubleshooting often communicates less technical depth than one that clearly explains:

```text
Problem

Investigation

Root Cause

Resolution

Remaining Limitation
```

# 26. Troubleshooting Should Be Hypothesis Driven

The troubleshooting process often resembled threat hunting itself.

For example:

```text
Problem:
Workbook does not populate

Hypothesis:
Telemetry is missing

Test:
Check KQL

Result:
Telemetry exists

New Hypothesis:
Workbook parameters are incorrect

Test:
Configure workspace and Key Vault

Result:
Workbook populates
```

This is more effective than randomly changing settings.

A useful troubleshooting pattern is:

```text
Observe
   |
   v
Form Hypothesis
   |
   v
Test
   |
   v
Evaluate
   |
   v
Refine
```

# 27. Security Engineering Is About Relationships Between Components

The project initially looked like a collection of Azure services:

```text
Sentinel

Log Analytics

Key Vault

Logic Apps

MITRE ATT&CK

Workbooks
```

But the real value appeared when they were connected.

```text
Key Vault
   |
   v
Telemetry
   |
   v
Log Analytics
   |
   v
Sentinel
   |
   +---------------------+
   |                     |
   v                     v
Detection             Hunting
   |                     |
   +----------+----------+
              |
              v
         Investigation
              |
              v
          Automation
              |
              v
        Analyst Response
```

Security architecture is therefore less about individual products and more about how information and decisions flow between them.

# 28. SIEM Is Not Just Log Storage

The project reinforced that a SIEM should not be viewed simply as:

```text
A place where logs are stored.
```

The operational value comes from turning telemetry into:

```text
Searchable Evidence

Detection Logic

Hunting Capability

Incident Context

Security Visualization

Response Workflows
```

The complete process is:

```text
Raw Data
   |
   v
Security Context
   |
   v
Security Decision
```

# 29. SOAR Is Not Just "Automatic Response"

Similarly, SOAR is not simply:

```text
Automatically block something.
```

SOAR can also automate:

```text
Enrichment

Tagging

Task Creation

Entity Extraction

Notifications

Case Preparation

Evidence Collection

Workflow Coordination
```

This makes SOAR valuable even when high-impact containment remains under human control.


# 30. Detection Engineering Is an Iterative Process

A detection should not be considered permanently finished after deployment.

The longer-term process should be:

```text
Develop
   |
   v
Deploy
   |
   v
Observe
   |
   v
Validate
   |
   v
Tune
   |
   v
Measure
   |
   v
Improve
```

Future production data would allow the analytics developed in this project to be tuned more accurately.


# 31. Hunting Can Become Detection

The custom Key Vault hunting queries demonstrated another useful SOC progression.

A query may begin as:

```text
Analyst Investigation
```

If it repeatedly identifies meaningful behavior, it may eventually evolve into:

```text
Detection Logic
```

The progression is:

```text
Question
   |
   v
Hunting Query
   |
   v
Useful Finding
   |
   v
Repeated Validation
   |
   v
Detection Candidate
   |
   v
Analytics Rule
```

However, the small dataset in this project was not sufficient to justify converting every hunting query into a production detection.


# 32. Detection Content Should Match the Environment

The RDP and SSH templates provided a clear example.

A detection may be technically valid but operationally irrelevant if the environment does not contain:

```text
The workload

The telemetry

The attack surface
```

This reinforced the need for environment-specific detection engineering.

The question should not be:

```text
What rules can I enable?
```

It should be:

```text
What threats matter to this environment, and what telemetry can detect them?
```

# 33. Security Visualization Should Support Decisions

Workbooks should not exist simply because dashboards look impressive.

A useful workbook should help answer questions.

For the Key Vault environment, examples include:

```text
Is the resource generating telemetry?

What activity is occurring?

How does activity change over time?

Are operational patterns unusual?

Should the analyst investigate further?
```

Visualization should support investigation rather than replace it.

# 34. Portfolio Projects Should Be Technically Defensible

One of the broader lessons from the project was the importance of being able to explain every major claim.

For example, instead of saying:

```text
Built a fully automated enterprise SOC.
```

the evidence supports more precise statements such as:

```text
Implemented Microsoft Sentinel SIEM monitoring.

Validated Azure Key Vault telemetry ingestion.

Developed and executed custom KQL hunting queries.

Configured a structured Sentinel Hunt.

Enabled Microsoft Defender Threat Intelligence analytics.

Reviewed MITRE ATT&CK detection mappings.

Configured and validated a Key Vault security workbook.

Built native Sentinel incident triage automation.

Deployed an MDTI Logic App playbook.

Configured managed identity and Sentinel RBAC.

Successfully tested the Sentinel playbook trigger.
```

Precise claims are stronger because they can be demonstrated.

# 35. What I Would Do Differently Next Time

If rebuilding the environment with broader resources and licensing, I would introduce validation earlier.

Instead of:

```text
Build Everything
      |
      v
Test Everything at the End
```

I would use:

```text
Build Component
      |
      v
Validate Component
      |
      v
Capture Evidence
      |
      v
Document
      |
      v
Move to Next Component
```

This would reduce troubleshooting complexity and improve evidence collection.

# 36. What I Would Add to a Larger Environment

A larger implementation would benefit from additional telemetry sources.

These could include:

```text
Microsoft Entra ID

Microsoft Defender for Endpoint

Windows Security Events

Linux Syslog

Azure Activity

Azure Firewall

Network Security Groups

Microsoft 365

Additional Azure Resources
```

This would enable significantly richer correlation and detection engineering.

# 37. What I Would Add to Detection Engineering

With more telemetry, I would expand detection engineering into:

```text
Identity attacks

Credential attacks

Privilege escalation

Suspicious administrative changes

Endpoint compromise

Network anomalies

Cloud persistence

Data-access anomalies

Threat-intelligence matches
```

Each detection would be mapped to relevant ATT&CK techniques and tested against controlled activity where appropriate.

# 38. What I Would Add to Threat Hunting

Future Hunts could investigate:

```text
Unusual authentication

Rare administrative activity

Suspicious source IP addresses

Abnormal resource access

High-frequency Key Vault activity

Unexpected secret access

Identity privilege changes

Cross-resource activity

Threat-intelligence indicators
```

The Hunt methodology developed during this project could be reused for each scenario.

# 39. What I Would Add to SOAR

The SOAR architecture could evolve into:

```text
Incident
   |
   v
Automated Enrichment
   |
   v
Risk Classification
   |
   v
Analyst Approval
   |
   v
Containment
   |
   v
Incident Update
   |
   v
Notification
   |
   v
Case Closure
```

Potential integrations could include:

```text
Identity response

Endpoint isolation

IP blocking

SOC notifications

ITSM ticketing

Threat intelligence

Evidence collection
```

High-impact actions would require stronger validation and appropriate approval controls.

# 40. What I Would Add to Monitoring

A mature SOC implementation would also monitor the monitoring system itself.

This could include:

```text
Connector health

Ingestion failures

Analytics rule health

Playbook failures

Logic App execution errors

Automation success rates

Telemetry gaps

Data-volume changes
```

A SIEM/SOAR platform needs operational monitoring just like the systems it protects.

# 41. What I Would Add to Metrics

Future SOC metrics could include:

```text
Mean Time to Detect

Mean Time to Acknowledge

Mean Time to Investigate

Mean Time to Respond

Mean Time to Resolve

False Positive Rate

Automation Success Rate

Detection Coverage

Incident Volume by Severity

ATT&CK Coverage
```

These would help measure security operations effectiveness rather than only technical configuration.


# 42. Most Important Technical Lessons

The most important technical lessons from the project were:

```text
1. Validate telemetry before building detections.

2. Inspect raw data before writing complex KQL.

3. Time range is part of investigation logic.

4. Zero results can still be valid evidence.

5. Threat hunting does not need to find a threat every time.

6. Detection coverage matters more than rule count.

7. ATT&CK mapping does not mean an attack occurred.

8. Workbook templates require environment configuration.

9. Deployment does not equal operational readiness.

10. Managed identity does not automatically grant authorization.

11. RBAC and API permissions are separate layers.

12. Licensing can directly affect security architecture.

13. SOAR should match the confidence of the detection.

14. High-impact automation should retain appropriate human oversight.

15. Automation should be validated in layers.

16. Small datasets require careful conclusions.

17. Security documentation should reflect evidence.

18. Limitations should be documented rather than hidden.
```

# 43. Most Important SOC Lesson

The project ultimately reinforced one central idea:

> Security operations is an evidence-driven process.

Every stage depends on evidence.

```text
Telemetry provides evidence.

KQL searches evidence.

Analytics evaluates evidence.

Threat hunting investigates evidence.

ATT&CK contextualizes behavior.

Workbooks visualize evidence.

Incidents organize evidence.

SOAR enriches and acts on evidence.

Analysts interpret evidence.
```

Without reliable evidence, security decisions become assumptions.

# 44. Final Reflection

At the beginning of the implementation, Microsoft Sentinel could easily be viewed as a collection of features:

```text
Logs

Analytics

Incidents

Hunting

Workbooks

Automation
```

By the end of the project, the relationship between those features was much clearer.

```text
                         TELEMETRY
                             |
                             v
                           KQL
                             |
                  +----------+----------+
                  |                     |
                  v                     v
              HUNTING               ANALYTICS
                  |                     |
                  v                     v
             INVESTIGATE            DETECT
                  |                     |
                  +----------+----------+
                             |
                             v
                          INCIDENT
                             |
                  +----------+----------+
                  |                     |
                  v                     v
               ATT&CK               WORKBOOKS
                  |                     |
                  +----------+----------+
                             |
                             v
                           SOAR
                             |
                             v
                       ANALYST DECISION
                             |
                             v
                          RESPONSE
```

The most valuable outcome of the project was therefore not simply deploying Microsoft Sentinel.

It was learning how telemetry, detection, hunting, investigation, visualization, identity, authorization, automation, and human decision-making fit together within a modern cloud SOC.

That understanding provides the foundation for expanding the environment into more advanced detection engineering, incident response, threat hunting, and security automation scenarios.

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
- [SOAR Automation](SOAR-Automation.md)
- [Validation Report](ValidationReport.md)
- [Troubleshooting](Troubleshooting.md)
