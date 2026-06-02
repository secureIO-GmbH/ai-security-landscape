# Contributing

Thanks for considering a contribution! The most useful thing you can add is a
new topic to the landscape. You don't need to write any code or run a build.
A single Markdown file is enough.

## Add a new topic in five steps

1. **Fork the repository** on GitHub.
2. **Copy the template**:
   `src/content/topics/_template.md` → `src/content/topics/your-topic-slug.md`.
   Use a short, lowercase, dash-separated filename (e.g.
   `automated-dlp-labeling.md`). The filename becomes the topic's slug, and
   it shows up in URLs and in cross-references from other topics. Pick
   something stable.
3. **Fill in the frontmatter** at the top of the file. Every field is
   explained inline in the template. Required fields:
   - `title`
   - `maturity` (`established` / `emerging` / `experimental`)
   - `ml_types` (at least one, see below)
   - `departments` (at least one, see below)
   - `factors` (five scores, each 0 to 10, see below)
4. **Write a short description** in Markdown below the frontmatter. Two
   to four sentences is plenty for a minimal topic. For richer topics,
   see *Body structure* below.
5. **Open a pull request.** CI runs `astro check` and a full Astro build
   on every PR. If your frontmatter is invalid or a connection points to a
   slug that doesn't exist, the check fails and the PR shows you where to
   look. The maintainer then reviews and merges.

## Position on the map

You don't set X / Y coordinates. The map derives them from the lists you
provide in `departments` and `ml_types`.

**X axis: Scope of Business Exposure** (left to right), grouped into three
categories that the simplified view shows by default:

| Category | Departments |
|---|---|
| Technical Security | Infrastructure & Cloud Security · AppSec & Product Security |
| Process & System Security | Security Engineering & Data Security · Security Operations (SOC & Incident Response) |
| Organizational Security | GRC & TPRM · Business Resilience & Security Awareness |

**Y axis: Required Data Maturity** (bottom to top, data-lean to data-rich):

| ML / AI approach | Position |
|---|---|
| Reinforcement Learning | Bottom of the axis |
| Unsupervised Learning | |
| Semi-Supervised Learning | |
| Supervised Learning | |
| Large Language Models | Top of the axis |

