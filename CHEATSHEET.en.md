# 🎞️ PPT Master — Quick Guide (English)

> Commands verified against [hugohe3/ppt-master](https://github.com/hugohe3/ppt-master) README and Getting Started docs (2026-10-12).
> What it is: drop in a PDF, DOCX, web page or pasted text — an AI agent turns it into a **native PowerPoint deck**: real shapes, text boxes, charts and tables you can click and keep editing. Not slide images.

---

## 0. Prerequisite

Only **Python 3.10+**. Everything else arrives with `pip install -r requirements.txt`.

Pandoc is an edge case: only for legacy formats (`.doc`, `.odt`, `.rtf`, `.tex`, `.rst`, `.org`, `.typ`). `.docx`, `.html`, `.epub`, `.ipynb` are handled natively.

## 1. Install (pick one)

**A — Git clone (recommended):**

```bash
git clone https://github.com/hugohe3/ppt-master.git
cd ppt-master
pip install -r requirements.txt
```

**B — ZIP:** GitHub page → **Code → Download ZIP** → unzip → `pip install -r requirements.txt`. No Git history, so no `git pull` updates. The Releases page also offers a smaller skill-only package (~56 MB, no bundled example decks).

**C — Skill marketplace:**

```bash
# Cross-agent CLI (Claude Code, Cursor, Codex, …)
npx skills add hugohe3/ppt-master

# Or inside Claude Code
/plugin marketplace add hugohe3/ppt-master
/plugin install ppt-master@ppt-master
```

Marketplace installs fetch only the skill files — you still run `pip install -r requirements.txt` from the installed location.

## 2. Open it in an agent

PPT Master is a workflow ("skill") that runs inside any agent-capable AI tool. You only use its chat panel — no coding.

| Type | Examples |
|---|---|
| IDE-native agent | VS Code, Cursor, Trae, Windsurf, Zed |
| IDE plugin / extension | Claude Code (VS Code / JetBrains), GitHub Copilot, Cline |
| CLI agent | Claude Code CLI, Codex CLI, Gemini CLI |

Author's pick: **Claude Code**. Point the tool at the project folder: **File → Open Folder** in an IDE, or `cd ppt-master` in a CLI agent. Upstream recommends a large-context model (Kimi K3 or Claude, ~1M tokens) plus AI image generation for best results.

## 3. Create a deck — the 3-step loop

1. **Drop your source** into `projects/` (PDF, DOCX, images, Markdown, a URL) — or paste text straight into the chat.
2. **Tell the AI what to build:**

```
Please create a PPT from projects/q3-report/sources/report.pdf
```

```
Please turn the following into a PPT: [paste your content]
```

3. **Get the deck** at `exports/<name>_<timestamp>.pptx` — native DrawingML, editable in PowerPoint, Keynote, WPS or LibreOffice.

By default the AI first confirms a design spec (template, format — PPT 16:9 — page count). To skip that round trip, say so:

```
Quickly generate a 5-page deck from projects/q3-report/sources/report.pdf — no need to confirm with me
```

Charts and tables export as editable shapes by default; add `--native-charts-and-tables` to get data-backed PowerPoint Chart/Table objects with **Edit Data** (saved as a separate `_native_charts_tables.pptx`).

## 4. Templates — two different routes

Default is **free design**; no template needed. Reach for one only to reuse a fixed layout set or a brand identity.

| You want… | Route | What happens |
|---|---|---|
| This exact deck, new content | **Template fill** | Picks the pages that fit (one page can serve several output slides), writes text / table / chart data straight into the original file. Design, layouts, images, animations preserved. No new pages, no layout changes, no image swaps. |
| A new deck in this deck's style | **create-template** | Parses the `.pptx` into a reusable bundle, then generates a fresh deck through the SVG pipeline — new structure, any page count. |

**Template fill** — hand the AI the deck plus your material and say it plainly:

```
Fill this deck with the new content: [deck.pptx + your material]
```

**create-template** — must be triggered explicitly; the AI will not switch on its own:

```
You: Replicate this as a template via /create-template: projects/brand/our_deck.pptx
```

That builds a bundle (layout SVGs + `design_spec.md` + theme colors, fonts, images). A template lives in the skill library at `skills/ppt-master/templates/layouts/<id>/` (register it with `register_template.py`) or inside one project at `projects/<project>/templates/`. **Invoke it by directory path** in chat — a bare name never triggers it:

```
You: Make a deck from sources/report.pdf with template skills/ppt-master/templates/layouts/academic_defense/
```

Template fill also has a direct CLI route:

```bash
python3 scripts/template_fill_pptx.py analyze <project>/sources/<source.pptx> -o <project>/analysis/slide_library.json
python3 scripts/template_fill_pptx.py scaffold <project>/analysis/slide_library.json -o <project>/analysis/fill_plan.json --slides "1,3,4"
python3 scripts/template_fill_pptx.py check-plan <project>/analysis/slide_library.json <project>/analysis/fill_plan.json -o <project>/analysis/check_report.json
python3 scripts/template_fill_pptx.py apply <project>/sources/<source.pptx> <project>/analysis/fill_plan.json -o <project>/exports/filled.pptx
```

## 5. Images (optional)

Two paths, mixable per image: **AI generation** (the host agent's image tool, or `image_gen.py` with a provider key) and **web search** (`image_search.py`). Web search works with zero config (Openverse / Wikimedia Commons); free `PEXELS_API_KEY` / `PIXABAY_API_KEY` in `.env` add higher-quality stock sources. Licensing is handled automatically, with an inline credit added when a license requires it.

## 6. Update

```bash
# Git clone installs
python3 skills/ppt-master/scripts/update_repo.py
```

ZIP installs: download the new ZIP, unzip to a fresh folder, copy your old `.env` and `projects/` over, rerun `pip install -r requirements.txt`.

## 7. Cost

Free and open source (MIT). Upstream states the only cost is your own AI model usage — no PPT subscription on top. The pipeline runs on your machine; apart from talking to your AI model, your data stays local.

## 8. Stuck?

- AI lost context → ask it to read `skills/ppt-master/SKILL.md`.
- Layout, export or model problems → the upstream [FAQ](https://github.com/hugohe3/ppt-master/blob/main/docs/faq.md).

---

*Full guide: [Getting Started](https://github.com/hugohe3/ppt-master/blob/main/docs/getting-started.md) · Repo: https://github.com/hugohe3/ppt-master*
