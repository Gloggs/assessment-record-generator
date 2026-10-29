# Assessment Record Generator

A Python web app that turns two audio recordings of a spoken English assessment into a drafted assessment record for each learner, ready for the assessor to review, edit and sign off.

**This repository describes the project. The source code is private**, because it was built around real assessment work. I am happy to walk through the code in an interview.

## The problem

A Functional Skills English Level 2 assessment (Speaking, Listening and Communicating) has two parts: each learner gives a presentation, then the group holds a discussion. Afterwards the assessor writes a record for every learner, with a comment against 18 criteria and a feedback paragraph, each backed by what the learner actually said.

Writing these up from the recordings took me about 4 hours per session. The tool cut that to about 30 minutes, and the assessor still makes every judgement.

## What it does

1. The assessor uploads their notes and the two recordings.
2. The app transcribes the audio and works out who is speaking.
3. The assessor checks the transcripts and confirms which voice is which learner.
4. The assessor declares that they listened to the recordings and records what happened on the day. Nothing is drafted until they have.
5. The app drafts each learner's record, quoting what they said.
6. The assessor edits and approves each draft, then downloads one Word document per learner.

## Governance: the assessor stays in charge

The tool drafts. **The assessor is accountable for every record**, so their input is built into the workflow, recorded with their name and the time, and enforced in the code. It is not left to good intentions.

### Before a draft can be generated

Generation will not start until the assessor has made four declarations. This is checked in the code that generates, not only by greying out a button.

| Checkpoint | What the assessor does | What stops generation |
|---|---|---|
| **Recording checked** | Listens to both recordings on the day and confirms the audio is clear enough to assess, or describes what was wrong | No declaration on record |
| **Learners identified** | Reviews which voice is linked to which learner and confirms every name is right | Any learner with no voice linked. If a match changes afterwards, the confirmation is withdrawn automatically |
| **Day record written** | Writes their own account of what happened on the day: the room, interruptions, adjustments. If nothing happened, says so explicitly | An empty or token entry |
| **Windows confirmed** | Places each learner's presentation and question window by ear | Any window still on the software's guess |

The checkpoints show at the top of the Session page, with who confirmed each one and when.

![Checkpoints outstanding: three are waiting for the assessor](screenshots/governance-checkpoints-locked.png)

**One learner is not linked to a voice, so the assessor cannot confirm who is who.** The confirm button stays disabled until every learner in the group is linked.

![The assessor cannot confirm identities while a learner has no voice linked](screenshots/governance-identity-blocked.png)

**Until the declarations are made, the pre-generation checks show red crosses and Generate stays locked.**

![Pre-generation checks with three red crosses and the Generate button locked](screenshots/governance-generation-locked.png)

**The assessor makes each declaration in their own words, with their name and the time recorded.**

