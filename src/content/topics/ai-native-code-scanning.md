---
title: "AI-native Code Scanning"
maturity: emerging
ml_types:
  - "Large Language Models"
departments:
  - "AppSec & Product Security"
hero_image:
  src: "../../assets/topics/ai-native-code-scanning/cost-curve.png"
  alt: "Log-log chart of cost per AI scan against codebase size, from 1k to 1M lines of code. Three lines: an optimised linear baseline at $0.88 per thousand lines, a linear upper estimate at $3 per thousand lines for complex code, and a super-linear naive curve that grows much faster when the agent re-reads the whole context on every reasoning step. A measured point at 3k LOC, $2.64 sits between the lower two lines. Range markers at 50k LOC ($44 to $150 per scan), 500k LOC ($440 to $1500 efficient, up to about $1900 naive), and a small PR-diff annotation at 1.5k LOC ($1 to $6 per review)."
  caption: "Cost per scan against codebase size. Log-log scale. Selective use keeps you on the lower lines; continuous full-repo scanning drives you onto the red curve."
factors:
  business_impact: 8
  cost_efficiency: 3
  implementation_ease: 6
  capability_fit: 8
  data_readiness: 8
factor_notes:
  business_impact:
    - "+ Finds logic vulnerabilities and exploitable code paths that pattern-based SAST misses"
    - "+ Findings come with reasoning, severity assessment and remediation, ready for an engineer to act on"
    - "+ Reduces false-positive triage load that traditional SAST burns analyst hours on"
    - "- Real impact depends on selective use; naive continuous scanning destroys the business case via cost"
  cost_efficiency:
    - "- Cost scales roughly linearly with code size at best, super-linearly in naive setups (agent re-reads context every reasoning step)"
    - "- Full scan of a 500k LOC repository easily reaches four-figure cost per run"
    - "- Independent enterprise studies put annual cost for a 150-developer organisation in the $1M to $2.4M range"
    - "+ Selective use (PR diffs, pre-release scans only) collapses the cost down to dollars per review"
  implementation_ease:
    - "+ Tool surface is small. Cloud LLM API and a code-fetch script is enough to start"
    - "- Output is free-form text by default. Integrating into SARIF / SIEM / triage pipelines is real work"
    - "- Determining the right SDLC integration point (commit vs. PR vs. release) is a design decision, not a config"
  capability_fit:
    - "+ Code reasoning over realistic application context is one of the strongest current LLM capabilities"
    - "+ Authentication, data flow and exploit reasoning are areas where the model outperforms rules"
    - "- SCA / dependency vulnerability tracking is weak and inconsistent; coverage depends on the model's advisory knowledge at inference time"
    - "- Results are non-deterministic: repeated scans of the same code can return slightly different findings"
  data_readiness:
    - "+ Source code is already available in every engineering workflow"
    - "+ Repository structures are standardised enough that no preparation pipeline is needed"
    - "- Architecture context (design docs, threat model, deployment information) often missing; without it, the LLM works on incomplete inputs"
references:
  - url: "https://www.anthropic.com/research/mythos-preview"
    description: "Anthropic Frontier Red Team, Claude Mythos Preview (April 2026). The announcement that pushed AI-native vulnerability research into mainstream security discussion. Concrete numbers from the run: a 27-year-old OpenBSD vulnerability discovered for under $20k in roughly a thousand autonomous runs (under $50 per run), thousands of high- and critical-severity findings across major open-source projects, and a 89% match between Mythos severity assessments and expert human judgment."
  - url: "https://www.helpnetsecurity.com/2026/04/22/claude-mythos-mozilla-vulnerabilities-scanning/"
    description: "Mozilla's response to Mythos finding 271 Firefox flaws. Mozilla's framing is the most interesting part of the story: rather than treating the disclosure as a crisis, they argue that AI-native discovery 'shifts security toward defenders' because attackers no longer hold the asymmetric advantage of being the only ones running this kind of analysis. A useful counterweight to pure cost arguments."
  - url: "https://www.endorlabs.com/learn/the-token-economics-of-using-ai-coding-agents-for-security-tasks"
    description: "Endor Labs, the token economics of AI coding agents for security tasks. The most concrete public analysis of what continuous AI-native scanning actually costs at enterprise scale. Headline numbers: $600k to $1.3M annually for a 150-developer organisation in the initial estimate, revised to $1M to $2.4M after pricing updates; a real 50-engineer team that produced an $8400 first-month bill from a 50k-token security policy injected into every prompt. Also makes the case for using deterministic tools as scaffolding around the LLM, with up to 91.7% token reduction at preserved accuracy."
  - url: "https://www.bsi.bund.de/SharedDocs/Downloads/EN/BSI/Publications/Studies/ML-SAST/ML-SAST-Studie-final.pdf"
    description: "BSI study on ML-SAST. The academic predecessor: supervised-learning pattern recognition applied to source code for vulnerability discovery, well before LLM reasoning was viable. Useful both as a comparison point (what the rule-based ML approach can and cannot do) and as a methodology reference for evaluating any AI-based code scanner."
