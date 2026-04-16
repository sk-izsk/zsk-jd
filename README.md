# zsk-jd — Job Description Resume Tailoring Skill

> Paste a job description. Get a tailored, ATS-optimized resume and cover letter — in **PDF or LaTeX**. With skill-gap warnings and interview probability scoring. Works on any LLM.

---

## What It Does

1. You paste a job description using `/zsk-jd`
2. Claude detects your resume format (PDF or LaTeX) and sets output mode accordingly
3. Claude parses the JD — extracts required skills, seniority, keywords, tone
4. Compares against your resume — shows match score + interview probability
5. Warns you in red if the fit is weak **before** generating anything
6. On your confirmation, outputs a fully tailored resume + optional cover letter
7. All output follows strict ATS rules — no banned words, no robotic phrasing, varied bullet structure

---

## Two Ways to Use This Skill

### Way 1 — PDF Mode

Your resume lives as `base_resume.pdf`. The skill reads it, tailors it, and outputs a clean PDF resume (and PDF cover letter if requested). No LaTeX knowledge required.

**Best for:** Anyone who doesn't work with LaTeX, prefers a ready-to-send PDF, or wants the simplest setup.

```
your-project/
├── base_resume.pdf    ← skill reads and outputs PDF automatically
└── SKILL.md
```

### Way 2 — LaTeX Mode

Your resume lives as `base_resume.tex`. The skill tailors the LaTeX source directly and outputs a compilable `.tex` file. Full control over formatting, surgical edits only, and `--patch` mode available.

**Best for:** Developers who maintain a LaTeX resume, want version-controlled diffs, or need maximum token efficiency.

```
your-project/
├── base_resume.tex    ← skill reads and outputs LaTeX automatically
└── SKILL.md
```

### Which One Should You Use?

| | PDF Mode | LaTeX Mode |
|---|---|---|
| Setup effort | Low — just place your PDF | Medium — requires LaTeX resume |
| Output format | Ready-to-send PDF | Compilable .tex (needs pdflatex/Overleaf) |
| Token efficiency | Moderate | **Best** (especially with --patch) |
| Patch/diff support | No — full output always | Yes — `--patch` outputs changed blocks only |
| Cover letter output | PDF | LaTeX |
| Best for | Non-developers, quick turnaround | Developers, version control, token savings |

**Recommendation:** If you have a LaTeX resume, use LaTeX mode — it is significantly more token-efficient and gives you patch diffs. If you don't, PDF mode is the easier path with zero setup overhead.

---

## Priority Order

If multiple resume sources exist, the skill uses this priority. **First match wins.**

```
1. base_resume.pdf   ← highest priority → output: PDF
2. base_resume.tex   ← second priority  → output: LaTeX
3. Inline PDF paste  ← fallback         → output: PDF
4. Inline LaTeX paste← fallback         → output: LaTeX
```

If both `base_resume.pdf` and `base_resume.tex` exist in the same directory, **PDF wins**.

Output format always mirrors input format — PDF in, PDF out. LaTeX in, LaTeX out.

---

## Token Usage Comparison

### LaTeX vs PDF as input format

| Factor | PDF input | LaTeX input |
|--------|-----------|-------------|
| Extraction overhead | High — LLM must parse visual layout | None — already structured markup |
| Formatting ambiguity | Claude must infer structure from text | Explicit in environments and commands |
| Edit precision | Full rewrite required | Surgical section edits only |
| Patch mode available | No | Yes — `--patch` outputs diffs |
| Re-use across sessions | Must re-read file each session | File persists, zero re-read cost |
| Output accuracy | Layout can drift between reads | Compilable, deterministic output |

### Token cost estimates by approach

| Approach | Input tokens | Output tokens | Total (est.) |
|----------|-------------|---------------|--------------|
| Naive: "Rewrite my resume for this JD" (paste PDF text) | ~2,000–4,000 | ~1,500–3,000 | ~5,500 avg |
| zsk-jd PDF mode with `base_resume.pdf` | ~1,500–2,500 | ~1,200–2,000 | ~3,200 avg |
| zsk-jd LaTeX mode with `base_resume.tex` | ~1,200–2,000 | ~800–1,500 | ~2,500 avg |
| zsk-jd LaTeX mode `--patch` | ~600–900 | ~300–600 | ~1,100 avg |
| zsk-jd `--cover-only` (either mode) | ~600–900 | ~400–700 | ~1,200 avg |

**Bottom line:**

- LaTeX `--patch` mode is **4–5x cheaper** than a naive paste-and-ask approach
- LaTeX full mode is **~2x cheaper** than PDF mode for the same output quality
- PDF mode is still **~2x cheaper** than naive pasting with no skill
- The warning gate eliminates back-and-forth clarification rounds that waste tokens

---

## Install

### Option 1 — npx (fastest)

```bash
npx skills add https://github.com/sk-izsk/zsk-jd
```

Copies `SKILL.md` and `base_resume.tex` template into current directory.

### Option 2 — Manual (zip)