The full rationale for the Y axis ordering lives on the
[Required Data Maturity](https://secureio-gmbh.github.io/ai-security-landscape/about/data-maturity/)
explainer page.

### Spanning multiple areas

List more than one entry in `departments` or `ml_types` if the topic
genuinely spans them. The map renders a box that stretches across the
listed areas. That is how hybrids and cross-cutting topics show up.

```yaml
departments:
  - "AppSec & Product Security"
  - "Security Engineering & Data Security"   # box spans two columns
ml_types:
  - "Semi-Supervised Learning"
  - "Reinforcement Learning"                  # box stretches across two rows
```

The box colour is taken from the **first** department listed. That same
colour shows up as the accent border on every card that references the
topic across the site, so pick the department that best represents the
topic's home.

## Key factors (0 to 10, 10 is always good)

| Factor | What 10 means | What 0 means |
|---|---|---|
| `business_impact` | High strategic value if it works | Marginal value |
| `cost_efficiency` | Very affordable (setup + ongoing) | Expensive |
| `implementation_ease` | Drop-in for typical infrastructure | Months of integration work |
| `capability_fit` | AI/ML is a great fit for this task | Poor fit, low expected quality |
| `data_readiness` | Required data is already available and clean | Major data preparation needed |

These scores are subjective by design. Try to be realistic, not optimistic.

### Factor notes (optional but encouraged)

Add short keyword-style notes per factor that explain *why* you scored it
that way. Prefix each note with `+ ` (pro) or `- ` (con). Keep them
keyword-sized, not sentences. Even moderately high scores benefit from a
single con-note that names a real downside; pro-only lists tend to look
uncritical. A perfect 9 or 10 can be pro-only if there genuinely is no
meaningful con.

```yaml
factor_notes:
  cost_efficiency:
    - "- High token usage for large architectures"
    - "+ No hosting cost if you bring your own LLM"
  capability_fit:
    - "+ Structured output ready"
    - "+ Strong code understanding"
```

The site renders these as `+` / `−` bullets under each factor bar. Readers
see the trade-offs at a glance instead of guessing what is behind the
number.

## Hero image (optional)

If you have a diagram, architecture sketch, or other illustration that
helps explain the topic, place it under
`src/assets/topics/<slug>/<filename>.png` and reference it from the
frontmatter:

```yaml
hero_image:
  src: "../../assets/topics/your-topic-slug/diagram.png"
  alt: "Short description of what the image shows, for accessibility."
  caption: "Optional caption rendered under the image."
```

Astro optimises the image and converts it to WebP automatically during
build. PNG and JPEG sources both work; keep the original under ~1 MB if
possible. The schema treats `hero_image` as optional, so leaving it out
is fine.

If you build the image yourself, target roughly 1600 px wide and a 16:9
to 16:7 aspect ratio so it fits the topic page without forcing the reader
to scroll.

## References (Examples & References)

List representative open-source projects, papers, or vendor pages.
**Each reference needs a short description that ties it to this topic.**
A reader should click a link with a clear expectation of what they will
find there and why it matters for this use case.

```yaml
references:
  - url: "https://github.com/example/project"
    description: "What this project is, and how it represents the approach this topic describes."
```

If a topic has no public reference yet, leave the `references` block
empty rather than padding it with marginally relevant links.

## Connections

If your topic builds on or complements an existing one, add it to the
`connections` block. Each entry needs:

- `to`: the slug (filename without `.md`) of the related topic.
- `relation`: `before` / `after` / `parallel`. Defaults to `after`.
- `note`: one sentence describing the relationship. Focus on **what data
  or output is reused** between the two topics.

```yaml
connections:
  - to: "genai-threat-modeling"
    relation: before
    note: "Threat-model findings provide context for SAST prioritization."
```

### How temporal relations work

The `relation` describes the linked topic from **this topic's** perspective.

| You write | Meaning |
|---|---|
| `relation: after` (default) | The linked topic comes *after* this one. It consumes this topic's output. |
| `relation: before` | The linked topic comes *before* this one. Its output is your input. |
| `relation: parallel` | Related but with no temporal ordering. |

### The bidirectional rule

**You only declare a connection once.** If topic A says
`{to: B, relation: after}`, then:

- On A's page, B appears under **"After finishing"**.
- On B's page, A automatically appears under **"Where to start?"** with
  the same `note`. The site flips the relation for the reverse view.

So pick whichever topic feels like the natural place to declare the link.
The other side is generated for you. The `note` should make sense in both
directions, so focus on the data or output that is reused rather than on
who reuses it.

## Body structure

A minimal topic is fine. For longer topics, the reviewed topics on the
site use a numbered structure that works well:

```markdown
## 1. Concrete example
A worked-through scenario, ideally with named tools and a small story.

## 2. How it works (or "Why X matters")
The architectural picture or the reasoning behind the approach.

## 3. Use cases
A short list of concrete applications.

## 4. Notes & limitations
Honest caveats: known failure modes, sensitive dependencies, ongoing risks.

## 5. When this is the wrong tool
Anti-patterns. Helps readers self-select before investing time.
```

Sections are optional and orderable. Pick the depth that matches your
topic. A two-paragraph intro plus a Use Cases list is enough for many
topics; longer ones benefit from at least a *Concrete example* and a
*When this is the wrong tool* block.

## Style notes

- Keep titles short and human-readable.
- Prefer linking to an open-source project, paper, or vendor page in
  `url:`.
- Avoid marketing language. Be specific about what the AI/ML does and
  what data it needs.
- Prefer periods, colons, semicolons, or parentheses over em-dashes (`—`)
  in topic prose. Bindestriches in compound words (`MCP-based`,
  `low-risk`) are fine.
- Cross-link other topics with `[Topic Title](/topics/<slug>/)` the first
  time you mention them in the body. Don't link the same topic twice in
  the same body.

## Run the site locally (optional)

If you want to preview your change before opening a PR:

```bash
npm install
npm run dev
```

Then open <http://localhost:4321>.

## Questions

Open a GitHub issue. Friendly questions and "is X in scope?" discussions
are always welcome.