connections:
  - to: "smart-vulnerability-triage"
    relation: after
    note: "Findings from the AI scanner join the output of traditional SAST, DAST, SCA and threat-model tools in the central RAG-based triage hub for cross-tool deduplication and ranking."
  - to: "genai-threat-modeling"
    relation: before
    note: "Threat-model findings tell the AI scanner which components and severity classes to prioritise; without that context the scanner scans flat across the whole codebase."
  - to: "iac-policy-as-code-copilot"
    relation: parallel
    note: "Both apply LLM reasoning over code-shaped artifacts (source code vs. IaC), share the same cost-scaling and deployment patterns, and benefit from the same caching and selective-use strategies."
---

One of the conceptually simplest ways to apply AI to security is to let
an LLM read the codebase directly and look for vulnerabilities, without
the deterministic and rule-based scaffolding of a classical SAST tool.
The topic became unavoidable with Anthropic's Claude Mythos Preview in
April 2026: the Frontier Red Team's model autonomously discovered
thousands of high- and critical-severity vulnerabilities across major
open-source projects, including a 27-year-old OpenBSD bug, for under
$50 per autonomous research run. The question is no longer whether this
approach challenges established SAST tooling strategies, but how
deeply.

## 1. The anchor: a 27-year-old OpenBSD bug

OpenBSD is the operating system that takes security as a first-class
design principle. Its code has been audited continuously for almost
three decades, by some of the most cautious systems programmers in the
industry. In its Mythos Preview report Anthropic disclosed that the
model, working autonomously, found a vulnerability in that codebase
that had been present and undetected for 27 years. The whole research
run cost under $20,000 for roughly a thousand autonomous attempts, or
less than $50 per attempt.

The point is not the bug itself. The point is that the bug had survived
27 years of human review in a project that takes review seriously, and
an LLM found it for fifty dollars. A category of code analysis just
became available that did not exist last year.

## 2. The scale: Mozilla's 271 Firefox flaws

The OpenBSD story is about depth. The same Mythos run was also pointed
at the Firefox codebase and surfaced 271 distinct security flaws.
Anthropic measured 181 working autonomously generated exploits against
the JavaScript engine, against 2 generated by the previous-generation
Opus 4.6 model under the same conditions.

Mozilla's public response was, perhaps surprisingly, optimistic. Their
argument: until now attackers had asymmetric access to this kind of
analysis. With Mythos-class models available to defenders too, the
asymmetry shifts. They are not wrong, but they are also describing the
optimistic version of a story whose pessimistic version involves the
same models being available to people with different intentions.

For the purposes of this topic, the two stories taken together prove
the same thing in two different ways: the OpenBSD bug shows that
AI-native scanning can find *what years of human review missed*; the
Firefox numbers show that it can do so *at industrial volume*.

## 3. AI-native scanning vs. traditional SAST

The two are not interchangeable. They answer different questions.

| Aspect | Traditional SAST | AI-native scanning |
|---|---|---|
| Detection method | Rules, patterns, taint analysis | Reasoning over code and behaviour |
| Strongest at | Memory safety, known weakness patterns, large-codebase coverage | Logic flaws, exploit chains, context-dependent vulnerabilities |
| Determinism | Deterministic, repeatable | Non-deterministic, runs may differ |
| False-positive profile | High, requires triage layer | Lower, with built-in reasoning per finding |
| Output | Findings with CWE IDs and line numbers | Findings with explanation, exploitability assessment, fix |
| Speed | Seconds to minutes | Minutes to tens of minutes |
| Cost | Fixed licence | Per-token, scales with code size |
| SBOM, dependency tracking | Solid (when paired with SCA) | Weak; advisory coverage varies between runs |
| Auditability | Strong (every finding traces to a rule) | Weak by default (reasoning chain is not the same as a deterministic trace) |