![The assessor's declarations: recording, who is who, and the day record](screenshots/governance-declarations-confirmed.png)

![All four checkpoints confirmed, with the assessor's name and time](screenshots/governance-checkpoints-met.png)

**Only then does Generate unlock.**

![Every pre-generation check passed and Generate enabled](screenshots/governance-generation-cleared.png)

### After a draft is generated

The assessor approves each learner's record. **The batch download is held until every record has been approved.**

![Batch download held while two of three records are approved](screenshots/governance-batch-held.png)

![Batch download available once all three are approved](screenshots/governance-batch-released.png)

### The audit trail

Every declaration and withdrawal is logged. Each record's "How this document was produced" panel lists the assessor's declarations, the Database page shows the log across sessions, and every session export carries the same record, so an external quality check can see who confirmed what and when without needing the app.

![A record's provenance panel listing the assessor's declarations](screenshots/governance-provenance.png)

![The governance log across sessions](screenshots/governance-log.png)

### Other controls that keep a person in the loop

- **Observations.** The assessor records what the recording cannot show, and any reasonable adjustment applied. These can evidence a criterion, and are never blended with a transcript quote.
- **No evidence is not a fail.** If a criterion cannot be evidenced from the transcript, it goes to the assessor. The model is not allowed to invent text.
- **Quotes are checked.** Every quote must appear in that learner's own assessment window, and names are checked against the roster.
- **Edits are visible.** The draft as generated is kept beside the assessor's version, and each record states how many times it was edited, or that it was released as drafted.
- **Defects need a decision.** Downloading with checks outstanding needs an explicit acknowledgement, recorded against the session.
- **Judgements are never made by code.** The pass tick, the task-brief tick and the quality-assurance rows on the form are left for people.
- **The instruction sent to the model is locked** with a checksum, and any local change is recorded on every record.
- **Degraded runs are recorded.** If transcription fell back from the best path, the session says so on screen, in the database and in the export.
- **Access is controlled.** A PIN the assessor creates, a sign-in lockout, and a server that listens on the local machine only.

### What this does not claim

- A tick cannot prove that someone listened. A checkpoint is an attestation: it is a named, timed, recorded step that cannot be skipped by accident, and an untrue declaration is on the record against a person.
- It is a single-user local application. Someone with access to the machine could edit the database directly, so device security and backups still matter.
- Per-learner downloads stay open so one problem record never holds up the others, and each shows whether it has been approved.

## Screenshots

These are taken from a **demo session with invented learners** (Maya Fictional, Daniel Example and Priya Sample), invented learner numbers and invented speech. They show the app working on a finished session, and no real assessment data appears in them. The app has a black and white design with a light and a dark theme.

### Signing in

There is no default PIN. The first time the app opens, the assessor creates one. After that it asks for it, and repeated wrong attempts lock the sign-in for a growing period.

| | |
|---|---|
| ![Create a PIN on first launch](screenshots/create-pin.png) | ![Sign in with a PIN](screenshots/sign-in.png) |
| First launch: the assessor creates their own PIN | Every later launch: sign in with it |

### Working through a session

| | |
|---|---|
| ![Session overview in the light theme](screenshots/session-light.png) | ![Session overview in the dark theme](screenshots/session-dark.png) |
| A finished session reopens where the assessor left off | The same screen in the dark theme |

**Step 2, review and confirm.** The assessor checks the transcript and confirms the window in which each learner gave their presentation and answered questions. Every duration in the record comes from these confirmed windows.

![Step 2: review the transcript](screenshots/step2-review.png)

![Step 2: confirmed spans for each learner](screenshots/step2-spans.png)

**Step 3, generate.** Checks run before any money is spent: group size, every voice matched, a topic set and every window confirmed. The assessor can add what they saw on the day, and the app shows an estimated cost.

![Step 3: pre-generation checks and estimated cost](screenshots/step3-generate.png)

**Step 4, review and download.** Each learner's record opens as a card. Every comment is backed by a quote and a timestamp, and the card says how the document was produced.

![Step 4: a learner's record with its provenance panel](screenshots/learner-card-provenance.png)

![Step 4: criterion comments with quotes and timestamps](screenshots/task1-evidence.png)

![Step 4 in the dark theme](screenshots/step4-dark.png)

### Data and settings

| | |
|---|---|
| ![Database page](screenshots/database.png) | |
| Saved sessions, backups and a self-contained export for an external quality check | |

![Settings page](screenshots/settings.png)

Settings: assessor and centre details, the AI model, session timings, backups, and data protection tools for retention and for erasing a single learner.

## How it works

```mermaid
flowchart TD
    A[Assessor notes and two recordings] --> B[Readiness checks]
    B --> C[Transcription with Whisper, on the local machine]
    C --> D[Speaker separation with pyannote]
    D --> E[Match voices to learner names]
    E --> F[Verify each match and keep the evidence]
    F --> G[Find each learner's presentation and question window]
    G --> H[Measure strengths and areas to develop]
    H --> I[Draft comments and feedback with the Claude API]
    I --> J[Validate every quote and flag anything to review]
    J --> K[Assessor edits]
    K --> L[Word document per learner]
```

## Engineering decisions

**Fail early, not 40 minutes in.** A long transcription used to fail late and quietly, for example when a graphics library was missing. Readiness checks now run before any work starts and say exactly what to fix.

**No invented evidence.** Every criterion is in one of three states: evidenced by a real quote from that learner, searched with nothing found (which goes to the assessor and never counts as a fail), or supplied by the assessor from their own observation. A quote that cannot be found in the right transcript, inside that learner's own speaking window, is rejected.

**Measure first, then ask the model.** Candidate moments, strengths and areas for development are found by ordinary code before the AI model is called. The model is given that evidence to write from, so it cannot make up what a learner said, and two learners do not get the same generic feedback.

**The same voice across two recordings.** Speaker separation numbers the voices from scratch in each file, so "speaker 1" in the presentation is not "speaker 1" in the discussion. The app matches voices across both recordings by their sound, and keeps a record of why it made each match.

**Validation that can block.** Checks after drafting return issues with a severity. Serious ones stop the download until they are fixed. Minor ones are flagged for the assessor.

**The record must survive.** Everything is stored in one SQLite database with JSON backups, so a record can be produced again if an external quality check asks for it.

**Security reviewed, then fixed.** I ran an AI-assisted security audit (Cloudflare's open-source security-audit skill) over the highest-risk parts of the code. It was a read-only review of the source, so I treated its findings as leads to verify rather than facts. The real ones are fixed and tested: the app has no default PIN, locks sign-in after repeated wrong attempts and listens only on the local machine; erasing a learner now removes every copy of their data and not only the database rows; and a record can no longer be matched to the wrong learner's document.

## Testing and quality

- Over 500 automated tests across 23 test files, written with pytest
- Every change is linted, format-checked and tested by GitHub Actions
- Tests cover speaker verification, evidence rules, validators, document building, backup, session restore, the PIN gate, data erasure and the assessor checkpoints
- The prompt sent to the model is locked with a checksum, so it cannot change by accident

## Stack

| Part | Built with |
|---|---|
| Interface | Streamlit with a custom design system, light and dark |
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
- The app is protected by a PIN that the assessor creates on first launch. It is stored only as a salted hash, repeated wrong attempts lock the sign-in, and the app listens on the local machine only.
- Retention and erasure tools remove a session or a single learner from the database, the backups and the generated documents.

## Size

- About 21,500 lines of application code
- About 5,800 lines of tests
- Set-up scripts that install, repair and update the app on a new PC with one command

## What I learned

- An AI model is one step in a pipeline, not the pipeline. The checks around it are what make the output trustworthy.
- Tests matter most where a wrong answer is costly. A record with a quote the learner never said is worse than no record.
- Good error messages save more time than clever code.

## Contact

Michael Ijisonuwe · [LinkedIn](https://uk.linkedin.com/in/michael-ijisonuwe-944657254) · ijisonuwe@outlook.com
