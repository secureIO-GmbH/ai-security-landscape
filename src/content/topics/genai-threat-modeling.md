---
title: "GenAI for Threat Modeling"
maturity: emerging
ml_types:
  - "Large Language Models"
departments:
  - "Security Engineering & Data Security"
  - "AppSec & Product Security"
hero_image:
  src: "../../assets/topics/genai-threat-modeling/mcp-threat-modeling-workflow.png"
  alt: "Platform-agnostic MCP workflow diagram. On the customer side: an agentic AI ecosystem, individual users via AI chat clients (Claude, ChatGPT, Gemini), developers via AI IDEs (Cursor, Windsurf, VSCode + Copilot), a CI/CD pipeline, and the security team. They all connect through a Threat Modeling Tool MCP Server (streamable HTTP or stdio) on the vendor side, which talks to the threat-modeling tool's backend. The security team can also reach the backend directly. External services such as wikis, ticketing and document repositories sit alongside the AI surfaces."
  caption: "MCP-based threat modelling during development. The reality, not the plan."
factors:
  business_impact: 8
  cost_efficiency: 5
  implementation_ease: 7
  capability_fit: 9
  data_readiness: 6
factor_notes:
  business_impact:
    - "+ Automates a process that, in practice, often never happens"
    - "+ STRIDE output flows directly into SDLC tools (Jira, GitHub Issues)"
    - "+ Keeps threat models in sync with each architectural change"
  cost_efficiency:
    - "- MCP-based threat modelling tools are not yet production-ready"
    - "- No mature open-source MCP option exists; commercial offerings are still emerging"
    - "- Every meaningful code change can trigger a new run (recurring token cost)"
  implementation_ease:
    - "+ MCP plugs cleanly into existing IDE and chat workflows"
    - "+ Works against existing IaC, OpenAPI specs and design docs"
    - "- Calibration to project-specific constraints is non-trivial"
  capability_fit:
    - "+ LLMs reason well over structured architecture inputs"
    - "+ Structured output (STRIDE, Open Threat Model) is reliably achievable"
    - "- Weak at quantitative risk scoring and domain-specific threats (ICS, mainframe)"
  data_readiness:
    - "+ Source code and IaC are always available"
    - "- Design docs are often incomplete, stale, or scattered"
    - "- OpenAPI / gRPC specs are inconsistently maintained across teams"
references:
  - url: "https://arxiv.org/abs/2603.22489"
    description: "Huang et al. (2025), \"Model Context Protocol Threat Modeling and Analyzing Vulnerabilities to Prompt Injection with Tool Poisoning.\" Applies STRIDE/DREAD to MCP integrations themselves and identifies tool poisoning as the most prevalent client-side vulnerability. Essential reading before deploying MCP-based threat modelling in production."
  - url: "https://github.com/iriusrisk/iriusrisk-cli"
    description: "IriusRisk CLI with MCP support. The reference implementation for production MCP-based threat modelling. Exposes the IriusRisk engine over a Streamable HTTP MCP server, usable from IDEs (Cursor, VSCode + Copilot), chat clients (Claude, ChatGPT), or agentic pipelines."
  - url: "https://github.com/secureIO-GmbH/security-skills"
    description: "secureIO security-skills, an open-source Claude Code skill suite. The `/threat-model` skill runs STRIDE analysis directly inside Claude Code using the conversation as the integration layer, without requiring an external MCP server or a commercial threat-modelling backend. A lightweight, vendor-free alternative for teams that already use Claude Code in their workflow and want threat modelling as a built-in command rather than a separate platform."
  - url: "https://github.com/mrwadams/stride-gpt"
    description: "STRIDE GPT, a popular open-source GPT wrapper for STRIDE. Not an MCP integration (simple chat interface, no tool calls), but a useful fallback and a clean reference for the prompting patterns used internally by MCP servers."
  - url: "https://owasp.org/www-community/Threat_Modeling"
    description: "OWASP Threat Modeling. Canonical reference for the manual STRIDE / PASTA / LINDDUN workflows that LLM tools automate."
