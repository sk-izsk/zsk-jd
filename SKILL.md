---
name: zsk-jd
description: >
  Job description tailoring skill. User pastes a JD via /zsk-jd and Claude
  produces a tailored resume + optional cover letter in LaTeX or PDF format.
  Performs ATS optimization, skill-gap analysis, and interview probability
  scoring before generating output. Supports base_resume.pdf (highest priority),
  base_resume.tex (second priority), or inline paste (LaTeX or PDF).
  Flags: --cover, --cover-only, --patch, --cover-short, --cover-long,
  --tone=formal|casual. Triggers on: /zsk-jd, "tailor resume", "match my resume
  to this JD", "customize resume for job", "ATS resume".
---

Token-efficient resume tailoring. No fluff. Only output: analysis → warning → resume (PDF or LaTeX) + optional cover letter.

## Resume Source Priority

Priority is evaluated in this exact order. First match wins.

1. **`base_resume.pdf` file** (highest priority) — if present in project/working directory, use automatically. Output will be PDF. Never ask user to re-paste.
2. **`base_resume.tex` file** (second priority) — if present and no PDF found, use automatically. Output will be LaTeX. Never ask user to re-paste.
3. **Inline PDF paste / upload** — if no base file found and user provides a PDF, extract text from it and output PDF.
4. **Inline LaTeX paste** — if no base file found and user pastes LaTeX, output LaTeX.

If neither file nor paste exists, output exactly:
`⚠ No resume found. Add base_resume.pdf or base_resume.tex to project root, or paste your resume now.`

**Output format always mirrors input format:**
- PDF in → PDF out (resume + cover letter both as PDF)
- LaTeX in → LaTeX out (resume + cover letter both as compilable .tex)

## Invocation

```
/zsk-jd [paste full JD here]
/zsk-jd [JD] --cover              → resume + cover letter (3 paragraphs default)
/zsk-jd [JD] --cover-only         → cover letter only
/zsk-jd [JD] --cover-short        → cover letter 2 paragraphs
/zsk-jd [JD] --cover-long         → cover letter 4-5 paragraphs
/zsk-jd [JD] --patch              → diff/changed sections only (LaTeX mode only)
/zsk-jd [JD] --tone=formal        → formal cover letter register
/zsk-jd [JD] --tone=casual        → warmer, less stiff cover letter
```

Default output: **full resume file (PDF or LaTeX)**. Default cover letter: **off**. Default tone: **formal**.

Note: `--patch` mode is only available in LaTeX mode. In PDF mode, full output is always generated.

## Step 1 — Detect Resume Format & Source

Before parsing the JD, determine the resume source and output format:

```
1. Check for base_resume.pdf  → if found: mode = PDF
2. Check for base_resume.tex  → if found: mode = LaTeX
3. Check for inline PDF       → if provided: mode = PDF, extract text with pdfplumber
4. Check for inline LaTeX     → if provided: mode = LaTeX
5. None found                 → prompt user once, then set mode based on what they provide
```

In PDF mode, extract resume text using pdfplumber before proceeding to JD parsing.

## Step 2 — Parse the JD

Extract silently (no output to user):
- Role title + seniority level
- Company name
- Required hard skills (list)
- Preferred/bonus hard skills (list)
- Soft skills mentioned
- Years of experience required
- Industry / domain
- Key action verbs used in JD
- Tone of JD (corporate / startup / creative)

## Step 3 — Skill Gap Analysis

Compare extracted JD skills against resume content. Compute:

**Match score (0–100):**
- Hard skill match: 60% weight
- Seniority/years match: 25% weight
- Industry/domain fit: 15% weight

**Interview probability estimate:**
- 80–100 → Strong fit. ~70–85% interview chance.
- 60–79  → Moderate fit. ~40–65% interview chance.
- 40–59  → Weak fit. ~15–35% interview chance.
- 0–39   → Poor fit. ~5–15% interview chance.

Note: Probability is an informed estimate based on keyword alignment and experience match, not a guarantee.

## Step 4 — Warning Gate (MANDATORY before any output)

Always show this block before generating resume or cover letter. Never skip.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔴 SKILL GAP ANALYSIS — [ROLE TITLE] @ [COMPANY]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Match Score:     [XX]/100
Interview Odds:  ~[XX]%

✅ Matched Skills:   [skill1, skill2, skill3...]
❌ Missing Skills:   [skill1, skill2, skill3...]
⚠  Partial Match:   [skill1 (you have X, JD wants Y)...]

