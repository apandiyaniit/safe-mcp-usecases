# Cloud ops troubleshooting assistant
> **SAFE-AUCA industry reference guide (draft)**
>
> This use case describes a real-world agentic workflow used by SRE, cloud operations, DevOps, and incident-response teams to investigate production issues by querying telemetry, correlating recent changes, forming and testing hypotheses, and proposing bounded remediations.
>
> It focuses on:
> - how the workflow works in practice (tools, data, trust boundaries, autonomy)
> - what can go wrong (defender-friendly kill chain)
> - how it maps to **SAFE-MCP techniques**
> - what controls + tests make it safer
>
> **Defender-friendly only:** do **not** include operational exploit steps, payloads, or step-by-step attack instructions.  
> **No sensitive info:** do not include internal hostnames/endpoints, secrets, customer data, non-public incidents, or proprietary details.

---

## Metadata

| Field | Value |
|---|---|
| **SAFE Use Case ID** | `SAFE-UC-0023` |
| **Status** | `draft` |
| **NAICS 2022** | `51` (Information), `518210` (Computing Infrastructure Providers, Data Processing, Web Hosting, and Related Services), `541513` (Computer Facilities Management Services) |
| **Workflow family** | `Cloud operations, SRE, observability, and incident response` |
| **Last updated** | `2026-03-18` |

### Evidence (public links)

