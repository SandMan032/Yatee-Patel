# Blog Post Template & Style Guide

This file is a **reference template** for writing new posts on this blog (Hugo + Congo).
Hand it to any writing agent along with the post's raw content, and it will produce a
post that is consistent with everything already published.

It is **not** a live post — it lives at the repo root and is never built by Hugo.

---

## 1. How a post is stored (page bundles)

Every post is a **folder**, not a loose `.md` file. The folder holds the markdown *and*
its images together, so images can be referenced by plain filename.

```
content/<section>/<post-slug>/
├── index.md          ← the post (front matter + body)
├── feature.png       ← hero image, auto-detected (optional but recommended)
├── sample-diagram.png
└── screenshot-1.png
```

- `<section>` is one of: `deep-dives`, `learning-log`, `misc`.
- `<post-slug>` is kebab-case, derived from the title (e.g. `designing-a-resilient-execution-engine`).
- A file named **`feature.png`** (or `feature.jpg`) is automatically used as the hero/cover — you don't reference it in the body.
- Reference every other image by bare filename: `![alt](sample-diagram.png "caption")`.

---

## 2. Where files go — adding a post, a section, or a project

This is the part that's easy to get wrong. There are **three different "folders"** and
they are not the same thing.

### 2a. Add a new blog post to an existing section

The common case. Pick the right section (`deep-dives`, `learning-log`, or `misc`), then
create a **new folder inside it** named with the post's kebab-case slug, containing an
`index.md`:

```
content/learning-log/my-new-post-slug/
└── index.md          ← front matter (categories: ["Learning Log"]) + body
```

- The folder name becomes the URL slug.
- Put any images for the post in that same folder.
- Set `categories` to match the section (`["Learning Log"]` here).
- That's it — Hugo auto-lists it on the section page and the homepage. No index to edit.

### 2b. Create a whole new section (rare)

