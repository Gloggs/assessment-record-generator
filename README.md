# Assessment Record Generator

A Python web app that turns two audio recordings of a spoken English assessment into a drafted record for each learner. The assessor reviews, edits and approves every record.

The source code is private because it was built around real assessment work. I am happy to walk through it in an interview.

## The problem

A Functional Skills English Level 2 assessment (Speaking, Listening and Communicating) has two parts. Each learner gives a presentation, then the group holds a discussion. Afterwards the assessor writes a record for every learner: a comment against each criterion, backed by what the learner said, and a feedback paragraph.

Writing these up from the recordings took me about 4 hours per session. The tool cut that to about 30 minutes. The assessor still makes every judgement.

## What it does

1. The assessor uploads their notes and the two recordings.
2. The app transcribes the audio and works out who is speaking.
3. The assessor checks the transcripts and confirms which voice belongs to which learner.
4. The assessor confirms they listened to the recordings and writes down what happened on the day. Nothing is drafted until they have.
5. The app drafts each learner's record, quoting what they said.
6. The assessor edits and approves each draft, then downloads one Word document per learner.

## The assessor stays in charge

The app drafts. The assessor is responsible for every record. Their input is part of the workflow, it is recorded with their name and the time, and the code enforces it.

### Before any draft is generated

Four declarations must be made first. The check sits in the code that generates, as well as on the button.

| Checkpoint | What the assessor does | What stops generation |
|---|---|---|
| Recording checked | Listens to both recordings and confirms the audio is clear enough to assess, or describes what was wrong | No declaration on record |
| Learners identified | Checks which voice is linked to which learner and confirms each name | Any learner with no voice linked. If a match changes later, the confirmation is withdrawn |
| Day record written | Writes their own account of the day: the room, interruptions, adjustments. If nothing happened, says so | An empty or token entry |
| Windows confirmed | Places each learner's presentation and question window by ear | Any window still on the software's guess |

The checkpoints show at the top of the Session page, with who confirmed each one and when.

![Three checkpoints are waiting for the assessor](screenshots/governance-checkpoints-locked.png)

If a learner has no voice linked, the assessor cannot confirm who is who.

![Identity confirmation blocked because one learner has no voice linked](screenshots/governance-identity-blocked.png)

Until the declarations are made, the pre-generation checks show red crosses and Generate stays locked.

![Pre-generation checks with three red crosses and Generate locked](screenshots/governance-generation-locked.png)

The assessor makes each declaration in their own words.

