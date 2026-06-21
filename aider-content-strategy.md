# Content Strategy: Anti-Compression Contract + Aider Master Prompt

## Why this exists
LLMs default to summarizing because concise = "polished" in most training contexts.
For a reference site meant to support deep technical mastery, that's the opposite of
what you want. The fix isn't a better model — it's a structural contract that makes
a one-paragraph answer fail to satisfy the prompt.

## The Content Contract (reusable for every topic, every layer)

For EVERY individual topic (Forest, Domain, OU, Site, etc.), require these 8
subsections, each in full prose:

1. **Technical Definition** — precise, unambiguous
2. **Underlying Mechanism** — how AD/Windows actually implements this internally
   (replication mechanics, Kerberos/trust internals, schema/GC behavior, etc.)
3. **Why It Exists** — historical and architectural rationale
4. **Enterprise / Banking Reality** — how this is actually designed and operated
   at a large regulated financial enterprise: real design patterns, audit/compliance
   hooks, what a Tier-1 bank's architecture actually looks like for this concept
5. **Operational Considerations** — what breaks in practice, day-2 ops reality,
   relevant PowerShell/CLI examples, monitoring/alerting angle
6. **Common Misconceptions** — explicitly name and correct wrong mental models
7. **Interview Angle** — 3-5 questions pitched at VP/Principal architecture level
   (not junior trivia), each with a strong model answer
8. **Related Concepts** — cross-links to other topics on the site

Process rules that prevent compression in practice:
- Generate **one topic at a time**, never a whole page in one shot.
- If seed notes (yours or from another source) exist, the job is to **expand every
  point into full explanation** — never condense or remove what's already there.
- Once a section is approved, it's **locked** — no "cleanup" edits unless explicitly
  requested. Review every diff for unexpected shrinkage.

---

## Master Aider Prompt — paste this into Aider as the task prompt

```
You are helping me build a long-term technical reference site in MkDocs Material,
hosted on GitHub Pages. I am a senior Windows infrastructure engineer targeting a
VP-level IC architecture role at a large banking enterprise. This site is both my
deep interview-prep material and a reference I will return to throughout my career.

CRITICAL RULE: Do not summarize or compress technical content. Treat every page as
exhaustive reference documentation, not an overview. If you're tempted to write one
paragraph, write four. On this project, depth and completeness are explicitly more
valuable than brevity.

Target file: docs/identity/layer1-directory-trust/forests-domains-ous-sites.md

Cover these topics, each as its own `##` section:
- Forest
- Tree
- Domain
- OU
- Site
- Why Multiple Domains Existed Historically
- Modern Single-Domain Design Philosophy

For EACH topic section, include ALL of the following subsections as `###` headers,
written in full prose (not just bullet lists):

1. Technical Definition — precise, unambiguous
2. Underlying Mechanism — how Active Directory/Windows actually implements this
   internally (replication mechanics, Kerberos/trust mechanics, schema/GC behavior,
   SID filtering, etc., as applicable to the topic)
3. Why It Exists — historical and architectural rationale
4. Enterprise / Banking Reality — how this is actually designed and operated in a
   large regulated financial enterprise: real design patterns, audit and compliance
   considerations, what a Tier-1 bank's AD architecture looks like for this concept
5. Operational Considerations — what breaks, day-2 operations reality, relevant
   PowerShell/CLI examples, monitoring/alerting angle
6. Common Misconceptions — explicitly call out and correct wrong mental models
7. Interview Angle — 3-5 likely questions at a VP/Principal architecture level
   (not junior trivia), each with a strong model answer
8. Related Concepts — cross-links to other topics on this site

Formatting requirements:
- Use MkDocs Material admonitions (!!! note, !!! warning, !!! tip) for callouts
- Use Mermaid diagrams (```mermaid code blocks) for any topology/hierarchy/trust visuals
- Use tables when comparing things (e.g., OU vs Domain vs Forest as boundaries)
- Use fenced code blocks for any CLI/PowerShell examples

Process:
- Work through ONE topic section at a time. After drafting a section, stop and show
  me the diff before moving to the next topic.
- Do not shorten, remove, or "clean up" any previously written section unless I
  explicitly ask you to revise it.
- If I paste in raw seed notes, your job is to EXPAND every point into full
  explanation — never compress or drop information I've already given you.

Start with the Forest section now. Stop after that section for my review.
```

---

## How to use your existing ChatGPT draft

Don't throw it away — paste it into the target `.md` file first as raw seed content,
then add this instruction before starting Aider's run:

```
The file already contains a compressed draft I wrote earlier. Your job is to expand
it according to the 8-subsection contract above — keep every fact already present,
and add the missing depth (Underlying Mechanism, Enterprise/Banking Reality,
Operational Considerations, Interview Angle, etc.) that the draft is missing.
Do not shorten anything that's already there.
```

This anchors the floor at "at least as much as the ChatGPT version" instead of
giving the model a blank page to summarize onto.
