---
name: job-application-tracker
description: Maintain a persistent job application pipeline from your inbox and meeting notes. Use whenever the user asks about application status, which companies they have applied to, interview invites, rejections, a dashboard or summary of their search, or the daily tracker update. Always trigger for any pipeline, application status, or interview progress question.
---

# Job Application Tracker

Maintains one tracker file that is updated over time rather than regenerated.
Two modes.

**Daily incremental**, the default. Scans a short recent window and merges.
**Full rebuild**. Scans the long window and recreates the file. Used on first
run, when the file is missing, or when explicitly asked for a full refresh.

## Step 0 — Load config

Read `config.yml`. It gives the workspace directory, the tracker file name, the
scan windows, and the ghost threshold. Read the `resumes` list too, because the
tracker records which variant was sent.

## Step 1 — Resolve the tracker file

1. Confirm filesystem access first, for example `mcp__Filesystem__list_allowed_directories`.
2. Look for the tracker at `{workspace.dir}/{workspace.tracker_file}`.
3. Found, read it and parse the rows plus the `Last scan` timestamp. Go incremental.
4. Not found, go full rebuild and create it at the end.

## Data sources

**Mail** — application confirmations, recruiter outreach, scheduling,
rejections, offers.
**Meeting notes** — what actually happened in an interview. This is where the
real signal lives, so notes drive the progress detail on active rows while mail
drives status changes.

## Daily incremental

### A. Scan mail

Mail search granularity is per day, so query two days and filter in process to
the configured window, defaulting to 36 hours. The window overlaps the previous
run on purpose so a late or skipped run misses nothing. Deduplication handles
the overlap.

Run these with `mcp__Gmail__search_threads`, page size 50, dedupe by thread id.

    subject:(application OR applied OR "thank you for applying" OR "we received your application") newer_than:2d
    subject:(interview OR "phone screen" OR "hiring manager" OR "next steps" OR "move forward") newer_than:2d
    subject:(unfortunately OR "not moving forward" OR "other candidates" OR "we regret") newer_than:2d
    subject:(offer OR "pleased to" OR "join our team") newer_than:2d

Subject lines alone miss a lot, because applicant tracking systems send generic
subjects from recognisable domains. Add a sender sweep:

    from:(greenhouse.io OR lever.co OR ashbyhq.com OR myworkday.com OR icims.com OR smartrecruiters.com OR jobvite.com OR workable.com) newer_than:2d

Drop threads whose newest message falls outside the window. Only call
`mcp__Gmail__get_thread` when the snippet cannot tell you company, role, status
or referral.

### B. Scan meeting notes

1. List meetings in the same window.
2. A meeting maps to an application when an attendee domain matches a company
   in the pipeline or the mail sweep, or the title names a pipeline company, or
   the title contains interview language: interview, screen, hiring manager,
   onsite, system design, technical, recruiter.
3. Pull summary and notes for mapped meetings. Pull the transcript only when
   exact wording matters.
4. Extract: which round, how it went, concerns or strengths flagged, and the
   next step.

### C. Merge

Apply the merge rules. Update `Last scan`. Write the file back.

### D. Report the delta

Show only what moved this run, then the refreshed table. If nothing moved, say
so plainly and leave the file untouched.

## Full rebuild

Same queries with `newer_than:{full_rebuild_days}d`. Pull meeting notes across
the same period. Build the pipeline, then write the file.

## Status ladder

Highest match wins.

1. **Offer** — offer language.
2. **Rejected** — rejection language.
3. **Withdrawn** — the user pulled out. Never inferred, only set when told.
4. **Interviewing** — a round is scheduled or has happened.
5. **Recruiter follow up** — a human replied and wants something from you, but
   nothing is scheduled yet. This is the row that rots if you do not watch it.
6. **Screening** — recruiter outreach or a screen requested.
7. **Applied** — confirmation only, no reply.
8. **Ghosted** — applied more than `ghost_after_days` ago with no reply.

A logged interview meeting is strong evidence. If notes confirm a round
happened, status is at least Interviewing whatever the mail says.

## Tracker file format

    # Job Application Pipeline
    Last scan: {ISO timestamp}   |   Mode: {incremental | full rebuild}

    | Company | Role | Status | Lane sent | Score | Referral | Last activity | Progress and next step |
    |---|---|---|---|---|---|---|---|

    ---
    Summary: X tracked | Y active | Z referrals | W offers | V rejections

    ## Interview log
    Per company detail from meeting notes, newest first.

    ### Company — Role
    - {date}, {round}. {what happened}. Next: {next step}.

Sort order: Offer, Interviewing, Recruiter follow up, Screening, Applied,
Ghosted, Withdrawn, Rejected.

**Lane sent** and **Score** come from job-match-eval's handoff line. They are
the two columns everyone skips and the only ones that tell you what is working.
If an application has no evaluation behind it, leave them blank rather than
guessing, and note that the row was applied to blind.

## Merge rules

- Match on company plus role. Same company, different role is a separate row.
- Deduplicate by thread id and meeting id. Track the latest applied activity
  date per row and ignore anything at or before it.
- Status only advances, except Rejected, Offer and Withdrawn which always win.
- Rows with no activity in the window are left exactly as they are.
- Meeting notes drive progress text on active rows. Mail drives status.
- Last activity is the most recent matched email or meeting.

## Weekly review

When asked for a review rather than an update, answer these from the file:

- Which lane is getting replies, by reply rate per lane sent.
- What is stalled. Any Recruiter follow up or Interviewing row with no activity
  in ten days.
- Score calibration. Do the rows that advanced score higher than the rows that
  went nowhere? If not, the scoring rubric needs tightening, not the resume.
- How many of these did you actually want.

## Privacy

This skill reads your mail. Keep that boundary visible.

- Never paste full email bodies or transcripts into chat unless asked.
- Never write email content into the tracker file beyond the status and a short
  progress note.
- The tracker file is local. Do not sync it anywhere without saying so.

## Tips

- Be generous about what counts as an application. Better a false positive than
  a missed process.
- Merge several threads from one company into one row at the highest status.
- An interview meeting for a company not in the tracker means a row is missing.
  Add it.
