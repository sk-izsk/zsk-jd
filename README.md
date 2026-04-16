# zsk-jd — Job Description Resume Tailoring Skill

> Paste a job description. Get a tailored, ATS-optimized LaTeX resume and cover letter. With skill-gap warnings and interview probability scoring. Works on any LLM.

---

## What It Does

1. You paste a job description using `/zsk-jd`
2. Claude parses the JD — extracts required skills, seniority, keywords, tone
3. Compares against your resume — shows match score + interview probability
4. Warns you in red if the fit is weak **before** generating anything
5. On your confirmation, outputs a fully tailored LaTeX resume (and optional cover letter)
6. All output follows strict ATS rules — no banned words, no robotic phrasing, varied bullet structure

---

## Install

### Option 1 — npx (fastest)

```bash
npx zsk-jd init
```

Copies `SKILL.md` and `base_resume.tex` template into current directory.

### Option 2 — Manual (zip)

1. Download ZIP from [github.com/sk-izsk/zsk-jd](https://github.com/sk-izsk/zsk-jd)
2. Extract → copy `SKILL.md` into your project
3. Rename `base_resume.tex` to match your resume, or replace contents

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

## Setup — Two Ways to Provide Your Resume

### Option A: `base_resume.tex` file (Recommended — zero tokens per session)

Place your LaTeX resume in your project root:

```
your-project/
├── base_resume.tex       ← skill reads this automatically
└── .cursor/              (or wherever you store skills)
```

The skill detects and loads it automatically. **You never paste it again.** This is the priority source — if the file exists, it always wins over a pasted resume.

### Option B: Inline paste (fallback)

If no `base_resume.tex` exists, the skill will ask you once per session:

```
⚠ No resume found. Paste your LaTeX resume now.
```

Paste it once. It's stored for the rest of the session.

---

## Don't Have a LaTeX Resume Yet?

No problem. You have two easy paths:

**Option 1 — Overleaf (easiest)**

1. Go to [overleaf.com](https://www.overleaf.com)
2. Create free account → New Project → choose a resume template
3. Edit your details in the visual editor
4. Download as `.tex` → save as `base_resume.tex`

**Option 2 — Ask Claude**

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

### Patch mode — only changed sections (faster, fewer tokens)

```
/zsk-jd [JD] --patch
```

### All flags

| Flag | What it does | Default |
|------|-------------|---------|
| *(none)* | Full tailored LaTeX resume | ✓ |
| `--cover` | Resume + 3-paragraph cover letter | off |
| `--cover-only` | Cover letter only | off |
| `--cover-short` | Resume + 2-paragraph cover letter | off |
| `--cover-long` | Resume + 4-5 paragraph cover letter | off |
| `--patch` | Output changed sections only, not full file | off |
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

Proceed with tailoring? Reply YES to continue or NO to cancel.
```

If more than half of required skills are missing or match score is below 50, you get a stronger red warning with a recommendation to reconsider before proceeding.

If you reply **NO**, the skill tells you exactly what skills to build to become a stronger candidate for this role.

---

## Token Usage Comparison

This skill is designed to minimize token consumption vs. naive approaches.

| Approach | Input tokens | Output tokens | Total (est.) |
|----------|-------------|---------------|--------------|
| "Rewrite my resume for this job" (no skill, paste PDF text) | ~2,000-4,000 | ~1,500-3,000 | ~5,500 avg |
| zsk-jd with inline LaTeX paste (first use) | ~1,200-2,000 | ~800-1,500 | ~3,200 avg |
| zsk-jd with `base_resume.tex` (file loaded, not pasted) | ~600-900 | ~800-1,500 | ~1,700 avg |
| zsk-jd `--patch` mode | ~600-900 | ~300-600 | ~1,100 avg |
| zsk-jd `--cover-only` | ~600-900 | ~400-700 | ~1,200 avg |

**Bottom line:** Using `base_resume.tex` + `--patch` mode is roughly **4-5x cheaper** than a naive paste-and-ask approach. The skill also eliminates back-and-forth clarification rounds that waste tokens.

### Why LaTeX beats PDF for tokens

| Factor | PDF input | LaTeX input |
|--------|-----------|-------------|
| Extraction overhead | High — OCR/parse needed | None — already structured |
| Formatting ambiguity | Claude must guess layout | Explicit in markup |
| Edit precision | Full rewrite often needed | Surgical edits only |
| Re-use across sessions | Must paste every time | File persists |
| Output accuracy | Formatting can drift | Compilable output |

---

## ATS Rules Enforced

The skill automatically enforces these rules on every resume and cover letter it generates:

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
- Keeps LaTeX compilable — no broken environments

---

## Example Session

```
You:    /zsk-jd We are looking for a Senior Backend Engineer with 6+ years experience
        in Go, Kubernetes, and distributed systems. Experience with gRPC and AWS 
        required. FinTech background preferred. [...]

Claude: ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
        🔴 SKILL GAP ANALYSIS — Senior Backend Engineer @ FinCo
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
        Match Score:     48/100
        Interview Odds:  ~22%
        ✅ Matched:   AWS, distributed systems, PostgreSQL
        ❌ Missing:   Go, Kubernetes, gRPC
        Seniority:   JD wants 6 yrs / You have ~4 yrs  →  gap
        
        🔴 WARNING: SIGNIFICANT MISMATCH DETECTED
        More than half of required skills not found in your resume.
        Proceed anyway? Reply YES to continue or NO to cancel.

You:    NO

Claude: Understood. To be competitive for this role, focus on:
        - Go: build 1-2 projects, target 3-6 months
        - Kubernetes: CKA cert or deploy personal project on k8s
        - gRPC: add to an existing Go service
        Re-run /zsk-jd once you have these to show on your resume.
```

---

## Honest Limitations

- The skill cannot fabricate skills or experience you don't have — it only surfaces and repositions what's real
- Interview probability is an estimate based on keyword and seniority alignment, not a recruiter's actual decision
- LaTeX output must be compiled — use Overleaf, pdflatex, or XeLaTeX locally
- Very niche or unusual LaTeX resume packages may need minor manual adjustments after output

---

## File Structure

```
your-project/
├── base_resume.tex        ← your base resume (skill auto-loads this)
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