![The assessor's declarations for the recording, who is who and the day](screenshots/governance-declarations-confirmed.png)

![All four checkpoints confirmed, with the assessor's name and time](screenshots/governance-checkpoints-met.png)

Only then does Generate unlock.

![Every pre-generation check passed and Generate available](screenshots/governance-generation-cleared.png)

### After a draft is generated

The assessor approves each record. The batch download stays held until every record is approved.

![Batch download held while two of three records are approved](screenshots/governance-batch-held.png)

![Batch download available once all three are approved](screenshots/governance-batch-released.png)

### The audit trail

Every declaration and withdrawal is logged. Each record's provenance panel lists the declarations, the Database page shows the log across sessions, and every session export carries the same record. An external quality check can see who confirmed what, and when, without needing the app.

![A record's provenance panel listing the assessor's declarations](screenshots/governance-provenance.png)

![The governance log across sessions](screenshots/governance-log.png)

### Other controls that keep a person in the loop

- **Observations.** The assessor records what the recording cannot show, and any adjustment applied. An observation can evidence a criterion. It is never mixed into a quote.
- **No evidence is not a fail.** If the transcript cannot evidence a criterion, it goes to the assessor. The model may not invent text.
- **Quotes are checked.** Every quote must appear in that learner's own assessment window, and names are checked against the roster.
- **Edits are visible.** The draft as generated is kept next to the assessor's version. Each record says how often it was edited, or that it was released as drafted.
- **Defects need a decision.** Downloading with checks outstanding needs an explicit acknowledgement, recorded against the session.
- **Judgements stay with people.** The pass tick, the task-brief tick and the quality-assurance rows on the form are never filled in by code.
- **The instruction sent to the model is locked** with a checksum, and any local change is recorded on every record.
- **Degraded runs are recorded.** If transcription fell back from the best path, the session says so on screen, in the database and in the export.
- **Access is controlled.** The assessor creates a PIN, repeated wrong attempts lock the sign-in, and the server listens on the local machine only.

### What this does not claim

- A tick cannot prove that someone listened. A checkpoint is a named, timed, recorded step that cannot be skipped by accident. An untrue declaration is on the record against a person.
- It is a single-user local app. Someone with access to the machine could edit the database directly, so device security and backups still matter.
- Per-learner downloads stay open so one problem record never holds up the others. Each one shows whether it has been approved.

## Screenshots

These come from a demo session with invented learners, numbers and speech. No real assessment data appears. The design uses the Guild UK colours and one typeface throughout.

| | |
|---|---|
| ![Create a PIN on first launch](screenshots/create-pin.png) | ![Sign in with the PIN](screenshots/sign-in.png) |
| First launch: the assessor creates their own PIN | Every later launch: sign in with it |
| ![Session overview in the light theme](screenshots/session-light.png) | ![Session overview in the dark theme](screenshots/session-dark.png) |
| A finished session reopens where the assessor left off | The same screen in the dark theme |

### Step 2: review and confirm

The assessor checks the transcript and confirms the window in which each learner presented and answered questions. Every duration in the record comes from these windows.

![Step 2: the transcript](screenshots/step2-review.png)

![Step 2: confirmed windows for each learner](screenshots/step2-spans.png)

### Step 3: generate

Checks run before any money is spent: group size, every voice matched, a topic set and every window confirmed. The assessor can add what they saw on the day, and the app shows an estimated cost.

![Step 3: pre-generation checks and estimated cost](screenshots/step3-generate.png)

### Step 4: review and download

Each learner's record opens as a card. Every comment cites a quote and a timestamp, and the card says how the document was produced.

![Step 4: a learner's record and its provenance panel](screenshots/learner-card-provenance.png)

![Step 4: comments with quotes and timestamps](screenshots/task1-evidence.png)

![Step 4 in the dark theme](screenshots/step4-dark.png)

### Data and settings

![Database page](screenshots/database.png)

Saved sessions, backups, and a self-contained export for an external quality check.

![Settings page](screenshots/settings.png)

Assessor and centre details, the AI model, session timings, backups, and the tools for retention and for erasing one learner.

## How it works

```mermaid
flowchart TD
    A[Assessor notes and two recordings] --> B[Readiness checks]
    B --> C[Transcription with Whisper, on the local machine]
    C --> D[Speaker separation with pyannote]
    D --> E[Match voices to learner names]
    E --> F[Verify each match and keep the evidence]
    F --> G[Assessor declarations]
    G --> H[Find each learner's presentation and question window]
    H --> I[Measure strengths and areas to develop]
    I --> J[Draft comments and feedback with the Claude API]
    J --> K[Validate every quote and flag anything to review]
    K --> L[Assessor edits and approves]
    L --> M[Word document per learner]
```

## Engineering decisions

**Fail early.** A long transcription used to fail late and quietly, for example when a graphics library was missing. Readiness checks now run before any work starts and say exactly what to fix.

**No invented evidence.** Every criterion is in one of three states: evidenced by a real quote from that learner, searched with nothing found (which goes to the assessor and never counts as a fail), or supplied by the assessor from their own observation. A quote that cannot be found in the right transcript, inside that learner's own speaking window, is rejected.

**Measure first, then ask the model.** Ordinary code finds the candidate moments, strengths and areas for development before the AI model is called. The model writes from that evidence, so it cannot make up what a learner said, and two learners do not get the same generic feedback.

**The same voice across two recordings.** Speaker separation numbers the voices from scratch in each file, so "speaker 1" in the presentation is not "speaker 1" in the discussion. The app matches voices across both recordings by their sound and keeps a record of why it made each match.

**Validation that can block.** Checks after drafting return issues with a severity. Serious ones stop the download until they are fixed. Minor ones are flagged for the assessor.

**The record must survive.** Everything is stored in one SQLite database with JSON backups, so a record can be produced again if an external quality check asks for it.

**Security reviewed, then fixed.** I ran an AI-assisted security audit (Cloudflare's open-source security-audit skill) over the highest-risk parts of the code. It was a read-only review of the source, so I treated its findings as leads to check. The real ones are fixed and tested: no default PIN, a lockout after repeated wrong attempts, a server that listens only on the local machine, erasure that removes every copy of a learner's data, and records that can no longer be matched to the wrong learner's document.

## Testing and quality

- Over 500 automated tests across 23 test files, written with pytest
- Every change is linted, format-checked and tested by GitHub Actions
- Tests cover speaker verification, evidence rules, validators, document building, backup, session restore, the PIN gate, data erasure and the assessor checkpoints
- The prompt sent to the model is locked with a checksum, so it cannot change by accident

## Stack

| Part | Built with |
|---|---|
| Interface | Streamlit, with a light and a dark theme and one typeface |
| Transcription | faster-whisper (Whisper large-v3), GPU with a CPU fallback |
| Speaker separation | pyannote.audio |
| Drafting | Anthropic Claude API, one call per learner, with prompt caching |
| Documents | python-docx |
| Storage | SQLite with JSON backups |
| Quality | pytest, Ruff, GitHub Actions |

## Privacy

- The audio is transcribed on the assessor's own machine and is never uploaded.
- The transcript text is sent to the Claude API to draft the comments.
- Learner names, recordings, transcripts, documents and the database are excluded from version control. The repository holds source code only.
- The PIN is created by the assessor and stored only as a salted hash. The app listens on the local machine only.
- Retention and erasure tools remove a session, or one learner, from the database, the backups and the generated documents.

## Size

- About 21,500 lines of application code
- About 5,800 lines of tests
- Set-up scripts that install, repair and update the app on a new PC with one command

## What I learned

- An AI model is one step in a pipeline, not the pipeline. The checks around it are what make the output trustworthy.
- Tests matter most where a wrong answer is costly. A record with a quote the learner never said is worse than no record.
- Good error messages save more time than clever code.
- A control only counts if the code enforces it and the record shows it.

## Contact

Michael Ijisonuwe · [LinkedIn](https://uk.linkedin.com/in/michael-ijisonuwe-944657254) · ijisonuwe@outlook.com