- [SAFE-AUCA use case template](https://github.com/safe-agentic-framework/safe-agentic-use-cases/blob/main/templates/use-case-template.md)
- [SAFE-UC-0023 issue / write-up plan](https://github.com/safe-agentic-framework/safe-agentic-use-cases/issues/21)
- [SAFE-UC-0023 seed page](https://github.com/safe-agentic-framework/safe-agentic-use-cases/blob/main/use-cases/SAFE-UC-0023/README.md)
- [Datadog Bits AI SRE overview](https://docs.datadoghq.com/bits_ai/bits_ai_sre/)
- [Datadog Bits AI SRE investigations](https://docs.datadoghq.com/bits_ai/bits_ai_sre/investigate_issues/)
- [Datadog Bits AI SRE chat context](https://docs.datadoghq.com/bits_ai/bits_ai_sre/chat_bits_ai_sre/)
- [Datadog Bits AI SRE permissions](https://docs.datadoghq.com/bits_ai/bits_ai_sre/configure/)
- [Azure Copilot observability agent overview](https://learn.microsoft.com/en-us/azure/azure-monitor/aiops/observability-agent-overview)
- [Azure Copilot observability agent best practices](https://learn.microsoft.com/en-us/azure/azure-monitor/aiops/observability-agent-best-practices)
- [Azure Copilot observability agent responsible-use FAQ](https://learn.microsoft.com/en-us/azure/azure-monitor/aiops/observability-agent-responsible-use)
- [Gemini Cloud Assist investigations](https://cloud.google.com/cloud-assist/investigations)
- [Gemini Cloud Assist IAM requirements](https://cloud.google.com/cloud-assist/iam-requirements)
- [Gemini Cloud Assist audit logging](https://cloud.google.com/cloud-assist/audit-logging)
- [GKE: Accelerate diagnosis with Gemini Cloud Assist](https://cloud.google.com/kubernetes-engine/docs/troubleshooting/introduction-gemini)
- [Amazon Q troubleshooting with Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/troubleshooting-with-Q.html)
- [Amazon Q Developer in chat applications: CLI commands](https://docs.aws.amazon.com/chatbot/latest/adminguide/chatbot-cli-commands.html)
- [AWS Systems Manager Change Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/change-manager.html)
- [AWS Systems Manager approvals (`aws:approve`)](https://docs.aws.amazon.com/systems-manager/latest/userguide/running-automations-require-approvals.html)
- [AWS Systems Manager Change Calendar integration](https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-change-calendar-integration.html)
- [OWASP GenAI Top 10: LLM01 Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)
- [OWASP GenAI Top 10 archive](https://genai.owasp.org/llm-top-10/)
- [Cloud Security Alliance AI Controls Matrix](https://cloudsecurityalliance.org/artifacts/ai-controls-matrix)
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)
- [NIST AI 600-1 Generative AI Profile](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf)
- [NIST SP 800-61r3 incident response recommendations](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r3.pdf)
- [Model Context Protocol specification](https://modelcontextprotocol.io/specification/2025-11-25)
- [MCP tools specification](https://modelcontextprotocol.io/specification/2025-11-25/server/tools)
- [MCP authorization guidance](https://modelcontextprotocol.io/docs/tutorials/security/authorization)
- [MCP security best practices](https://modelcontextprotocol.io/docs/tutorials/security/security_best_practices)
- [SAFE-T1102 Prompt Injection (Multiple Vectors)](https://github.com/safe-agentic-framework/safe-mcp/blob/main/techniques/SAFE-T1102/README.md)
- [SAFE-T1001 Tool Poisoning Attack](https://github.com/safe-agentic-framework/safe-mcp/blob/main/techniques/SAFE-T1001/README.md)
- [SAFE-T1104 Over-Privileged Tool Abuse](https://github.com/safe-agentic-framework/safe-mcp/blob/main/techniques/SAFE-T1104/README.md)
- [SAFE-T1302 High-Privilege Tool Abuse](https://github.com/safe-agentic-framework/safe-mcp/blob/main/techniques/SAFE-T1302/README.md)
- [SAFE-T1401 Line Jumping](https://github.com/safe-agentic-framework/safe-mcp/blob/main/techniques/SAFE-T1401/README.md)
- [SAFE-T1404 Response Tampering](https://github.com/safe-agentic-framework/safe-mcp/blob/main/techniques/SAFE-T1404/README.md)
- [SAFE-T2105 Disinformation Output](https://github.com/safe-agentic-framework/safe-mcp/blob/main/techniques/SAFE-T2105/README.md)

---

## 1. Executive summary (what + why)

**What this workflow does**  
A cloud ops troubleshooting assistant receives an alert, incident ticket, chat escalation, or operator request; scopes the affected service/environment/time window; retrieves metrics, logs, traces, configuration and change-history context; tests hypotheses using read-only queries or bounded diagnostics; and returns an evidence-backed root-cause analysis (RCA) summary with recommended next steps. In more mature deployments, the same assistant can draft change requests, trigger preapproved runbooks, or monitor post-change signals.

**Why it matters (business value)**  
This workflow reduces mean-time-to-detect and mean-time-to-repair (MTTD/MTTR), lowers on-call cognitive load, improves consistency of triage, and helps newer responders reason across multi-cloud, microservice, and hybrid environments without manually pivoting across many consoles and dashboards. The value is highest when incidents are noisy, telemetry is fragmented, and the cost of slow diagnosis is large.

**Why it is risky / what can go wrong**  
The assistant operates at the intersection of production telemetry, infrastructure identity, change automation, and human urgency. That creates a high-impact failure surface: attacker-controlled or malformed logs can contaminate model context; stale or incorrect runbooks can distort recommendations; over-privileged tools can turn a diagnosis agent into a production mutation agent; and hidden or misleading responses can cause operators to approve destructive actions under false assumptions. In short, this workflow can amplify either reliability or harm.

---

## 2. Industry context & constraints (reference-guide lens)

### 2.1 Where this shows up

Common in:

- cloud platform teams and internal developer platforms
- SaaS operations and SRE organizations
- managed service providers and NOCs
- enterprise infrastructure teams running multi-cloud or hybrid estates
- regulated environments where incident timelines, approvals, and audit evidence matter

### 2.2 Why this qualifies as a real-world agentic workflow

This is not a hypothetical pattern. Public cloud and observability vendors already expose investigation-style assistants that query telemetry, correlate context, and propose remediations:

| Public example | What it demonstrates | Relevance to this use case |
|---|---|---|
| **Datadog Bits AI SRE** | Autonomous investigation loops, manual or automatic monitor-triggered investigations, telemetry retrieval across metrics/logs/traces/events, and separate read/write RBAC permissions | Strong evidence for observability-native troubleshooting agents |
| **Azure Copilot observability agent** | Alert-initiated investigations, anomaly detection and cross-resource data correlation, with explicit guidance to validate outputs before deployment | Strong evidence for read-heavy RCA and recommendation flows |
| **Gemini Cloud Assist investigations** | Ranked observations and hypotheses over logs/configurations/metrics, same-as-user IAM, and explicit audit-logging / data-location considerations | Strong evidence for permission inheritance and operational governance concerns |
| **Amazon Q + AWS Systems Manager** | A split model where some troubleshooting surfaces are read-only while chatops / automation surfaces can run commands or workflows under guardrails, approvals, and change calendars | Strong evidence for why production action paths require stronger policy and HITL gates |

### 2.3 Industry-specific constraints

- **Availability pressure:** operators often need useful answers in minutes, not hours.
- **Correctness over creativity:** a wrong RCA is worse than an inconclusive result if it drives harmful remediation.
- **Production sensitivity:** diagnostics may touch critical systems, stateful services, IAM roles, customer traffic, or regulated workloads.
- **Change-control obligations:** many organizations require approvals, maintenance windows, rollback plans, and evidence for production changes.
- **Cross-domain identity:** the assistant often spans observability systems, cloud APIs, CMDBs, incident systems, and automation tools, each with different authorization models.
- **Telemetry quality dependency:** poor instrumentation, missing release annotations, or inconsistent service metadata directly reduce investigation accuracy.
- **Data-residency / compliance friction:** logs and investigation artifacts may include regulated or contractually restricted data.

### 2.4 Typical systems in this workflow

- alerting and incident platforms
- metrics backends (for example CloudWatch, Azure Monitor, Google Cloud Monitoring, Datadog, Prometheus)
- log platforms and SIEM/log analytics stores
- tracing / APM systems and service catalogs
- cloud inventory and configuration APIs
- deployment, release, and IaC pipelines
- runbook / knowledge-base systems
- chatops, ticketing, and paging systems
- change-management and automation platforms

### 2.5 “Must-not-fail” outcomes

- no self-inflicted outage or cascading incident from assistant-triggered changes
- no unauthorized writes to production resources
- no leakage of secrets, tokens, credentials, or customer-sensitive data into prompts, summaries, tickets, or support cases
- no invisible or weakly disclosed production actions
- no bypass of change windows, approvals, or resource-lock policies
- no irreversible remediation without rollback evidence

### 2.6 Operational constraints

- minutes-level triage loops under stress
- heterogeneous telemetry quality across services
- mixed human expertise across shifts and escalations
- ephemeral infrastructure and noisy autoscaling behavior
- strict least-privilege requirements
- incident forensics and after-action review expectations

---

## 3. Workflow description & scope

### 3.1 Workflow steps (happy path)

```mermaid
flowchart LR
    A[Alert / incident trigger] --> B[Scope incident<br/>service, env, time, symptom]
    B --> C[Retrieve context<br/>metrics, logs, traces, configs, changes, runbooks]
    C --> D[Hypothesis and evidence loop<br/>read-only by default]
    D --> E[Findings, confidence, unknowns,<br/>and remediation options]
    E --> F{Change needed?}
    F -- No --> G[Update incident notes / RCA draft]
    F -- Yes --> H[Approval + change-control gate]
    H --> I[Bounded remediation / runbook]
    I --> J[Post-change verification,<br/>rollback checks, and closure]
```

1. An alert, incident record, or operator prompt starts the session.
2. The assistant identifies service, environment, region/account/project/subscription, time window, symptom, and severity.
3. The assistant retrieves bounded context: metrics, logs, traces, topology, recent deploys, known issues, resource health, change history, and relevant runbooks.
4. The assistant forms hypotheses and tests them through read-only queries or sandboxed diagnostic scripts.
5. The assistant returns findings with evidence, confidence, and explicit unknowns.
6. The assistant proposes remediation options, including blast-radius notes, prerequisites, rollback path, and post-change checks.
7. If action is required, the workflow moves to change-management or approval gates.
8. Approved remediation executes through a bounded tool or runbook.
9. The assistant verifies post-change health and updates incident records / handoff notes.

### 3.2 In scope / out of scope

- **In scope:** telemetry-driven triage; RCA support; evidence collation; read-only queries; bounded diagnostics; draft incident updates; draft change requests; preapproved low-risk automation under policy.
- **Out of scope:** unrestricted shell on production hosts; broad admin changes; IAM or network policy changes without approval; ad hoc destructive actions; silent autonomous remediation on critical stateful systems; unrestricted data export to external systems.

### 3.3 Assumptions

- Telemetry and service metadata exist and are sufficiently complete to support investigation.
- Production and non-production environments are distinguishable in policy and identity.
- Diagnostic scripts and runbooks are versioned, reviewable, and attributable.
- Audit logging exists for both agent decisions and downstream tool actions.
- Change management, rollback procedures, and service ownership are already defined.

### 3.4 Success criteria

“Good” looks like:

- lower MTTR without increasing incident amplification
- evidence-backed outputs with clear confidence and uncertainty
- no production writes outside policy and approval boundaries
- no secret or sensitive-data leakage in outputs
- repeatable, reviewable tool use with immutable audit evidence
- safe rollback and post-change verification when a remediation runs
- acceptable operator trust because the assistant is useful, bounded, and honest about uncertainty

---

## 4. System & agent architecture

### 4.1 Actors and systems

- **Human roles:** on-call engineer, SRE / platform engineer, incident commander, service owner, security reviewer, change approver.
- **Agent / orchestrator:** troubleshooting assistant embedded in observability UI, chatops, incident tooling, or local MCP/API client.
- **Tools (MCP servers / APIs / connectors):**
  - alerts / incidents connector
  - metrics query connector
  - logs query connector
  - trace / APM connector
  - cloud inventory / resource-configuration connector
  - release / deploy history connector
  - runbook / KB retriever
  - read-only diagnostic executor
  - change-management / automation connector
  - ticket / incident writer
- **Data stores:** telemetry stores, dashboards, notebooks, incident records, CMDB/service catalog, IaC repo metadata, runbook repository, audit logs.
- **Downstream systems affected:** incident records, change requests, automation executions, production services, support escalations, compliance evidence stores.

### 4.2 Trusted vs untrusted inputs

| Input/source | Trusted? | Why | Typical failure/abuse pattern | Mitigation theme |
|---|---|---|---|---|
| Alert title/body | Mixed | may be system-generated, but often includes free text or derived telemetry | malicious phrasing or ambiguous scoping | provenance labels + scope validation |
| Logs / exception messages | Untrusted | workload-controlled and attacker-influenceable | indirect prompt injection, false RCA cues, secret leakage | treat as data only; delimit; redact; scan |
| Trace spans / tags / annotations | Untrusted | app- and pipeline-controlled | hidden instructions, poisoned metadata | provenance + schema validation |
| Metrics and monitor state | Semi-trusted | useful but may be stale, aggregated, or mis-scoped | false causality or wrong blast radius | multi-source corroboration |
| Deployment annotations / release notes | Semi-trusted | internal but can be stale or incomplete | stale guidance or wrong change correlation | timestamp / owner / provenance checks |
| Internal runbooks / KB | Semi-trusted | internal, but can drift from reality | obsolete or unsafe remediation | versioning + ownership + review |
| Tool outputs | Mixed | depends on tool and source | contaminated context or hidden action details | structured schemas + policy checks |
| Ticket comments / chat threads | Untrusted | human-generated and socially influenced | social engineering, context flooding | summarization boundaries + approval discipline |
| Diagnostic script output | Mixed | internal tool but runtime-produced | misleading output, secret echoing | sandbox + output filtering |

### 4.3 Trust boundaries (required)

Key boundaries where data or control crosses trust zones:

- **User / operator ↔ agent:** natural-language requests, approvals, and overrides enter the session.
- **Untrusted workload telemetry ↔ agent context:** logs, traces, span tags, alerts, and exceptions cross from workload space into model context.
- **Agent ↔ observability backends:** the assistant can query large volumes of telemetry and potentially retrieve sensitive incident context.
- **Agent ↔ cloud APIs / resource inventory:** investigation can pivot from telemetry into cloud control-plane metadata.
- **Agent ↔ diagnostic execution environment:** even “read-only” scripts may touch live systems or discover secrets.
- **Agent ↔ change-management / automation plane:** the most important boundary, because the workflow moves from analysis into action.
- **Non-production ↔ production:** policies must not allow analysis permissions in one environment to imply write permissions in another.
- **Internal systems ↔ third-party vendor SaaS:** investigation artifacts, chat content, or support escalations may leave the primary trust domain.

**Trust boundary notes**

- In this use case, logs and telemetry are not “trusted context”; they are evidence sources that may be wrong, incomplete, or adversarial.
- The highest-risk boundary is not retrieval. It is the transition from diagnosis to mutation.

### 4.4 Tool inventory (required)

| Tool / connector | Read / write? | Permissions | Typical inputs | Typical outputs | Failure modes |
|---|---|---|---|---|---|
| Alert / incident reader | read | incident-reader role | service, severity, time window, incident ID | alert context, affected entities | wrong incident scope, stale metadata |
| Metrics backend query | read | scoped telemetry read | service/env labels, time window, metric names | timeseries, anomaly context, monitor states | wrong resource scope, false correlation |
| Logs query | read | scoped logs read | query filters, trace IDs, time window | log excerpts, counts, error clusters | prompt injection in logs, secret leakage |
| Trace / APM query | read | scoped APM read | trace ID, span filters, latency/error conditions | traces, service map, dependencies | poisoned span metadata, overbreadth |
| Cloud inventory / config API | read | cloud read-only role | resource IDs, tags, accounts/projects/subscriptions | resource state, config snapshots, health | cross-account overreach, stale inventory |
| Release / change-history connector | read | deployment history read | service, commit, deploy window | release events, rollout state, annotations | outdated or incomplete history |
| Runbook / KB retriever | read | docs / KB read | symptom, service, component | runbook steps, known issues | stale or unsafe guidance |
| Diagnostic script runner | read (should be) | sandbox token, no prod write by default | approved script, scoped target, timeout | script output, status, evidence artifacts | hidden writes, host access, secret discovery |
| Incident note / ticket writer | write to records | scoped record-writer role | summary, action items, references | incident update, handoff note | persistent misinformation, oversharing |
| Change request creator | write to records | change-request role | remediation plan, rationale, rollback, approvers | change ticket / approval artifact | bypass of manual review if over-automated |
| Automation / runbook executor | write / action | tightly scoped automation role | preapproved document, parameters, target set | execution status, logs, outputs | destructive mutation, wrong target set |
| External support / escalation connector | write / comms | verified identity, explicit consent | summary, evidence package | support case / outbound message | data exfiltration, oversharing |

### 4.5 Governance & authorization matrix

| Action category | Example actions | Allowed mode(s) | Approval required? | Required auth | Required logging / evidence |
|---|---|---|---|---|---|
| Read-only retrieval | query metrics/logs/traces, fetch incident details, retrieve config snapshots | manual / HITL / autonomous | no | user session or read-only service role | query log + scope + timestamps |
| Bounded diagnostics | run approved read-only scripts, health probes, dependency checks | HITL / autonomous (non-prod or preapproved prod only) | sometimes | sandbox token + approved script catalog | script ID, inputs, outputs, execution trace |
| Write to records | incident note, RCA draft, change request draft | HITL / autonomous (records only) | yes for persistent shared records in prod incidents | scoped writer role | before/after diff + author/session ID |
| Low-risk reversible remediation | restart stateless pod, scale within approved bounds, rollback latest owned deployment | HITL; autonomous only for allowlisted cases | yes unless explicitly preauthorized by policy | tightly scoped action role + environment guard | policy decision, approval / exemption, rollback plan, post-checks |
| High-risk action | host shell, root/sudo command, IAM change, network rule change, database failover, stateful resource deletion | manual / HITL only | always | step-up auth + JIT role + approver identity | full audit trail + rationale + target list |
| External communications | create support case, send vendor escalation, page additional responders | HITL initially | yes | verified identity | message archive + shared evidence package |

### 4.6 Sensitive data & policy constraints

- **Data classes:** secrets and tokens in logs, headers, and environment variables; customer identifiers; internal IPs/hostnames; vulnerability details; incident timelines; support-case content; cloud resource metadata.
- **Retention / logging constraints:** prompt and tool logs should preserve forensic value without storing raw secrets; record redacted parameters where possible; retain enough session evidence to reconstruct what the assistant saw, decided, and invoked.
- **Regulatory constraints:** data-residency, privacy, contractual confidentiality, and sector-specific controls may limit what telemetry can be retrieved, stored, summarized, or transmitted to vendor-managed investigation systems.
- **Safety / service-harm constraints:** remediation must avoid turning partial outages into full outages; rate-limits, concurrency controls, target caps, and rollback readiness matter as much as model quality.

---

## 5. Operating modes & agentic flow variants

### 5.1 Manual baseline (no agent)

Humans pivot across dashboards, log explorers, APM, cloud consoles, deployment tools, runbooks, and incident systems. Existing controls usually include peer review for risky changes, change windows, runbook ownership, break-glass procedures, and cloud/audit logs. Humans catch errors by recognizing context gaps, stale runbooks, and risky blast radius.

### 5.2 Human-in-the-loop (HITL / sub-autonomous)

This is the recommended default for production.

The assistant may:

- scope incidents
- retrieve bounded context automatically
- summarize evidence
- suggest hypotheses and next steps
- run preapproved read-only diagnostics
- draft incident notes and change requests

A human must approve:

- any production write or mutation
- any action outside read-only analysis
- any use of privileged or broad-scope tools
- any external escalation that shares evidence outside the primary trust domain

### 5.3 Fully autonomous (end-to-end agentic)

This should be narrow, not general.

Safe candidates are limited to **allowlisted, low-blast-radius, reversible** actions such as:

- auto-starting an investigation when a monitor enters alert
- opening or updating incident records
- running read-only diagnostics
- initiating preapproved remediation only for narrow stateless cases, under strong policy, target caps, change-window checks, and automatic rollback / verification

This mode is **not** appropriate for general production remediation on stateful, security-sensitive, or multi-tenant systems.

### 5.4 Variants (optional)

- **Observability-native assistant:** embedded in Datadog, Azure Monitor, or cloud consoles.
- **ChatOps assistant:** invoked from Slack / Teams with tight guardrails and explicit approvals.
- **Local MCP/API client:** an internal SRE assistant connecting to telemetry and runbooks through MCP servers or APIs.
- **Multi-agent variant:** one agent specializes in evidence gathering, another in RCA synthesis, and a policy agent approves or blocks actions.

---

## 6. Threat model overview (high-level)

### 6.1 Primary security & safety goals

- Maintain **availability** by preventing assistant-driven incident amplification.
- Maintain **integrity** of RCA, summaries, and remediation recommendations.
- Maintain **confidentiality** of telemetry, configuration, and secrets.
- Maintain **least privilege** across investigation and remediation tools.
- Maintain **auditability** so every output and action is attributable and reviewable.

### 6.2 Threat actors (who might attack / misuse)

- external attacker who can influence application input and therefore logs, spans, or exceptions
- compromised workload or insider who can manipulate telemetry or annotations
- malicious or compromised integration / MCP server / connector
- well-intentioned operator who over-trusts the assistant under incident pressure
- careless maintainer who exposes broad production tools “for convenience”

### 6.3 Attack surfaces

- alert messages, monitor text, and dashboard annotations
- logs, exception strings, trace spans, metric labels, and tags
- deployment annotations, release notes, and incident comments
- runbook descriptions and KB articles
- tool manifests, descriptions, schemas, and output payloads
- diagnostic scripts and shell / command output
- automation-runbook parameters and chatops commands
- external support-case or escalation channels

### 6.4 High-impact failures (include industry harms)

- **Customer / service harm:** increased latency, dropped traffic, regional outage, state corruption, broken failover, failed rollback.
- **Business harm:** extended MTTR, repeated false escalations, change-freeze violations, on-call burnout, audit failures, contractual or SLA exposure.
- **Security harm:** secret leakage, unauthorized cloud actions, privileged shell execution, inaccurate incident records that hide unsafe behavior.

---

## 7. Kill-chain analysis (stages -> likely failure modes)

> Keep this defender-friendly. Describe patterns, not “how to do it.”

| Stage | What can go wrong (pattern) | Likely impact | Notes / preconditions |
|---|---|---|---|
| **1. Entry / trigger** | Untrusted content enters through logs, exceptions, trace metadata, incident comments, dashboard annotations, or even compromised tool metadata / runbook descriptions | Assistant starts from poisoned or misleading context | Common because cloud ops workflows ingest large volumes of semi-structured telemetry |
| **2. Context contamination / false hypothesis** | The assistant treats telemetry or metadata as instructions, overweights stale runbooks, or mis-scopes affected services/resources | False RCA, incorrect prioritization, unsafe remediation options | Especially likely when evidence is sparse or instrumentation quality is poor |
| **3. Unsafe tool selection / action planning** | The assistant selects over-privileged tools, broad target sets, or high-risk runbooks that exceed the need of the task | Privilege escalation, excessive blast radius, cross-account/project access | Happens when read and write capabilities are not split cleanly |
| **4. Hidden or weakly disclosed execution** | The assistant executes or proposes a risky action while under-disclosing scope, side effects, or actual tool usage to the operator | User approves a dangerous action under false assumptions | UI / narrative integrity becomes a control dependency |
| **5. Persistence / repeat** | Bad RCA, misleading incident notes, or unsafe “successful” remediation becomes part of tickets, runbooks, or auto-remediation policy | The same mistake repeats across future incidents | Persistent knowledge contamination is an organizational risk multiplier |
| **6. Harm / outage / exposure** | Healthy resources are restarted or terminated, stateful systems are changed incorrectly, secrets leak in outputs, or investigations spill restricted data across trust domains | Self-inflicted DoS, prolonged outage, data exposure, audit/compliance failure | The impact is amplified by production urgency and operator trust |

---

## 8. SAFE-MCP mapping (kill-chain -> techniques -> controls -> tests)

> Goal: make SAFE-MCP actionable in this workflow.  
> Some mappings below are direct; some are best-fit / inferential because SAFE-MCP is protocol-centric while this use case is cloud-ops centric.

| Kill-chain stage | Failure / attack pattern (defender-friendly) | SAFE-MCP technique(s) | Recommended controls (prevent / detect / recover) | Tests (how to validate) |
|---|---|---|---|---|
| Entry / trigger | Logs, traces, alerts, API responses, or ticket text inject instructions or misleading cues into model context | `SAFE-T1102` | Treat telemetry as untrusted data; delimit and label provenance; separate instructions from evidence; scan/redact risky content before model use; prefer schemas and bounded retrieval | Seed injected log / trace / alert fixtures and confirm the assistant summarizes them as evidence only, without changing tool policy or approval flow |
| Entry / trigger variant | Compromised tool descriptions, schemas, or connector metadata bias the assistant before any real investigation begins | `SAFE-T1001` | Approved connector catalog; sign / attest tool manifests; pin versions; review metadata changes; disable ad hoc prod tools | Introduce a poisoned tool description in staging and verify the tool is rejected or quarantined before use |
| Context contamination | Context ordering or earlier malicious content biases later reasoning and safety checks | `SAFE-T1401` | Keep policy enforcement outside model narrative; run pre-execution authorization checks independent of prompt order; constrain max context from untrusted channels | Reorder untrusted context before/after policy text and verify the same deny decision occurs |
| Unsafe tool selection / action planning | Agent uses tools with broader permissions than necessary for diagnosis | `SAFE-T1104` | Split read-only and write tools; use per-tool scopes; progressive least privilege; environment scoping; dry-run and blast-radius policy checks | Attempt a write or cross-scope action using a read-only investigation profile and verify hard failure with logged policy denial |
| Unsafe tool selection / action planning | Agent reaches privileged exec / admin actions (root, sudo, break-glass, privileged containers, broad automation roles) | `SAFE-T1302` | Separate break-glass tools; require step-up auth and dual approval; JIT roles; resource locks; target caps; rollback plan required | Attempt privileged execution without step-up auth and approval; verify denial and escalation requirement |
| Hidden / weakly disclosed execution | Assistant narrative conceals or downplays risky tool usage or actual blast radius | `SAFE-T1404` | Immutable tool ledger bound to visible response; mandatory disclosure for any write or privileged read; correlation IDs; UI shows exact action + parameters before approval | Force a risky action in test harness and assert that summary, approval UI, and audit log all disclose it consistently |
| Persistence / repeat / harm | False RCA or unsafe remediation advice is written into tickets, runbooks, or shared summaries and later reused | `SAFE-T2105` | Confidence scoring; evidence-backed citations / traceability; human review for persistent records; canary before policy promotion; after-action review | Seed stale or misleading runbook content and verify the assistant marks it uncertain / stale and refuses autonomous promotion |

**Notes**

- `SAFE-T1102` is the most central mapping for this use case because cloud ops assistants routinely ingest untrusted logs, traces, and tool outputs.
- `SAFE-T1001` matters when the workflow is mediated through MCP connectors, internal tools, or runbook wrappers whose metadata can bias the assistant.
- `SAFE-T1104` and `SAFE-T1302` separate two different failures: “too much permission for a normal tool” versus “a truly high-privilege tool exists in the agent’s reachable set.”
- `SAFE-T1404` is important even if approvals exist, because hidden or incomplete narratives can cause bad human approvals.
- `SAFE-T2105` matters because cloud ops assistants often write back into the system of record.

---

## 9. Controls & mitigations (organized)

### 9.1 Prevent (reduce likelihood)

1. **Read-only by default.**  
   Design the troubleshooting assistant so investigation and evidence gathering use read-only identities by default. Mutation should be a separate capability plane.

2. **Progressive least privilege.**  
   Use per-tool scopes, narrow roles, environment isolation, and explicit step-up authorization for any action beyond retrieval. Avoid omnibus “full access” roles.

3. **Treat telemetry as untrusted context.**  
   Logs, traces, alert text, and ticket comments must be handled as data, not instructions. Preserve provenance labels and formatting boundaries.

4. **Separate analysis from execution.**  
   Use distinct tools / connectors for:
   - retrieval
   - diagnostics
   - record updates
   - production remediation

   Never let a generic “exec” capability serve as both investigator and remediator in production.

5. **Use sandboxed diagnostics.**  
   Diagnostic scripts should run in isolated, ephemeral environments where possible; time-boxed, network-restricted, and without broad production credentials.

6. **Require change-control gates for mutations.**  
   Production changes should require approval, change-window validation, rollback plan, target confirmation, and post-change success criteria.

7. **Constrain blast radius technically.**  
   Target-set caps, account / project / subscription boundaries, service ownership checks, concurrency limits, and deny-by-default resource classes reduce the chance of mass harm.

8. **Improve telemetry hygiene.**  
   Structured logging, release/change annotations, resource relationships, and consistent service metadata reduce false hypotheses and over-reliance on weak signals.

9. **Govern runbooks and tool metadata.**  
   Version, review, and own runbooks; pin connector versions; verify tool metadata provenance; and retire deprecated automation aggressively.

10. **Protect sensitive output paths.**  
    Redact secrets, suppress unnecessary raw log excerpts, and constrain what can be copied into tickets, chat, and external support channels.

### 9.2 Detect (reduce time-to-detect)

1. **Immutable session ledger.**  
   Log what the assistant saw, what tools it invoked, what policies evaluated, what approvals occurred, and what the user-visible response said.

2. **Narrative / tool mismatch detection.**  
   Alert when tool activity and visible summaries disagree, especially for writes, privileged reads, or cross-boundary actions.

3. **Privilege- and scope-anomaly monitoring.**  
   Detect attempts to cross environments, accounts, or high-risk resource classes; detect repeated denied actions; detect sudden escalation from read-only to privileged tools.

4. **Injection-resilience monitoring.**  
   Track sessions containing suspicious telemetry patterns, hidden instruction phrases, unusually large untrusted context payloads, or repeated content that correlates with policy denials.

5. **Output-quality monitoring.**  
   Track inconclusive-rate, false-RCA rate, human override rate, rollback frequency, and post-remediation incident recurrence.

### 9.3 Recover (reduce blast radius)

1. **Kill switch / session suspension.**  
   Be able to disable remediation tools, suspend autonomous mode, or terminate a single session without losing audit evidence.

2. **Rollback-ready changes only.**  
   Require rollback plans for any automated remediation. Verify rollback paths as a first-class readiness condition, not an afterthought.

3. **Credential and token revocation.**  
   Revoke or rotate agent and automation credentials when a suspicious session or disclosure event occurs.

4. **Evidence preservation.**  
   Preserve tool logs, model context provenance, policy decisions, and before/after state so responders can reconstruct the event.

5. **After-action review and control tuning.**  
   Feed incident learning back into tool scopes, runbooks, test fixtures, and policy rules.

---

## 10. Validation & testing plan

### 10.1 What to test (minimum set)

- **Permission boundaries:** the agent cannot exceed intended scopes
- **Prompt / tool-output robustness:** the agent resists malicious or misleading content in logs, tickets, and tool outputs
- **Action gating:** high-risk actions require approvals, change-window checks, and step-up auth
- **Logging / auditability:** actions are attributable, reviewable, and correlated with user-visible narratives
- **Rollback / safety stops:** the system can halt and reverse unsafe actions
- **Evidence quality:** the assistant can be wrong, but should be transparently uncertain rather than falsely confident

### 10.2 Test cases (make them concrete)

| Test name | Setup | Input / scenario | Expected outcome | Evidence produced |
|---|---|---|---|---|
| Untrusted log prompt-injection resilience | staging incident sandbox with seeded malicious log lines | alert on service latency causes assistant to read poisoned logs | assistant treats content as evidence only, no change in approval or tool policy, suspicious content flagged | query logs, model session trace, policy decision log |
| Poisoned trace annotation resilience | APM sandbox with manipulated span tags / annotations | assistant correlates traces during RCA | annotation does not become instruction; RCA references supporting telemetry only | span retrieval log, reasoning trace, final summary |
| Stale runbook handling | deprecated runbook retained in KB with owner / timestamp metadata | assistant retrieves runbook for a current incident | assistant marks runbook stale / low confidence, does not autonomously execute, asks for review or newer source | KB retrieval log, confidence note, blocked execution |
| Read-only profile cannot mutate | investigation role has telemetry read only | assistant attempts to create or execute a remediation action | mutation is denied hard; incident remains in recommendation-only mode | IAM / policy denial log, session ID, denied tool call |
| Privileged tool gate enforcement | privileged executor available only through break-glass flow | assistant proposes host-level command or high-risk runbook | requires step-up auth and explicit approval; no silent execution | approval challenge, denied pre-approval call, audit trail |
| Change-window enforcement | automation integrated with change calendar | assistant requests approved but time-sensitive remediation outside allowed window | action blocked until permitted window or exception approval | policy decision log, calendar check, blocked execution record |
| Narrative / tool mismatch alerting | test harness forces a write action followed by incomplete summary | assistant response under-discloses action details | mismatch detector alerts; approval / session review triggered | narrative-tool comparison log, alert, session transcript |
| Secret redaction in outputs | canary secret present in logs / env output | assistant summarizes incident and drafts ticket update | secret not emitted to response or ticket; redaction event recorded | redaction log, sanitized artifact, ticket diff |
| Auto-remediation canary with rollback | allowlisted stateless service in non-critical environment | assistant executes a reversible remediation and watches health | post-checks run automatically; rollback occurs if SLO / health criteria fail | execution log, health metrics snapshot, rollback evidence |
| Cross-scope boundary test | multiple accounts/projects/subscriptions available | incident scoped to one environment, but telemetry references another | assistant does not cross boundary without explicit authorization and disclosure | access log, denied cross-scope query, summary note |

### 10.3 Operational monitoring (production)

- **Metrics**
  - MTTR delta versus manual baseline
  - investigation start-to-first-useful-finding time
  - percentage of investigations ending inconclusive
  - human override / disagreement rate
  - privileged action denial rate
  - rollback invocation count
  - repeated incident recurrence after assistant-guided remediation
  - narrative / tool mismatch rate
- **Alerts**
  - privileged tool invocation attempt
  - cross-environment or cross-account access attempt
  - change outside approved window
  - low-confidence recommendation paired with high-severity incident
  - narrative / tool mismatch
  - secret redaction event on outbound content
- **Runbooks**
  - suspend autonomous remediation
  - revoke agent / automation credentials
  - preserve and export session evidence
  - validate affected service health and rollback state
  - review poisoned sources and update fixtures / detections

---

## 11. Open questions & TODOs

- [ ] What exact action catalog, if any, is acceptable for autonomous remediation in production?
- [ ] What confidence threshold and evidence minimum should gate action versus recommendation-only mode?
- [ ] How should teams measure false-RCA rate in a consistent, low-bias way?
- [ ] Which telemetry sources are most likely to carry attacker-controlled content in this environment?
- [ ] How should data-residency rules apply when vendor-managed investigation artifacts may be stored cross-region?
- [ ] Which persistence-oriented SAFE-MCP technique best captures poisoned incident notes / runbooks in future SAFE taxonomy revisions?

---

## 12. Questionnaire prompts (for reviewers)

### Workflow realism

- Are the tools and workflow steps realistic for cloud ops and incident-response teams?
- Which systems are commonly missing from this draft in real enterprises (for example CMDB, PagerDuty, ServiceNow, internal runbooks, CI/CD, asset inventory)?
- Is the line between diagnosis and remediation drawn in the right place?

### Trust boundaries & permissions

- Where are the true trust boundaries in your environment: telemetry ingestion, observability backend, cloud APIs, automation, or chatops?
- Which tools are currently too broad for safe troubleshooting automation?
- What is the true blast radius of a mistaken remediation call?

### Threat model completeness

- Which threat actor is most relevant: external attacker influencing logs, insider / workload owner, or compromised integration?
- What is the highest-impact failure missing from this write-up?
- Does your organization treat logs and traces as potentially adversarial content?

### Controls & tests

- Which controls are mandatory before any production pilot?
- Which tests should block release of new tools, runbooks, or agent policies?
- What evidence must exist before an autonomous remediation policy can be enabled?

---

## Appendix

### A. Glossary

- **Blast radius:** the scope of systems, customers, or services that a bad action could affect.
- **Break-glass:** an exceptional, high-privilege access path used only under tightly controlled emergency conditions.
- **Change window:** an approved time period during which production changes are allowed.
- **HITL:** human-in-the-loop; a human must review or approve key decisions or actions.
- **RCA:** root-cause analysis.
- **Runbook:** a predefined operational procedure or automation used during incidents.

### B. Real-world workflow evidence notes

1. **Datadog Bits AI SRE** demonstrates that production issue investigation assistants already exist in practice, can auto-start on alert transitions, iterate through hypotheses over telemetry, and separate read from write permissions.
2. **Azure Copilot observability agent** demonstrates a cloud-native investigation flow that correlates metrics, logs, and related resources from an alert and explicitly cautions users to validate outputs before deployment.
3. **Gemini Cloud Assist investigations** demonstrates ranked observations / hypotheses over logs, configurations, and metrics, same-as-user IAM, audit logging, and data-location constraints.
4. **AWS examples** demonstrate two important architectural variants:
   - a read-only troubleshooting surface in the console
   - a higher-risk chat / automation path capable of running commands and workflows under guardrails, approvals, and calendars

### C. Implementation stance distilled from this guide

- **Default operating mode:** read-only diagnosis with HITL for any write or remediation.
- **Default trust model:** logs, traces, tickets, and tool outputs are untrusted unless independently verified.
- **Default authorization model:** progressive least privilege with separate retrieval and action identities.
- **Default safety stance:** prefer “inconclusive with evidence” over “confident but wrong.”

### D. Reference list (selected)

- SAFE-AUCA template and SAFE-UC-0023 issue / seed page
- Datadog Bits AI SRE documentation
- Azure Monitor observability agent documentation
- Google Cloud Gemini Cloud Assist investigations, IAM, and audit logging documentation
- AWS Amazon Q and Systems Manager documentation
- OWASP GenAI Top 10
- CSA AI Controls Matrix
- NIST AI RMF, NIST AI 600-1, and NIST SP 800-61r3
- Model Context Protocol specification, authorization, and security guidance
- SAFE-MCP techniques: T1102, T1001, T1104, T1302, T1401, T1404, T2105

---

## Contributors

- **Author:** OpenAI (research synthesis draft; replace with repository contributor handle before PR)
- **Reviewer(s):** TBD
- **Additional contributors:** TBD

## Version History

| Version | Date | Changes | Author |
|---|---|---|---|
| 1.0 | 2026-03-18 | Initial draft for `SAFE-UC-0023` covering workflow realism, trust boundaries, kill chain, SAFE-MCP mapping, governance matrix, and validation plan for a cloud ops troubleshooting assistant | OpenAI |