1. Download ZIP from [github.com/sk-izsk/zsk-jd](https://github.com/sk-izsk/zsk-jd)
2. Extract → copy `SKILL.md` into your project
3. Add your resume as `base_resume.pdf` or `base_resume.tex`

### Load into your AI tool

| Tool | How |
|------|-----|
| **Cursor** | Add `SKILL.md` content to `.cursorrules` |
| **Claude** | Paste `SKILL.md` at start of conversation |
| **Copilot** | Add to `.github/copilot-instructions.md` |
| **Gemini CLI** | `--system` flag with `SKILL.md` content |
| **ChatGPT** | Paste into Custom Instructions |

---

## Compatibility

Works on any LLM that accepts markdown system instructions:

| Platform | Supported |
|----------|-----------|
| Claude (claude.ai) | ✅ |
| GitHub Copilot | ✅ |
| Gemini CLI | ✅ |
| OpenAI Codex | ✅ |
| Cursor | ✅ |
| ChatGPT (custom instructions) | ✅ |
| Any OpenAI-compatible API | ✅ |

---

## Setup — Resume Sources in Detail

### PDF Mode: `base_resume.pdf` (Recommended for non-developers)

Place your PDF resume in your project root:

```
your-project/
├── base_resume.pdf       ← skill reads this automatically
└── SKILL.md
```

The skill detects it, extracts the content, tailors it, and outputs a clean PDF. You never paste anything. Cover letters are also output as PDF.

**Don't have a PDF resume?**
Export from Word, Google Docs, or any editor. Any standard single-column resume PDF works.

### LaTeX Mode: `base_resume.tex` (Recommended for developers)

Place your LaTeX resume in your project root:

```
your-project/
├── base_resume.tex       ← skill reads this automatically
└── SKILL.md
```

The skill performs surgical edits — only rewrites the summary, reorders skills, and adjusts bullet points. Everything else stays untouched. Use `--patch` to get only the changed blocks as a diff.

**Don't have a LaTeX resume yet?**

Option 1 — Overleaf (easiest):

1. Go to [overleaf.com](https://www.overleaf.com)
2. Create free account → New Project → choose a resume template
3. Edit your details → Download as `.tex` → save as `base_resume.tex`

Option 2 — Ask Claude:

```
Convert my resume into LaTeX. Here is my resume in plain text: [paste]
```

Save the output as `base_resume.tex`.

---

## Usage

### Basic — resume only

```
/zsk-jd [paste job description here]
```

### Resume + cover letter

```
/zsk-jd [JD] --cover
```

### Cover letter only

```
/zsk-jd [JD] --cover-only
```

### Patch mode — changed sections only (LaTeX mode only)

```
/zsk-jd [JD] --patch
```

### All flags

| Flag | What it does | Default |
|------|-------------|---------|
| *(none)* | Full tailored resume (PDF or LaTeX) | ✓ |
| `--cover` | Resume + 3-paragraph cover letter | off |
| `--cover-only` | Cover letter only | off |
| `--cover-short` | Resume + 2-paragraph cover letter | off |
| `--cover-long` | Resume + 4-5 paragraph cover letter | off |
| `--patch` | Changed sections only — LaTeX mode only | off |
| `--tone=formal` | Formal cover letter tone | ✓ |
| `--tone=casual` | Warmer, less stiff tone | off |

---

## Warning System

Before generating anything, the skill always shows a gap analysis:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔴 SKILL GAP ANALYSIS — Senior Engineer @ Acme Corp
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Match Score:     62/100
Interview Odds:  ~45%

✅ Matched:   React, TypeScript, Node.js, CI/CD, PostgreSQL
❌ Missing:   Kubernetes, Go, gRPC
⚠  Partial:   AWS (you have EC2/S3, JD wants full infra exp.)

Seniority:  JD wants 7+ yrs / You have ~5 yrs  →  slight gap
Industry:   FinTech  →  match
Output mode: PDF

Proceed with tailoring? Reply YES to continue or NO to cancel.
```

If more than half of required skills are missing or match score is below 50, you get a stronger red warning. If you reply **NO**, the skill tells you exactly what to build to become a stronger candidate.

---

## ATS Rules Enforced

**Never uses:**

- Em dashes
- Banned words: meticulous, orchestrated, pioneered, championed, realm, helm, showcasing, comprehensive, demonstrating, boost, measurable
- Robotic phrases: "proven record", "known for", "intersection of", "results-driven", "passionate about"
- Repetitive bullet structures (no 3+ bullets opening the same way)
- Generic bullets ("Responsible for...", "Worked on...")

**Always does:**

- Varies bullet sentence structure across all bullet points
- Uses the JD's own vocabulary naturally
- Writes professional summary in a unique, human voice specific to the role
- Matches the tone register of the JD (startup vs corporate)
- Keeps output compilable or valid — no broken LaTeX environments, no malformed PDF

---

## Honest Limitations

- The skill cannot fabricate skills or experience you don't have — it only surfaces and repositions what's real
- Interview probability is an estimate based on keyword and seniority alignment, not a recruiter's actual decision
- PDF output uses reportlab — visual fidelity matches the original resume's structure but font rendering may differ slightly from the source
- LaTeX output must be compiled — use Overleaf, pdflatex, or XeLaTeX locally
- `--patch` is only available in LaTeX mode
- Very niche or unusual LaTeX resume packages may need minor manual adjustments after output

---

## File Structure

```
your-project/
├── base_resume.pdf        ← PDF resume (highest priority — use this OR .tex)
├── base_resume.tex        ← LaTeX resume (second priority)
├── SKILL.md               ← the skill instructions (load into your AI tool)
└── README.md              ← this file
```

---

## Install into Your AI Tool

| Tool | How to load |
|------|-------------|
| **Cursor** | Add `SKILL.md` content to `.cursorrules` or system prompt |
| **Claude (claude.ai)** | Paste SKILL.md into conversation at start of session |
| **GitHub Copilot** | Add to `.github/copilot-instructions.md` |
| **Gemini CLI** | Pass as system prompt via `--system` flag |
| **Codex** | Include in system message of API call |
| **ChatGPT** | Paste into Custom Instructions → "What would you like ChatGPT to know?" |

---

## License

MIT — free to use, modify, and distribute.

*Made for job seekers who want real ATS results without token waste.*
