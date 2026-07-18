> From: https://yuanbao.tencent.com/chat/naQivTmsDa/0OgDkLBhV7Q

# you asked

message time: 2026-07-11 11:09:53

i have a qmd file, I am using `quarto::quarto_render` to generate a html report. how to add a navigation bar or side bar on the left hand side of the html report so that I can easily navigate different sections, headers and so on

---

# yuanbao response

For a single `.qmd` → HTML report (not a multi-page website), the simplest way is to turn on the **table of contents** and pin it to the **left** via YAML front matter. That gives you a floating left sidebar that auto-highlights as you scroll.

## Minimal YAML for your `.qmd`

```yaml
---
title: "My Report"
format:
  html:
    toc: true
    toc-location: left
    toc-depth: 3
    number-sections: true
---
```

- **`toc: true`** — generates the TOC from your `#`, `##`, `###` headers.
- **`toc-location: left`** — puts it on the left as a sidebar (default is `right`).
- **`toc-depth: 3`** — how many header levels to include (default 3: H1–H3).
- **`number-sections: true`** — optional, numbers your headers to match the TOC.

Other handy knobs:

| Option | What it does |
|---|---|
| `toc-title: "Contents"` | Custom TOC heading |
| `toc-expand: false` | Start fully collapsed; `true` = expand all |
| `toc-location: left-body` | TOC in left sidebar **plus** in-body (two copies) |

## If you want a wider sidebar

Default sidebar width is 250px. Bump it:

```yaml
format:
  html:
    toc: true
    toc-location: left
    grid:
      sidebar-width: 350px
```

## What if you actually want a "website-style" sidebar?

The TOC-left approach above only lists **this document's own headers**. If you want a sidebar with **arbitrary links to other pages/docs** (like a multi-page site nav), you'd need to convert the project to `type: website` with a `_quarto.yml`:

```yaml
# _quarto.yml
project:
  type: website
website:
  sidebar:
    style: docked
    contents:
      - text: "Report"
        href: index.qmd
      - text: "Appendix"
        href: appendix.qmd
format:
  html:
    toc: true
```

But for a **single report** where you just want to jump between `# Section 1`, `# Section 2`, etc. — the `toc-location: left` route is all you need. `quarto::quarto_render("my.qmd")` will pick it up automatically.