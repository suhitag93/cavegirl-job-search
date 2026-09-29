---
name: interview-coach
description: Build a personalised interview guide from your mail, calendar, meeting notes, resume, and the live job posting. Trigger on "prepare for my interview", "interview prep", "generate an interview guide", "help me get ready for my [Company] interview", or a posting pasted with intent to interview. Produces a four tab interactive guide covering the role, your fit, your pipeline, and what to say. Do not answer company specific interview questions without it.
---

# Interview Coach

Pulls five sources together into one guide you can actually open the morning of.

## Step 0 — Load config

Read `config.yml` for the owner, the resume variants and the workspace paths.
Read the resume variant that was sent for this application. The tracker's
**Lane sent** column tells you which one. If the tracker has no row, ask.

## Inputs

| Parameter | Required | Notes |
|---|---|---|
| company | yes | |
| role | yes | |
| round | no | Screen, technical, hiring manager, panel, final. Changes everything about the output, so ask if it is not obvious from calendar. |

## Hard rules

1. **Never invent experience.** Same rule as job-match-eval, and it matters more
   here, because a fabricated line in a document gets caught by a reader while a
   fabricated line in an interview gets caught by a follow up question in front
   of a person.
2. **Everything is a prompt, not a script.** Talking points are there to remind
   the user of a real story they lived. Write them so they cannot be read aloud
   word for word. Short, in their own vocabulary, with the specifics they will
   expand on.
3. **Name the hard questions.** Every real candidate has two or three questions
   they are dreading. Find them, from the gaps in the match evaluation and the
   shape of their history, and prepare honest answers. A guide that only covers
   strengths is a comfort object.
4. **No calendar event does not mean no interview.** If nothing is scheduled,
   say so and build the guide anyway.

## Sources, run in parallel

1. **Mail** — `mcp__Gmail__search_threads`, 90 days, company and role terms.
   Recruiter intros, tone, the referral if there is one, anything you were told
   about the format.
2. **Calendar** — `mcp__Google_Calendar__list_events`. The round, the time, the
   attendees, the format, the link. Attendee names are what make Tab 4 specific.
3. **Meeting notes** — prior rounds with this company, and any research call.
4. **Resume** — the variant that was actually sent.
5. **Web** — the live posting, plus the company's mission, funding, customers,
   recent launches and recent news. Recency matters. A guide built on two year
   old facts reads as homework not done.

If the posting cannot be found, ask for it. Do not build on generic framing.

## Output: four tab HTML guide

Always render as an interactive artifact, never as prose in chat.

**Tab 1, the role.** Company context, what the team actually does, the
responsibilities as written, and the signal language the posting keeps
repeating. Note what they seem anxious about, because that is what the
interview will probe.

**Tab 2, your fit.** Your experience mapped to their requirements, strongest
first. The specific outcomes and numbers to reach for. Then the gaps, named
plainly, each with the honest answer: what you have done that is adjacent, what
you would need to learn, and how fast.

**Tab 3, your pipeline.** Referral source. The schedule, who is in each round,
their role and anything known about them. What the round before this one
surfaced. A short pre call checklist.

**Tab 4, what to say.** Per interviewer where known:

- Two or three story prompts, each a real situation from the resume, tagged
  with what it demonstrates. Situation, what you owned, what you decided, what
  went wrong, what you did, the outcome. Prompts, not paragraphs.
- The hard questions for this specific role, with honest answers drafted.
- Questions to ask, differentiated per interviewer. A recruiter, an engineer
  and a hiring manager should not get the same question.
- Five things to land, as a checklist.

Save the guide to `{workspace.dir}/{workspace.guides_dir}/{YYYY-MM-DD}_{company}_{round}.html`.

## Framing by context

**Company stage.** Early stage rewards ambiguity tolerance and range. Large
enterprise rewards scale, governance and stakeholder navigation. Growth stage
rewards someone who has seen a process break and rebuilt it.

**Round.** A recruiter screen is about narrative and motivation. A technical
round is about depth and how you reason out loud. A hiring manager round is
about ownership and judgement under constraint. A final is usually about risk,
meaning they are deciding whether to bet, so it wants clarity about what you
want next.

**Role family.** Solutions and field engineering rewards discovery, architecture
patterns and enterprise constraints. Customer facing account roles reward
outcomes, adoption and escalation handling. Program roles reward dependency
management and decisions made with incomplete information. Product roles reward
prioritisation you can defend.

## When something is missing

| Missing | What to do |
|---|---|
| Posting | Ask for it. Do not proceed on generic framing. |
| Calendar event | Note it, build anyway, ask whether it is confirmed. |
| Prior meeting notes | Normal for a first round. Move on. |
| Resume variant unclear | Check the tracker's Lane sent column, then ask. |
| Talking points feel generic | You are working from the posting, not the resume. Go back to Tab 2 and pull real specifics. |

## After the interview

Offer to append what happened to the tracker's interview log: the round, how it
went, what was flagged, and the next step. That is what makes the next guide
better than this one.