connections:
  - to: "smart-vulnerability-triage"
    relation: after
    note: "Threat-model context (assets, threats, mitigations) feeds SAST prioritisation and false-positive reduction."
  - to: "iac-policy-as-code-copilot"
    relation: after
    note: "Threat-model findings (e.g. 'unauthenticated API') translate into automated IaC policies."
  - to: "critical-attack-path-detection"
    relation: after
    note: "Threat-model hypotheses are validated against real cloud asset data to surface the paths that are actually exploitable."
  - to: "llm-control-assessment"
    relation: parallel
    note: "Both apply LLMs to security-engineering documents (design docs, policies, IaC) and benefit from the same MCP integration patterns."
---

Use an LLM with structured tool access (typically over the Model Context
Protocol, MCP) to draft STRIDE-based threat models such as threat lists,
attack trees and mitigations from architecture diagrams, source code,
infrastructure-as-code and requirements documents. Threat modelling
becomes a continuous activity that runs alongside development, rather
than a one-off workshop before implementation that, in practice, rarely
happens on schedule.

## 1. Concrete example: MCP-driven threat modelling alongside development

The diagram above shows the realistic deployment shape. Walked through on a
single service it looks like this.

**Setup.** A team builds a new payment microservice. Source code lives in
GitHub, infrastructure in Terraform, the public API in an OpenAPI document.