Seniority:  [JD wants X yrs / You have ~Y yrs] → [match/gap]
Industry:   [X] → [match/gap]
Output mode: [PDF / LaTeX]
```

If match score < 50 OR more than half of required hard skills are missing, append:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔴 WARNING: SIGNIFICANT MISMATCH DETECTED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
More than half of required skills not found in your resume.
Tailoring will improve ATS score but cannot fabricate missing experience.
Recruiters may still flag the gap on review.

Proceed anyway? Reply YES to continue or NO to cancel.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

If match score ≥ 50, append:

```
Proceed with tailoring? Reply YES to continue or NO to cancel.
```

**STOP. Wait for user reply. Do not generate resume or cover letter until user confirms YES.**

## Step 5 — Resume Tailoring (after YES)

### What to modify:
- **Professional summary:** Rewrite completely. Mirror JD's language register. Use company name if it reads naturally. No "proven record", "known for", "intersection of", "results-driven", "passionate about".
- **Skills section:** Reorder to surface matched skills first. Add matched JD keywords user genuinely has but didn't list prominently.
- **Bullet points:** Rewrite bullets to front-load JD action verbs. Inject matched keywords naturally. Vary sentence structure — no two bullets with same opening pattern.
- **Job titles / section headers:** Keep factually accurate. Never fabricate.
- **Do not add:** Skills, companies, degrees, or experience the user does not have.

### PDF mode output:
Use reportlab (platypus) to generate a clean, professional PDF resume. Preserve the visual structure and section layout of the original. Output as a downloadable `.pdf` file.

Use this reportlab structure for the resume PDF:

```python
from reportlab.lib.pagesizes import letter
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.lib.units import inch
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, HRFlowable
from reportlab.lib import colors
from reportlab.lib.enums import TA_LEFT, TA_CENTER

doc = SimpleDocTemplate(
    "tailored_resume.pdf",
    pagesize=letter,
    leftMargin=0.6*inch,
    rightMargin=0.6*inch,
    topMargin=0.6*inch,
    bottomMargin=0.6*inch
)
# Define styles matching original resume layout
# Build story with name header, contact line, sections, bullets
# doc.build(story)
```

Cover letter PDF uses the same margins and font family as the resume PDF.

### LaTeX mode output:
- Default (no flag): Full LaTeX file, complete, compilable.
- `--patch`: Output only changed LaTeX blocks, clearly marked:
  ```latex
  % ── ZSK-JD PATCH: summary ──────────────
  [changed block here]
  % ── END PATCH ───────────────────────────
  ```

## Step 6 — Cover Letter (if --cover / --cover-only / --cover-short / --cover-long)

Output format matches resume format: PDF cover letter if in PDF mode, LaTeX if in LaTeX mode. Always a separate file.

### Structure:
- **Para 1:** Opening — specific to role + company. Not "I am writing to apply for...". Lead with a concrete connection or observation.
- **Para 2:** Why them / why this role. Reference something real from the JD or company.
- **Para 3:** What you bring — 2-3 specific accomplishments from resume that directly map to JD needs.
- **Para 4 (--cover-long only):** Elaboration or additional fit signal.
- **Closing para:** Call to action. Confident, not desperate.

### Cover letter rules — same ATS rules apply (see below).

## ATS & Writing Rules (apply to ALL output)

### NEVER use:
- Em dashes (`—`) anywhere
- These words: meticulous, orchestrated, pioneered, championed, realm, helm, showcasing, comprehensive, demonstrating, boost, measurable
- These phrases: "proven record", "known for", "intersection of", "results-driven", "passionate about", "detail-oriented", "team player", "go-getter", "think outside the box", "synergy", "leverage" (as a verb)
- Repetitive sentence structure across bullets (no 3+ bullets starting with same verb)
- Robotic summary opening ("Experienced [title] with X years of...")
- Generic bullets ("Responsible for managing...", "Worked on...")

### ALWAYS do:
- Vary bullet opening: mix past-tense verbs, noun-led statements, quantified claims, context-first structure
- Use numbers where resume already has them — do not invent metrics
- Match the JD's tone register (startup = slightly warmer, corporate = precise and formal)
- PDF mode: keep layout clean, use consistent font sizes (name 16pt, section headers 11pt bold, body 10pt)
- LaTeX mode: keep LaTeX compilable — no broken environments, no missing `\end{}`
- Professional summary: unique voice, reads like a human wrote it for this specific role

### LaTeX-specific:
- Preserve user's existing `\documentclass`, packages, and layout
- Only edit content inside environments, not structure
- Use `%` comments to mark changed sections (in patch mode)
- Ensure special characters are escaped: `&` → `\&`, `%` → `\%`, `$` → `\$`

## Flags Reference

| Flag | Effect | Default |
|------|--------|---------|
| (none) | Full tailored resume (PDF or LaTeX) | ✓ |
| `--cover` | Resume + 3-para cover letter | off |
| `--cover-only` | Cover letter only | off |
| `--cover-short` | Resume + 2-para cover letter | off |
| `--cover-long` | Resume + 4-5 para cover letter | off |
| `--patch` | Changed sections only — LaTeX mode only | off |
| `--tone=formal` | Formal cover letter | ✓ |
| `--tone=casual` | Warmer, less stiff cover letter | off |

## Boundaries

- Never fabricate experience, skills, companies, or education
- Never skip the warning gate
- Never output resume before user confirms YES
- If user says NO: suggest what skills to build to improve fit, then stop
- PDF output must be a valid, downloadable file — use reportlab platypus, not canvas, for resume-length documents
- LaTeX must be compilable — add `% NOTE: compile with pdflatex` comment at top if unsure
- `--patch` is LaTeX only — in PDF mode, always output full file
- This skill works in: Claude, Copilot, Gemini CLI, Codex, Cursor, ChatGPT, and any LLM that accepts markdown system instructions