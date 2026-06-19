---
title: "AI-native Code Scanning"
maturity: emerging
ml_types:
  - "Large Language Models"
departments:
  - "AppSec & Product Security"
factors:
  business_impact: 8
  cost_efficiency: 3
  implementation_ease: 6
  capability_fit: 8
  data_readiness: 8
factor_notes:
  business_impact:
    - "+ Finds logic and exploit-chain vulnerabilities pattern-based SAST misses"
    - "+ Findings ship with reasoning, severity and remediation"
    - "- Value depends on selective use; naive continuous scanning destroys the business case"
  cost_efficiency:
    - "- Linear at small scale, super-linear at production scale"
    - "- A 500k LOC scan easily reaches four-figure cost per run"
    - "+ Selective use (PR diffs, pre-release) keeps cost at dollars per review"
  implementation_ease:
    - "+ Small tool surface; cloud LLM API plus a code-fetch script is enough to start"
    - "- Free-form output; SARIF / SIEM integration is real work"
    - "- SDLC integration point (commit vs. PR vs. release) is a design decision"
  capability_fit:
    - "+ Code reasoning over real application context is an LLM sweet spot"
    - "+ Authentication, data flow and exploit reasoning beat rule-based detection"
    - "- Non-deterministic; SCA coverage depends on what the model already knows"
  data_readiness:
    - "+ Source code is available in every engineering workflow"
    - "+ Repository structures need no preparation pipeline"
    - "- Architecture context (design docs, threat models) is often missing"
references:
  - url: "https://www.anthropic.com/research/mythos-preview"
    description: "Anthropic Frontier Red Team, Claude Mythos Preview (April 2026). The announcement that pushed AI-native vulnerability research into mainstream security discussion. Concrete numbers from the run: a 27-year-old OpenBSD vulnerability discovered for under $20k in roughly a thousand autonomous runs (under $50 per run), thousands of high- and critical-severity findings across major open-source projects, and a 89% match between Mythos severity assessments and expert human judgment."
  - url: "https://www.helpnetsecurity.com/2026/04/22/claude-mythos-mozilla-vulnerabilities-scanning/"
    description: "Mozilla's response to Mythos finding 271 Firefox flaws. Mozilla's framing is the most interesting part of the story: rather than treating the disclosure as a crisis, they argue that AI-native discovery 'shifts security toward defenders' because attackers no longer hold the asymmetric advantage of being the only ones running this kind of analysis. A useful counterweight to pure cost arguments."
  - url: "https://www.endorlabs.com/learn/the-token-economics-of-using-ai-coding-agents-for-security-tasks"
    description: "Endor Labs, the token economics of AI coding agents for security tasks. The most concrete public analysis of what continuous AI-native scanning actually costs at enterprise scale. Headline numbers: $600k to $1.3M annually for a 150-developer organisation in the initial estimate, revised to $1M to $2.4M after pricing updates; a real 50-engineer team that produced an $8400 first-month bill from a 50k-token security policy injected into every prompt. Also makes the case for using deterministic tools as scaffolding around the LLM, with up to 91.7% token reduction at preserved accuracy."
  - url: "https://arxiv.org/pdf/2602.22518"
    description: "RepoMod-Bench, a benchmark for code repository modernization via implementation-agnostic testing. The most concrete academic evidence of super-linear scaling of agent cost with codebase size: pass rate drops from 91% on codebases below 10k LOC to 15% above 50k LOC, with most codebases above 100k LOC averaging below 20%. Useful when arguing that linear cost extrapolation underestimates production-scale spend."
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

## 1. The catalyst: Mythos and what it found

Anthropic's Claude Mythos Preview, published in April 2026, is the
single case study that pushed AI-native scanning into mainstream
security conversation. Pointed at the OpenBSD codebase, audited
continuously for almost three decades by some of the most cautious
systems programmers in the industry, Mythos autonomously found a
vulnerability that had been present and undetected for 27 years, for
under $50 per autonomous research run. Pointed at Firefox in the same
campaign, it surfaced 271 distinct security flaws and produced 181
working JavaScript-engine exploits, against 2 generated by the
previous-generation Opus 4.6 model under identical conditions. The
OpenBSD finding shows what AI-native scanning can dig out of code that
humans review with extreme care; the Firefox numbers show that it can
do it at industrial volume. Mozilla's public response was, perhaps
surprisingly, optimistic: this kind of analysis "shifts security toward
defenders" by closing the asymmetric access that attackers used to
have.

## 2. AI-native scanning vs. traditional SAST

The two are not interchangeable. They answer different questions.