**1.1 Inputs.** A security engineer (or an automation hook on `git push`)
invokes the threat-modelling tool through an MCP server (for example
IriusRisk's Streamable HTTP MCP). The MCP call hands over:

- The Terraform module (`aws_api_gateway_rest_api`, `aws_lambda_function`,
  `aws_db_instance` with its parameter group)
- The OpenAPI spec, including any `x-security` annotations
- The current `main` branch source for the Lambda
- A short design doc from Confluence

**1.2 Reasoning.** The LLM maps the inputs onto STRIDE:

- **Spoofing.** Lambda accepts only API-Key in header → weak; recommend AWS
  IAM SigV4 or Cognito-issued JWT.
- **Tampering.** API Gateway has no WAF attached in Terraform → threat.
- **Repudiation.** CloudTrail is enabled cluster-wide → mitigation confirmed.
- **Information Disclosure.** Database password is a Terraform variable, not
  pulled from AWS Secrets Manager → threat.
- **Denial of Service.** No rate-limiting on the API Gateway stage → threat.
- **Elevation of Privilege.** Lambda execution role grants `s3:*` on the
  whole bucket prefix → threat (least-privilege violation).

**1.3 Output.** The model emits an Open Threat Model (OTM) JSON. The MCP
client opens Jira tickets labelled `security/threat-model` directly on the
team's board. Each ticket carries the threat, the impacted component, and
a concrete mitigation, not a vague "this is unsafe".

**1.4 Continuous runs.** On every meaningful `git push`, the model re-runs.
The MCP client posts a diff of the threat model (new threats, resolved
threats, changed mitigations) into the team's chat. Architectural drift
becomes visible for the first time.

## 2. Why MCP matters here

Without MCP, threat-modelling tools live behind a separate web UI. In
practice this means:

- **2.1 Friction.** Developers have to copy architecture context into a
  separate tool, so they do not.
- **2.2 No continuity.** A threat model is generated once and goes stale
  the day the next PR merges.
- **2.3 No integration with the rest of the SDLC.** Findings stay in the
  threat-modelling tool instead of flowing into the issue tracker, the
  pipeline, or the IDE.

MCP collapses this. The model lives behind a tool call; the call can be
made from the IDE (Cursor, Windsurf, VSCode + Copilot), from a chat client
(Claude, ChatGPT, Gemini), or from CI. The threat model becomes a live
artifact rather than a document.

A standalone GPT wrapper such as STRIDE GPT does an acceptable job at the
reasoning step, but it has no tool layer. It is a chat interface, not an
integration, and is best treated as a learning aid or fallback rather than
a deployment target.

## 3. Use cases

- **3.1 Pre-implementation design review (SecEng).** New service:
  architecture diagram + IaC produce a STRIDE table and mitigations before
  code is written.
- **3.2 Continuous threat modelling alongside SAST/DAST (AppSec).** Code is
  already running. MCP runs threat modelling on every `main` branch update
  to find drift between intended design and actual implementation.
- **3.3 IDE-level code reasoning.** Developer writes authentication code;
  the MCP-driven hook flags threat-level issues (for example, JWT
  validated without issuer check → Spoofing) rather than only low-level
  CWE matches.
- **3.4 Re-runs on architectural change.** New external dependency, new
  cloud provider, new deployment region. Re-run, diff, route findings to
  the team.
- **3.5 Threat models as input to policy generation.** Threats like
  "unencrypted transit" map directly to IaC policies
  (`require-https-only=true`) handed off to the IaC copilot pipeline.
- **3.6 Hypothesis validation against the live estate.** A threat model
  hypothesises "lateral movement via unpatched RDS". The same finding is
  passed to attack-path detection (see related topic) to check whether the
  path is actually exploitable in the live environment.

## 4. Doesn't this just collapse into SAST?

A fair question. If MCP-based threat modelling runs on every push and
inspects the source code directly, where does the line to static code
analysis sit? The short answer: the two ask **different questions and
produce different artifacts**, and they are complementary rather than
overlapping.

| Aspect | Threat Modelling (LLM + MCP) | SAST |
|---|---|---|
| Primary question | Who could attack this, how, and why? | Which lines of code are unsafe? |
| Unit of analysis | Components, data flows, trust boundaries | Functions, expressions, taint paths |
| Inputs | Architecture, IaC, API spec, code, design docs | Source code (sometimes binaries) |
| Output | Threats + mitigations, STRIDE categories, OTM JSON | Findings list with CWE IDs and line numbers |
| Strength | Reasoning across the whole system, spanning files, components and services | Deterministic pattern matching with low false-negative rate per pattern |
| Weakness | Higher false positives, hard to audit reasoning chain | Blind to design-level issues that span files or systems |
| Timing | Design phase + continuous diffs | Pre-commit / per-build |
| Typical fix | Architectural change, control added, requirement clarified | Code change in a specific line / function |

**4.1 The overlap is real but narrow.** Both can spot "this endpoint is
unauthenticated". SAST sees it as a missing authorization decorator on a
function; threat modelling sees it as a Spoofing risk on the trust boundary
between caller and service. The fix may even be the same line of code, but
the framing differs. That framing determines whether the problem is treated
as "one bug" or "a class of weakness to design out".

**4.2 The genuine differences.** Threat modelling reasons across multiple
files, IaC, runtime configuration and external dependencies at the same
time. SAST cannot, because it has no notion of system topology. Conversely,
SAST will find a SQL-injection pattern in a single helper that the LLM
might miss because it never looked at that file in detail. Treat them as
**complementary layers**: threat modelling sets the priority and direction;
SAST executes against the prioritised areas with higher sensitivity.

**4.3 Where they should integrate.** Pass the threat model's component
list and severity to SAST as a tuning input. Increase sensitivity for
Injection patterns in components the threat model flagged as
"Tampering / Information Disclosure"; reduce noise in low-risk components.
This is exactly the `smart-vulnerability-triage` link.

## 5. Notes & limitations

**5.1 Output quality depends on input structure.** Best results with
strongly structured inputs (IaC + OpenAPI). Narrative-only design docs give
vague, generic output.

**5.2 Always require human review.** LLMs hallucinate threats and miss
domain-specific ones. The role of the security engineer shifts from
"writing the threat model" to "reviewing and curating the model the LLM
produced".

**5.3 The MCP integration is itself a threat-modelling target.** Huang et
al. (2025) show that MCP clients are vulnerable to tool poisoning. A
malicious MCP server (or a compromised one) can return manipulated threat
models, leading teams to trust false mitigations. Prompt injection via IaC
variables is also a realistic attack against this pipeline. Vet the MCP
source, validate inputs, and review the output before any automated
mitigation lands.

## 6. When this is the wrong tool

- **Mainframe / legacy stacks.** LLM training coverage of e.g. CICS, IMS or
  proprietary middleware is shallow; STRIDE output stays generic.
- **Highly sensitive architectures.** Sending the full system into a hosted
  LLM may not be acceptable; on-prem models lag in quality.
- **Documentation-only environments.** No IaC, no API specs, only Word
  documents. The LLM works on weak structured input and produces weak
  output.
- **Audit-trail-heavy compliance regimes.** When every threat must be
  traceably justified, LLM output is harder to defend than a documented
  workshop with named participants.
