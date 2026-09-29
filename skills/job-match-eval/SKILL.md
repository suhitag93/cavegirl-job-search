---
name: job-match-eval
description: Score a job description against your resume variants. Produces a keyword match score, recommends which resume to lead with, names the gaps, and drafts tailored replacement bullets and a summary. Also runs in watchlist mode across a list of target companies. Trigger on "score this JD", "match this job", "evaluate this role", "which resume should I use", "scan my watchlist", or a pasted job posting URL with intent to apply.
---

# Job Match Eval

Two modes.

**Single posting.** One job description in, one evaluation out.
**Watchlist.** Your target company list in, a ranked shortlist of roles you actually fit out.

## Step 0 — Load config

Read `config.yml` at the repo root. It gives you the owner, the resume
variants and their `lead_when` signals, the workspace paths, and the score
bands. Read every resume file listed under `resumes`. Do this on every run.

If `config.yml` is missing, tell the user to copy `config.example.yml` and stop.

## Hard rules

1. **Never invent experience.** Tailored bullets rephrase, reorder, reframe or
   re-emphasise what is already in the resume. You may not add a project, a
   metric, a tool or an outcome that is not there. If a rewrite needs a number
   the resume does not have, ask for it instead of estimating.
2. **No hyphens as connectors in drafted bullets.** Use spaces, "to",
   "through", "and", or rewrite the line. Hyphens inside an established term
   the posting itself uses are fine.
3. **Score conservatively.** Most real postings land between 55 and 75 against
   a real resume. If you are producing 85s routinely, you are matching on
   vocabulary rather than evidence. A high score you cannot defend line by line
   is worse than an honest 68.
4. **Say do not apply when that is the answer.** A reason to skip a posting is
   worth as much as a reason to apply when the user has four evenings a week.
5. **Show the source.** Put the posting URL at the top of every report.

## Mode A — Single posting

### 1. Get the posting

Accept a URL or pasted text.

- Try `mcp__nimble__*` web fetch, or the host's web fetch tool.
- If the page returns a shell with no body, which is common on Workday,
  Greenhouse embeds and anything behind a login, escalate to a browser tool
  (`mcp__claude-in-chrome__navigate` then `get_page_text`, or the built in
  browser equivalent).
- If neither works, ask for the text. Do not evaluate a posting you could not
  read.

Extract: company, role title, team, seniority, location and remote policy,
required years and degrees, required and preferred skills, domain signals, the
top five responsibilities by emphasis, and the comp range if listed.

### 2. Build the rubric

| Category | Weight |
|---|---|
| Hard requirements, the must haves | 30 |
| Core skills, tools, technologies | 35 |
| Domain and industry fit | 15 |
| Seniority and scope signals | 10 |
| Leadership and collaboration signals | 10 |

### 3. Score every resume variant

For each category grade 0 to 100:

- **Direct match** — the keyword appears verbatim or as a recognised synonym.
- **Adjacent match** — a related capability is present. Partial credit, and say
  what the adjacency is, for example the posting says Snowflake and the resume
  says BigQuery and Databricks.
- **Gap** — no evidence in the resume. Not partial credit. A gap.

Bands come from `config.yml`. Defaults: 85 and up strong, 70 to 84 solid,
55 to 69 stretch, below 55 weak.

### 4. Recommend a lane

Match the posting's emphasis against each variant's `lead_when` list in config.
When two variants land within five points, pick the one with the higher hard
requirements score and name the runner up. Give a one sentence reason.

### 5. Section by section

For the recommended variant, walk the resume and say what to change:

- **Tagline** — does it mirror the posting's own role vocabulary?
- **Summary** — propose targeted edits or a rewrite.
- **Experience** — per role, which bullets are highest leverage here, which are
  dead weight for this posting, which to rewrite.
- **Skills** — what to add, what to move to the front, what to cut for room.
- **Projects** — what to lead with, what to drop.

### 6. Draft the replacements

1. A revised tagline or summary, one to three sentences, in the posting's
   vocabulary.
2. Three to five rewritten bullets. Each one cites the source bullet it
   replaces, keeps any real metric, leads with the verb the posting would
   respond to, and obeys the no hyphen rule.
3. A skills block diff, ready to paste.

### 7. Write the report

Save to `{workspace.dir}/{workspace.evaluations_dir}/{YYYY-MM-DD}_{company}_{role-slug}.md`.
Create the directory if missing. Slugify: lowercase, spaces to underscores,
punctuation stripped.

In chat, give the short version only: company, role, recommendation, the
scores, the top three gaps, the honest verdict, and the file path. The file is
the deliverable. Do not dump the whole report into the conversation.

### 8. Hand off to the tracker

End every evaluation with the one line the tracker needs, so the loop closes:

    TRACKER: {company} | {role} | score {NN} | lane {id} | {apply or skip}

## Mode B — Watchlist

Triggered by "scan my watchlist", "what should I apply to this week", or a
request naming the target company list.

1. Read `target_companies_file` from config.
2. For each company, fetch the careers page and collect open roles. Filter to
   roles plausibly in your lanes before scoring, so you are not evaluating
   every warehouse opening at a company with one engineering post.
3. Run a light version of the rubric on each surviving role. Hard requirements
   and core skills only, which is enough to rank.
4. Return a ranked table: company, role, light score, recommended lane, and one
   line on why it made the list.
5. Offer to run the full single posting evaluation on the top three.

Run this weekly, not daily. Careers pages move slowly, and a daily scan trains
you to ignore the output.

## Report template

The saved file follows this shape. Fill every section. Cut a section only when
it genuinely does not apply, and say why.

    # Job Match Evaluation

    Company, role, posting URL, date evaluated, location and remote policy,
    comp range or "not disclosed".

    ## Recommendation
    Lead with, confidence, overall fit, and one paragraph of reasoning.

    ## Scores
    A row per resume variant with score and band, then the weighted breakdown
    table for the recommended one.

    ## Keyword coverage
    Matched direct, matched adjacent with the adjacency named, missing and
    weak with critical ones marked.

    ## Section by section
    Tagline, summary, experience per role, skills, projects.

    ## Tailored drafts
    New tagline, new summary, rewritten bullets each citing their source,
    skills diff.

    ## Risks and caveats
    Posting ambiguity, years mismatch, location problems, anything that reads
    as a red flag in how the posting is written.

## Invocation examples

- `job-match-eval https://jobs.example.com/abc123`
- "score this JD, which resume should I use"
- "scan my watchlist"
