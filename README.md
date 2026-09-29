# Job Search Skills

Three Claude skills that turn a job search from a pile of browser tabs into a
system: score a role before you spend an evening on it, keep a pipeline that
updates itself from your inbox, and walk into an interview with a guide built
from what actually happened.

They are designed to work together. The evaluator hands a score and a resume
lane to the tracker. The tracker tells the interview coach which resume you
actually sent. That loop is the point, and it is what tells you which version
of you is getting replies.

| Skill | Kind | What it does |
|---|---|---|
| `job-match-eval` | You call it | Scores a posting against your resume variants, names the gaps, drafts the rewrite. Also scans a watchlist of target companies. |
| `job-application-tracker` | Runs on a schedule | Reads your mail and meeting notes, keeps one pipeline file current. |
| `interview-coach` | You call it | Builds a four tab interview guide from mail, calendar, notes, resume and the live posting. |

---

## What you need

**Claude with skills support.** Claude Code, or the Claude desktop app with
skills enabled. Anything that can load a `SKILL.md`.

**MCP connectors.** These skills are thin. The work is done by connectors that
reach your real data. You do not need all of them, but each one you skip
removes a capability, listed below.

| Connector | Used by | What breaks without it |
|---|---|---|
| Gmail | tracker, coach | No pipeline, no referral detection |
| Google Calendar | coach | No schedule, no interviewer names |
| Google Drive | all | Only matters if your resumes live in Drive |
| Filesystem | tracker, eval | Nothing persists between runs |
| A meeting notes tool | tracker, coach | Interview detail gets thin |
| Web fetch or a browser | eval, coach | Cannot read postings |

---

## Setup

### 1. Clone and configure

    git clone https://github.com/<you>/job-search-skills.git
    cd job-search-skills
    cp config.example.yml config.yml

Open `config.yml` and fill in your name, your resume lanes, and where you want
the workspace to live. `config.yml` is gitignored, so your details never get
committed.

### 2. Add your resumes

Put two resume variants in `profile/` as plain markdown:

    profile/resume-primary.md
    profile/resume-secondary.md

Two lanes is the right number for most people. One means you are applying to
one kind of role, which is fine but does not need a scorer. Five means you have
not chosen, and the scores will be mush. See `profile/README.md`.

Add your target companies to `profile/target-companies.md`. Keep it to ten or
fifteen. A watchlist of forty produces output you will learn to skip.

These files are gitignored. Only the `.example.md` versions are committed, so
you can make this repo public without publishing your resume.

### 3. Connect the MCP servers

**In Claude Code**, add them to your MCP config. Gmail, Calendar and Drive are
OAuth connectors: you approve them once in a browser window and Claude gets a
scoped token. Filesystem is local and takes a list of directories it is allowed
to touch. Point it at this repo's `workspace/` directory and nothing else.

    claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem ./workspace

For the Google connectors, use your client's connector directory rather than
hand rolling credentials. In the Claude desktop app that is Settings, then
Connectors. In Claude Code it is `claude mcp add` with the connector's
published command.

**Verify before you trust it.** Ask Claude to list your connected tools. If
Gmail search is not there, the tracker will silently produce an empty pipeline
and you will think you have no applications.

### 4. Install the skills

Copy each folder under `skills/` into wherever your client loads skills from,
or point your client at this repo. Each skill is a single `SKILL.md` with no
dependencies.

### 5. First run

Build the pipeline from scratch before you schedule anything:

    "Full rebuild of my job application tracker"

That scans ninety days and creates `workspace/job-application-tracker.md`.
Read it. It will be wrong in places, because mail is messy. Fix the rows by
hand once. Everything after that is incremental and stays close to correct.

### 6. Schedule the tracker

The tracker is the one that should run on its own, because it is the habit
people drop first. Set it up as a scheduled task in your client, daily, early.
Ask for the daily incremental update, not a rebuild.

The other two are on demand by design. A daily watchlist scan returns the same
roles every morning until you stop reading it.

---

## Using them

### Score a posting before you spend the evening

    "Score this JD: https://jobs.example.com/12345"

You get a recommendation on which resume lane to lead with, a weighted score
per variant, the keywords you match directly and the ones you only match
adjacently, the gaps that are real, and three to five rewritten bullets that
cite the source bullet they replace.

The number is not the point. The gaps are. Ask for those first, because that
list is your development plan.

Expect scores between 55 and 75 on real postings. The skill is calibrated to
resist inflation, because a 90 you cannot defend line by line is worse than an
honest 68.

### Scan the watchlist

    "Scan my watchlist"

Crawls the careers pages in `profile/target-companies.md`, filters to roles in
your lanes, ranks them, and offers to run a full evaluation on the top three.
Weekly.

### Keep the pipeline current

    "Run the daily tracker update"

Scans the last thirty six hours of mail and meeting notes and merges changes
into the tracker. The window overlaps the previous run by twelve hours on
purpose, so a missed run does not lose anything, and deduplication stops the
overlap creating doubles.

Statuses go Offer, Rejected, Withdrawn, Interviewing, Recruiter follow up,
Screening, Applied, Ghosted. Recruiter follow up exists as its own status
because it is the row that rots. Somebody replied, wants something from you,
and nothing is scheduled.

Ask for a weekly review to get the questions that matter:

    "Weekly review of my pipeline"

Which lane is getting replies. What is stalled. Whether the roles that advanced
actually scored higher than the ones that went nowhere, which tells you if your
scoring is calibrated or just generous.

### Prepare for the interview

    "Prepare for my Example Co interview, hiring manager round"

Builds a four tab guide: the role, your fit, your pipeline, and what to say.
The last tab is story prompts rather than a script, on purpose. It also names
the two or three questions you are dreading and drafts honest answers, because
a guide that only covers your strengths is a comfort object.

Afterwards, let it write what happened back to the tracker's interview log.
That is what makes the next guide better than this one.

---

## The rules these skills follow

**Nothing is invented.** Every drafted bullet maps to a real line in your
resume. If a rewrite needs a number that is not there, the skill asks instead of
estimating. This matters most in the interview guide, where a fabricated line
gets caught by a follow up question in front of a person.

**No hyphens as connectors in drafted bullets.** A house style rule, enforced in
the scorer.

**Conservative scoring.** Vocabulary matching is not evidence.

**The gaps get named out loud.** Including the uncomfortable ones.

---

## Privacy

The tracker reads your mail. That is the whole mechanism, and it should stay
visible rather than becoming background noise.

- Email bodies and transcripts are never pasted into chat unless you ask.
- The tracker file holds a status and a short progress note, not email content.
- `workspace/` is gitignored. Nothing generated gets committed.
- Your resumes and company list are gitignored.
- Connectors use scoped OAuth tokens held by your client, not by this repo.
  There are no credentials in this repository and there should never be.

Before you connect a work account, check your employer's policy. A personal
account is the safer default for a job search.

---

## If something is not working

**Empty pipeline after a full rebuild.** The Gmail connector is probably not
connected. Ask Claude to list its tools and confirm.

**Everything scores in the 90s.** Your resume variants are too similar, or too
generic. Two lanes that score within five points of each other on every posting
are one lane.

**Guides feel generic.** The coach is working from the posting instead of your
resume. Check that the resume file path in `config.yml` resolves and that the
tracker knows which lane was sent.

**Tracker keeps creating duplicate rows.** Same company with slightly different
role titles. Normalise the titles in the file once by hand.

---

## License

MIT. See `LICENSE`.