<table class="comparison-table">
  <thead>
    <tr>
      <th>Aspect</th>
      <th>Traditional SAST</th>
      <th>AI-native scanning</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Detection method</td>
      <td>Rules, patterns, taint analysis</td>
      <td>Reasoning over code and behaviour</td>
    </tr>
    <tr>
      <td>Strongest at</td>
      <td>Memory safety, known weakness patterns, large-codebase coverage</td>
      <td>Logic flaws, exploit chains, context-dependent vulnerabilities</td>
    </tr>
    <tr>
      <td>Determinism</td>
      <td class="positive">Deterministic, repeatable</td>
      <td class="negative">Non-deterministic, runs may differ</td>
    </tr>
    <tr>
      <td>False-positive profile</td>
      <td class="negative">High, requires triage layer</td>
      <td class="positive">Lower, reasoning built into each finding</td>
    </tr>
    <tr>
      <td>Output</td>
      <td>CWE IDs and line numbers</td>
      <td>Explanation, exploitability assessment, fix suggestion</td>
    </tr>
    <tr>
      <td>Speed</td>
      <td>Seconds to minutes</td>
      <td>Minutes to tens of minutes</td>
    </tr>
    <tr>
      <td>Cost</td>
      <td class="positive">Fixed licence, predictable</td>
      <td class="negative">Per-token, scales with code size</td>
    </tr>
    <tr>
      <td>SBOM / dependency tracking</td>
      <td class="positive">Solid when paired with SCA</td>
      <td class="negative">Weak; advisory coverage varies between runs</td>
    </tr>
    <tr>
      <td>Auditability</td>
      <td class="positive">Strong, every finding traces to a rule</td>
      <td class="negative">Weak, reasoning chain is not a deterministic trace</td>
    </tr>
  </tbody>
</table>

The most accurate summary: AI-native scanning is a strong layer above
traditional SAST, not a replacement. It is the right tool for the
findings that pattern matchers cannot articulate, and the wrong tool
for the parts where pattern matchers are already optimal.

## 3. Cost reality: linear at small scale, super-linear above

The chart at the top of the page is the most important part of this
topic for anyone evaluating production use.

**3.1 The linear anchor at small scale.** A direct measurement on a
3,000-line repository (the secureIO internal Codex evaluation) gives
$2.64 per scan, or $0.88 per thousand lines of code, using a
GPT-5-class model with cache hits where possible. At small scale this
behaves linearly, and a straight extrapolation gives a useful
ballpark: 25k LOC ≈ $22, 100k LOC ≈ $88, 500k LOC ≈ $440. That is
the lower edge of the band on the chart.

**3.2 Why production scale is super-linear.** Three independent
sources point at super-linear behaviour once the codebase grows beyond
roughly 10k LOC:

- Endor Labs measured agents re-sending the entire accumulated context
  on every reasoning step. On their benchmark a naive LLM-only agent
  consumed 79.5M tokens; replacing the same workflow with deterministic
  tools cut that to 6.6M tokens, a 91.7% reduction at preserved
  accuracy. Cost per LOC was effectively decoupled from raw code volume
  and dominated by reasoning-loop overhead.
- The RepoMod-Bench academic benchmark observed a "scaling collapse":
  agents pass 91% of tasks on codebases below 10k LOC and 15% on
  codebases above 50k LOC. Useful-output cost scales worse than token
  cost.
- The same Endor analysis projects $1M to $2.4M annually for a
  150-developer organisation running naive per-commit AI-native
  scanning, against six- to seven-figure savings when scans are
  selective and tool-augmented.

The upper edge of the band on the chart is a power-law approximation
with exponent 1.25, calibrated to those observations. At 500k LOC it
reaches roughly $2,000 per scan, against $440 on the linear lower edge.

**3.3 Why falling token prices do not save you.** Per-token API prices
have been falling steadily, but model token *consumption* per task has
been rising at least as fast. Reasoning-augmented models and agentic
loops re-read context aggressively. The net cost per useful scan has
been roughly stable for two years.

**3.4 What does save you.** Caching, deterministic tool scaffolding
(deterministic SAST as a filter so the LLM looks at the right parts of
the code), and selective use. Run AI-native scans on PR diffs and
before major releases. Do not run them on every commit. The shaded
band on the chart is where this strategy sits; the naive everything-
continuously alternative sits well above the upper edge.

One caveat on PR-diff scans: a naive scan that only sees the changed
lines misses logic flaws that span the boundary between changed and
unchanged code, including most authorisation bypasses and trust-model
mistakes. Practical deployments pull the surrounding files, the
callers of touched functions and the relevant threat-model entries
into the prompt before sending. That keeps the scan useful and the
cost on the lower curve, but it is pipeline work, not a configuration
toggle.

## 4. SDLC integration pattern

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

## 5. Notes and limitations

**5.1 Non-determinism.** Two scans of the same code can give slightly
different findings. For high-severity issues this is rarely a
problem; for borderline cases it can be the difference between a
ticket opened and one missed. Validate critical findings on a
second pass.

**5.2 SCA stays with traditional tools.** Across every evaluation
visible so far, AI-native dependency analysis underperforms dedicated
SCA platforms. Vulnerability coverage depends on what the model
learned during training, not on a curated, time-stamped advisory
feed. Keep traditional SCA for SBOM, compliance reporting and any
audit that depends on repeatable matching.

**5.3 Cloud versus on-prem.** Cloud frontier models have stronger
reasoning. On-prem models reduce per-token cost and address data
residency. Most organisations will pay for cloud reasoning where the
codebase allows it and accept a quality cost on the parts that must
stay on-prem.

**5.4 Auditability.** A reasoning chain is not the same as a rule
trace. When an auditor asks "why was this finding raised?", the answer
"the model thought so" is less defensible than "rule X line Y". Plan
for this in regulated environments.

## 6. When this is the wrong tool

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