The most accurate summary: AI-native scanning is a strong layer above
traditional SAST, not a replacement. It is the right tool for the
findings that pattern matchers cannot articulate, and the wrong tool
for the parts where pattern matchers are already optimal.

## 4. Cost reality: linear in theory, super-linear in practice

The chart at the top of the page is the most important part of this
topic for anyone evaluating production use.

**4.1 The linear baseline.** A direct measurement on a 3,000-line
repository (the secureIO internal evaluation) gives $2.64 per scan, or
$0.88 per thousand lines of code, using a GPT-5-class model with cache
hits where possible. Linearly extrapolated:

- 25k LOC → $22 to $60 per scan
- 100k LOC → $88 to $300 per scan
- 500k LOC → $440 to $1,500 per scan

The upper bound of each range reflects more complex codebases, more
frameworks and more entry points. None of this is unreasonable for an
occasional pre-release run. It is unreasonable as a per-commit cost.

**4.2 Why naive setups are worse than the line suggests.** The Endor
Labs analysis of AI coding agents for security shows the real failure
mode: agents re-send the accumulated context on every reasoning step.
A 50k-token security policy injected into every prompt generated an
$8,400 bill in the first month for a 50-engineer team. Scaled up,
their estimate for a 150-developer organisation running naive
AI-native scanning on every commit is $1M to $2.4M annually. The
super-linear red curve on the chart is what that looks like.

**4.3 Why token prices do not save you.** Per-token API prices have
been falling steadily, but model token *consumption* per task has been
rising at least as fast. Reasoning-augmented models and agentic loops
re-read context aggressively. The net cost per useful scan has been
roughly stable for two years and is not the part of this curve that
makes the business case viable.

**4.4 What does save you.** Caching, deterministic tool scaffolding
(deterministic SAST as a filter so the LLM looks at the right parts of
the code), and selective use. Run AI-native scans on PR diffs and
before major releases. Do not run them on every commit.

## 5. SDLC integration pattern

This is the closest thing to consensus across the public evaluations
of AI-native code scanning so far.

- **Per commit** — traditional SAST and SCA. Fast, cheap, deterministic.
  Catches the patterns it is configured to catch.
- **Per pull request** — AI-native review of the diff only. This is
  where the contextual reasoning pays for itself. A 1,000-line diff
  costs a few dollars to review at most.
- **Per release** — full AI scan of the changed module, occasionally a
  full-repo scan. Slower and more expensive, but the cost is amortised
  over a release cycle.
- **Findings** — routed through the existing triage hub so that AI and
  traditional findings are deduplicated and ranked together.

The triage step is where this topic connects to the rest of the
landscape: AI-native scanning is one more producer in the same funnel
that absorbs SAST, DAST, SCA and threat-model output.

## 6. Notes and limitations

**6.1 Non-determinism.** Two scans of the same code can give slightly
different findings. For high-severity issues this is rarely a
problem; for borderline cases it can be the difference between a
ticket opened and one missed. Validate critical findings on a
second pass.

**6.2 SCA stays with traditional tools.** Across every evaluation
visible so far, AI-native dependency analysis underperforms dedicated
SCA platforms. Vulnerability coverage depends on what the model
learned during training, not on a curated, time-stamped advisory
feed. Keep traditional SCA for SBOM, compliance reporting and any
audit that depends on repeatable matching.

**6.3 Cloud versus on-prem.** Cloud frontier models have stronger
reasoning. On-prem models reduce per-token cost and address data
residency. Most organisations will pay for cloud reasoning where the
codebase allows it and accept a quality cost on the parts that must
stay on-prem.

**6.4 Auditability.** A reasoning chain is not the same as a rule
trace. When an auditor asks "why was this finding raised?", the answer
"the model thought so" is less defensible than "rule X line Y". Plan
for this in regulated environments.

## 7. When this is the wrong tool

- **Continuous full-repository scanning on a large codebase.** The
  curve does not forgive this pattern. Move to selective use or to
  hybrid setups where deterministic tools do the first pass.
- **Compliance and SBOM workflows.** Dedicated SCA, an authoritative
  vulnerability database, and deterministic matching remain the
  defensible answer.
- **Highly regulated environments without auditable reasoning.**
  Reasoning chains are not yet a substitute for rule traces in
  contexts that require every finding to be traceably justified.
- **Closed-source binaries without context.** Mythos-class models can
  reverse-engineer, but the cost-quality trade-off swings hard against
  this use case for production-grade scanning.