A section is a **top-level folder under `content/`** with its own `_index.md` (note the
leading underscore — that marks it as the section's landing page, not a post):

```
content/tutorials/                 ← new section folder
├── _index.md                      ← section landing page (title + intro blurb)
└── first-tutorial/index.md        ← posts live in sub-folders, as usual
```

The `_index.md` looks like this:

```yaml
---
title: "Tutorials"
description: "Short SEO description of the section."
---

An intro paragraph shown at the top of the section's list page.
```

> **Also add it to the nav menu.** A new section won't appear in the header until you add
> an entry in `config/_default/menus.toml` (copy an existing block, set `name`, `pageRef`
> to `/tutorials`, and a `weight`). Flag this step to the human — it's a config edit, not
> a content edit.

### 2c. Create a new project

Projects are a **taxonomy**, not a content section. Each project is a folder under
`content/projects/` with an `_index.md` (again, underscore = landing page) plus a
`feature.png` cover:

```
content/projects/my-new-project/    ← folder name = the project slug
├── _index.md
└── feature.png                     ← card cover image (needed for the project card)
```

The `_index.md`:

```yaml
---
title: "Human-Readable Project Name"
description: "Short SEO description."
summary: "1–2 sentence blurb shown on the project card and project page."
weight: 10                          ← lower numbers sort first on the projects grid
---

A paragraph introducing the project, shown at the top of its page above the list
of related posts.
```

**How posts join a project:** the folder name here (`my-new-project`) is the slug you put
in a post's `projects: [...]` front matter. Add `projects: ["my-new-project"]` to any post
and it automatically appears on that project's page — the link is bidirectional and
requires no other edits.

> **The slug must match exactly.** If a post lists `projects: ["my-new-project"]` but no
> `content/projects/my-new-project/` folder exists, the link silently goes nowhere. Create
> the project folder first, or use an existing slug.

---

## 3. Front matter (copy this block, fill every field)

```yaml
---
title: "Title in Title Case"
date: 2026-07-19
draft: false
description: "One-sentence SEO/meta summary. Shown in search results and link previews. ~150 chars, no markdown."
summary: "1–2 sentence summary shown on list and card views. Slightly longer/richer than description; sells the post."
tags: ["lowercase-kebab", "go", "systems-design"]
categories: ["Deep Dives"]
projects: ["algo-trading-bot"]
---
```

**Field rules:**

| Field | Required | Notes |
| --- | --- | --- |
| `title` | ✅ | Title Case, in quotes. This is the H1 — **do not** repeat it as an `#` heading in the body. |
| `date` | ✅ | `YYYY-MM-DD`. Publish date. |
| `draft` | ✅ | `true` while writing; `false` to publish. |
| `description` | ✅ | SEO meta only (never rendered on the page body). Plain text. |
| `summary` | ✅ | The card/list blurb. Make it compelling — it's what makes someone click. |
| `tags` | ✅ | 2–5 lowercase kebab-case topics. Reuse existing tags where possible. |
| `categories` | ✅ | Match the section: `["Deep Dives"]`, `["Learning Log"]`, or `["Misc"]`. |
| `projects` | optional | Slugs of projects this post belongs to (must match a folder in `content/projects/`). A post can join multiple projects. Omit if it belongs to none. |

Existing project slugs: `algo-trading-bot`, `this-blog`.

---

## 4. The opening (every post starts the same way)

Right after the front matter, open with a **`lead`** — a larger, emphasised intro
paragraph that frames the whole post in 1–3 sentences. This is the standard hook.

```markdown
{{< lead >}}
One or two sentences that state the core idea of the post and why it matters.
Punchy, no preamble.
{{< /lead >}}
```

> If the post is a placeholder/demo, put a `pencil` alert **above** the lead flagging it
> (see §5). Real posts skip that and go straight to the lead.

---

## 5. Body structure & voice

**Structure**
- Use `##` for main sections and `###` for sub-sections. Never use `#` (that's the title).
- Keep sections short and scannable; lead each with a plain-language sentence before diving into detail.
- Close with a short "where this goes next" / "takeaways" section, and optionally a `button` to a related section.

**Voice** (match the existing posts)
- First person, direct, engineer-to-engineer. Confident but not salesy.
- Explain *why*, not just *what*. Favour concrete rules of thumb over generalities.
- Short paragraphs. Bold the occasional **key phrase** for scanning.

---

## 6. Capabilities reference — use these building blocks

Everything below renders on this blog. Pull in whatever the content needs.

### Paragraphs & inline formatting
Standard markdown: **bold**, *italic*, `inline code`, [links](/projects/), and
footnotes[^1].

[^1]: Footnotes render at the bottom of the post.

### Headings
```markdown
## Main section
### Sub-section
```

### Lists
```markdown
- Bullet one
- Bullet two
  - Nested bullet

1. Ordered step
2. Ordered step

- [ ] Task list item (unchecked)
- [x] Task list item (done)
```

### Code blocks (with language for syntax highlighting + copy button)
Code copy is enabled site-wide, so always tag the language.

````markdown
```go
func main() {
    fmt.Println("hello")
}
```

```dockerfile
FROM golang:1.25 AS build
```

```bash
docker build -t app:"$(git rev-parse --short HEAD)" .
```
````

### Tables
```markdown
| Column | Column |
| --- | --- |
| cell | cell |
| cell | cell |
```

### Blockquotes
```markdown
> A quote or an aside. Good for a memorable one-liner or a caveat.
```

### Images (page-bundle — reference by filename)
Drop the image file in the post folder, then:
```markdown
![Descriptive alt text for accessibility](sample-diagram.png "Optional caption shown under the image")
```
- **Always** write meaningful alt text.
- The caption (the quoted string) is optional but nice for figures/charts.
- Do **not** reference `feature.png` in the body — it's the auto hero.

### Diagrams (Mermaid — rendered as SVG)
For flowcharts, sequence diagrams, architecture, etc. Prefer this over an image when
the diagram is structural.
```markdown
{{< mermaid >}}
flowchart LR
    A[Input] -->|data| B(Process)
    B --> C{Decision}
    C -->|yes| D[Output]
    C -.fallback.-> E[Handler]
{{< /mermaid >}}
```

### Callouts / alerts
Use for tips, warnings, key rules. Default style:
```markdown
{{< alert >}}
A neutral, important note.
{{< /alert >}}
```
With an icon (common icons: `lightbulb`, `check`, `pencil`, `fire`, `exclamation`):
```markdown
{{< alert icon="lightbulb" >}}
**Rule of thumb:** a short, memorable takeaway.
{{< /alert >}}
```
Coloured variant (used for emphasis boxes):
```markdown
{{< alert icon="lightbulb" cardColor="#1e3a5f" iconColor="#38bdd2" textColor="#e2e8f0" >}}
A highlighted insight in the blog's ocean-accent colours.
{{< /alert >}}
```
> The blog's accent colour is `#38bdd2` (ocean cyan). Reuse it for highlighted alerts to stay on-brand.

### Buttons (call-to-action link)
Usually at the end, linking to a related section:
```markdown
{{< button href="/learning-log/" target="_self" >}}
Read the Learning Log →
{{< /button >}}
```

---

## 7. Full skeleton (copy-paste starting point)

```markdown
---
title: "Your Title Here"
date: 2026-07-19
draft: false
description: "One-sentence SEO summary."
summary: "1–2 sentence card blurb that sells the post."
tags: ["tag-one", "tag-two"]
categories: ["Deep Dives"]
projects: ["algo-trading-bot"]
---

{{< lead >}}
The core idea of this post in one or two punchy sentences.
{{< /lead >}}

## The problem / context

Plain-language framing of what this is about and why it matters.

## The main idea

Explanation, with a diagram if it's structural:

{{< mermaid >}}
flowchart LR
    A[Start] --> B[Finish]
{{< /mermaid >}}

## Details

Prose, then a code example:

```go
// commented, minimal, illustrative
func example() {}
```

{{< alert icon="lightbulb" >}}
**Rule of thumb:** the one thing to remember from this section.
{{< /alert >}}

A comparison, when useful:

| Option | Trade-off |
| --- | --- |
| A | ... |
| B | ... |

![Alt text](diagram.png "Optional caption")

## Takeaways

1. First takeaway.
2. Second takeaway.

{{< button href="/projects/" target="_self" >}}
See the project →
{{< /button >}}
```

---

## 8. Checklist before publishing

- [ ] Folder is `content/<section>/<slug>/index.md`, slug is kebab-case.
- [ ] All six front-matter fields filled; `categories` matches the section.
- [ ] `projects` slugs exist under `content/projects/` (create the project folder first — see §2c — or omit the field).
- [ ] Opens with a `lead`; title is **not** repeated as an `#` heading.
- [ ] Every code block has a language tag.
- [ ] Every image lives in the post folder, has alt text, and isn't named `feature.*` unless it's the hero.
- [ ] `draft: false` when ready to ship.
```
