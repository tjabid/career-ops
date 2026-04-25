# Mode: apply — Live Application Assistant

Interactive mode for when the candidate is filling out an application form in Chrome. Reads what's on screen, loads prior offer context, and generates personalized answers for each form question.

## Requirements

- **Best with visible Playwright**: In visible mode, the candidate sees the browser and Claude can interact with the page.
- **Without Playwright**: The candidate shares a screenshot or pastes the questions manually.

## Workflow

```
1. DETECT    → Read active Chrome tab (screenshot/URL/title)
2. IDENTIFY  → Extract company + role from the page
3. SEARCH    → Match against existing reports in reports/
4. LOAD      → Read full report + Section G (if exists)
5. COMPARE   → Does the role on screen match the evaluated one? If changed → notify
6. ANALYZE   → Identify ALL visible form questions
7. GENERATE  → For each question, generate a personalized answer
8. PRESENT   → Show formatted answers ready for copy-paste
```

## Step 1 — Detect the offer

**With Playwright:** Take a snapshot of the active page. Read the title, URL, and visible content.

**Without Playwright:** Ask the candidate to:
- Share a screenshot of the form (Read tool can read images)
- Or paste the form questions as text
- Or say the company + role so we can search for it

## Step 2 — Identify and find context

1. Extract company name and role title from the page
2. Search `reports/` by company name (case-insensitive Grep)
3. If match found → load the full report
4. If Section G exists → load the prior draft answers as a base
5. If NO match → notify and offer to run a quick auto-pipeline

## Step 3 — Detect role changes

If the role on screen differs from the evaluated one:
- **Notify the candidate**: "The role has changed from [X] to [Y]. Do you want me to re-evaluate or adapt the answers to the new title?"
- **If adapt**: Adjust answers to the new role without re-evaluating
- **If re-evaluate**: Run full A-F evaluation, update the report, regenerate Section G
- **Update tracker**: Change the role title in applications.md if applicable

## Step 4 — Analyze form questions

Identify ALL visible questions:
- Free-text fields (cover letter, why this role, etc.)
- Dropdowns (how did you hear, work authorization, etc.)
- Yes/No (relocation, visa, etc.)
- Salary fields (range, expectation)
- Upload fields (resume, cover letter PDF)

Classify each question:
- **Already answered in Section G** → adapt the existing answer
- **New question** → generate answer from the report + cv.md

## Step 5 — Generate answers and write to report

For each question, generate the answer following:

1. **Report context**: Use proof points from Block B, STAR stories from Block F
2. **Prior Section H**: If already exists in the report, update rather than duplicate
3. **"I'm choosing you" tone**: Same framework as auto-pipeline
4. **Specificity**: Reference something concrete from the JD visible on screen
5. **career-ops proof point**: Include in "Additional info" if there's a field for it

**MANDATORY RULE — Always write to the report:**

ALWAYS write answers to the corresponding report as **Section H**, using the Edit/Write tool. Do NOT rely on the terminal as the only output — the terminal has formatting issues when copying.

Flow:
1. Generate all answers in memory
2. Find the report in `reports/` for this company+role
3. Append (or update if already exists) **Section H** at the end of the report:

```markdown
---

## H) Application Answers

**CV to upload:** `output/cv-{candidate}-{company}-{date}.pdf`

---

### Q1. [Exact form question]

[Answer ready for copy-paste, no blockquotes or terminal formatting]

---

### Q2. [Next question]

[Answer]

...
```

4. Confirm in terminal: `Answers written to reports/{report-file}.md — Section H` with the list of answered questions
5. Do NOT repeat the full content in the terminal — only the confirmation + any important notes

## Step 6 — Post-apply

If the candidate confirms they submitted the application:
1. Update status in `applications.md` from `Evaluated` to `Applied`
2. Answers are already in Section H of the report (written in Step 5)
3. Suggest next step: `/career-ops contacto` for LinkedIn outreach

## Scroll handling

If the form has more questions than are visible:
- Ask the candidate to scroll and share another screenshot
- Or paste the remaining questions
- Process in iterations until the full form is covered
