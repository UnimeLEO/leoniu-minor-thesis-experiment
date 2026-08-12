# leoniu-minor-thesis-experiment

An online experimental platform developed for a Minor Thesis project at the University of Melbourne.

The platform is designed to support controlled second-language (L2) writing experiments examining the effects of **pre-task planning** and **topic familiarity** on writing performance. It provides a browser-based environment for experimental task delivery, timed planning and writing, AI-assisted planning, writing-process logging, participant assignment management, and local research-data export.

The project has evolved through three main development stages:

- **Stable** — core experimental task implementation
- **Beta** — participant assignment and access-management system
- **Gamma** — integrated two-task participant workflow

Development is incremental: later versions build on the experimentally validated core of earlier versions while preserving previous versions as fallback implementations.

---

## Project Overview

The experiment uses a mixed design involving three pre-task planning conditions:

- **AI-assisted planning (AIP)**
- **Individual planning (IP)**
- **No planning (NP)**

and two topic-familiarity conditions:

- **High familiarity**
- **Low familiarity**

Participants complete timed argumentative writing tasks in a controlled browser environment.

Depending on the assigned condition, participants may receive up to 10 minutes of pre-task planning before entering a 30-minute formal writing stage.

The platform is designed to standardise the experimental procedure while reducing researcher-side assignment errors and participant-side operational mistakes.

---

## Core Objectives

The platform was developed with the following objectives:

1. **Standardise experimental task delivery**

   Ensure that participants in different planning and familiarity conditions experience the intended procedure consistently.

2. **Control task and condition assignment**

   Prevent participants from manually selecting or incorrectly entering experimental conditions.

3. **Support AI-assisted planning under controlled conditions**

   Provide access to an AI planning interface during the designated planning stage while preventing AI use during formal writing.

4. **Capture writing-process data**

   Record not only the final written product but also process-level information such as keystrokes, revisions, timing information, and periodic writing snapshots.

5. **Minimise unnecessary server-side research-data storage**

   Substantive research outputs are exported locally as a data package rather than being stored in the platform's assignment-management database.

6. **Maintain experimental robustness and recoverability**

   Preserve older working versions so that development of new infrastructure does not remove the ability to return to a previously tested implementation.

---

# Version History

## Stable — Core Experiment Platform

The original Stable version established the core experimental environment.

Its primary purpose was to answer the basic implementation question:

> **How can the writing experiment itself be conducted reliably in a browser?**

The Stable version introduced the main experimental functions that later versions continue to inherit.

### Core experimental functions

- AI-assisted planning, individual planning, and no-planning conditions
- High- and low-familiarity task routing
- 10-minute planning timer for AIP and IP conditions
- 30-minute formal writing timer
- Early completion of planning and writing stages
- Topic disclosure only after the participant starts the task
- Integrated AI planning interface
- Streaming AI responses
- Markdown rendering of AI responses
- Writing-area word count
- Keystroke and editing-process logging
- Periodic writing snapshots
- Recording of planned and actual task durations
- Planning and writing end-reason logging
- Final essay capture
- AI interaction log for AIP tasks
- Local research-data export as a ZIP package

The Stable version used manually supplied internal condition codes such as:

`ah`, `ih`, `nh`, `al`, `il`, and `nl`.

This implementation successfully established the experimental task engine, but participant assignment and condition management remained relatively manual.

As later versions introduced more robust access-management infrastructure, Stable was retained primarily as the original validated baseline and emergency fallback.

Stable is no longer under active feature development or maintenance.

---

## Beta — Participant Assignment and Access Management

Beta extended the Stable experimental core without fundamentally changing the writing task itself.

The main development goal shifted from:

> **How should the experiment run?**

to:

> **How can participants be assigned to the correct tasks, enter them securely, and be managed reliably during formal data collection?**

Beta therefore introduced a participant assignment and access-management layer around the existing experimental platform. :contentReference[oaicite:0]{index=0}

### Participant access system

Participants no longer entered internal experimental condition codes directly.

Instead, the participant interface used:

- **Subject Code**
- **Access Code**

The backend determined the corresponding task and experimental condition automatically.

Access Codes used an eight-character format:

`XXXX-XXXX`

with visually ambiguous characters excluded where appropriate.

### Server-side assignment

Each participant could be assigned two separate experimental tasks.

In Beta, the basic assignment model was:

`one task = one assignment = one Access Code`

For example:

- Participant P10 — Task 1 — `ih` — Access Code 1
- Participant P10 — Task 2 — `al` — Access Code 2

The experimental condition therefore remained under researcher control rather than participant control.

### Cloudflare D1 assignment database

Beta introduced a Cloudflare D1 database for assignment-management metadata.

The database tracked information such as:

- Subject Code
- Task number
- Condition
- Access Code
- Assignment status
- Creation time
- First task-start time

Supported assignment states included:

- `unused`
- `used`
- `disabled`
- `withdrawn`

Importantly, the D1 database was **not** used as the primary repository for substantive research data such as essays, keystrokes, writing snapshots, or AI conversation logs. Those outputs continued to be exported locally.

### Task-use tracking

Successful authentication did not itself mark a task as completed or started.

When the participant pressed **Start Task**, the frontend issued a server-side `mark_used` request.

This allowed the system to distinguish between:

- successful access authentication; and
- actual experimental task initiation.

The first task-start timestamp was preserved unless the researcher explicitly reset the assignment.

### Participant and Admin backend separation

Beta separated participant-facing and researcher-facing backend responsibilities.

The participant backend handled:

- authentication
- task-use marking
- AI-assisted planning requests

Administrative functions were moved into a separate Admin Worker.

The Admin system supported:

- assignment generation
- assignment listing
- assignment status management
- assignment deletion

### Protected Admin Dashboard

The Beta Admin Dashboard was protected using **Cloudflare Access**.

This removed the need for the earlier shared administrative secret and restricted the management interface to authorised researcher access.

The Admin Dashboard also introduced:

- protected login
- explicit lock/sign-out
- assignment generation
- status controls
- deletion of development/test assignments
- Melbourne-local display of database timestamps

### Additional interface refinements

Beta also introduced several smaller usability improvements, including:

- standardised Subject Code formatting
- consistent form-control fonts
- improved participant-facing access-code handling
- enlarged countdown timers
- clearer separation between participant-visible information and internal experimental condition codes

### Beta design principle

Although Beta substantially increased backend infrastructure, participant-facing complexity remained intentionally low.

The participant procedure was reduced to approximately:

`Subject Code + Access Code → Continue → Start Task → Planning/Writing → Finish`

while assignment logic, condition routing, and task-status management occurred in the background.

## Gamma — Integrated Two-Task Workflow

Gamma is the third major development stage of the experiment platform.

It extends the mature Beta task engine into a continuous participant workflow built around:

`one participant + one Subject Code + one Access Code = Task 1 + Task 2`

The Gamma series progressively introduced participant-level sequence management, automatic task routing, questionnaire handoff, session recovery, formal participant-entry controls, and additional safeguards against accidental interruption.

---

### Gamma 3.0.0 — Mother Template

Established the initial Gamma development baseline from the final Beta v3.5 participant frontend.

Key changes:

- Created an independent `/gamma/` development branch without overwriting Beta.
- Updated visible version metadata to `GAMMA 3.0.0`.
- Preserved the tested Beta experimental core, including:
  - AIP / IP / NP task logic
  - writing-process logging
  - ZIP export
  - AI streaming and Markdown rendering
  - whole Access Code paste
  - enlarged red countdown timers
- Prepared the frontend for migration to a Gamma-specific backend.
- Defined the planned architecture of one Access Code covering a complete two-task participant sequence.

At this stage, Gamma-specific sequence and questionnaire logic had not yet been implemented.

---

### Gamma 3.1.0 — Single-AC Two-Task Backend

Introduced the core backend architecture that distinguishes Gamma from Beta.

Key changes:

- Created the independent D1 SQLite database:

  `minor-thesis-participants-gamma`

- Replaced Beta's task-level assignment model with a participant-level sequence model.
- Introduced the `participant_sequences` table containing:
  - Subject Code
  - one Access Code
  - Task 1 condition
  - Task 2 condition
  - participant status
  - separate Task 1 / Task 2 start and finish timestamps
- Established the rule:

  `one Access Code = Task 1 + Task 2`

- Created the Gamma participant Worker:

  `mt-gamma-ai`

- Implemented:
  - `authorize`
  - `start_task`
  - `finish_task`
- Added automatic server-side routing from Task 1 to Task 2 according to sequence progress.
- Added server-side state enforcement for:
  - task order
  - task start
  - task completion
  - disabled / withdrawn participants
  - AI access permissions
- Preserved the existing controlled AI-planning proxy and streaming configuration.
- Created the first Gamma Admin system for:
  - participant-sequence generation
  - listing
  - status management
  - reset / restore operations
  - deletion
- Connected the participant frontend to the new two-task state machine.

This release established the functional backend foundation for the complete Gamma workflow.

---

### Gamma 3.2.0 — Questionnaire Handoff and Task 2 Recovery

Introduced the first complete transition from the writing platform to an external post-task questionnaire and back.

Key changes:

- Added a `Next Step` button after task completion.
- Required the local ZIP data package to be downloaded before `Next Step` becomes available.
- Added a bilingual confirmation modal before leaving the writing task.
- Added temporary Questionnaire A simulator support for development testing.
- Implemented:

  `Task 1 → Questionnaire A → Gamma`

- Added temporary browser-side sequence-session storage.
- Added automatic session restoration after Questionnaire A.
- Re-authorised the participant against the Gamma Worker on return.
- Automatically restored the participant to **Task 2 Ready** without requiring the Subject Code and Access Code to be entered again.

This release established the core cross-site Task 1 → questionnaire → Task 2 workflow.

---

### Gamma 3.2.1 — Task 2 Resume UI Patch

Refined the Task 2 restoration experience introduced in 3.2.0.

Key changes:

- Improved the restored-session notification.
- Made the Task 2 resume banner span the full experiment layout, including AI-assisted conditions.
- Corrected the restored-task display so that the writing topic remains concealed until the participant actively presses **Start Task**.
- Preserved the principle that returning from Questionnaire A prepares Task 2 but does not automatically begin the task or timer.

---

### Gamma 3.3.0 — Complete Questionnaire Chain

Extended questionnaire handoff from Task 1 only to the complete study sequence.

Key changes:

- Preserved:

  `Task 1 → Questionnaire A → Task 2 Ready`

- Added the Task 2 completion route:

  `Task 2 → Questionnaire B`

- Added a temporary Questionnaire B simulator for end-to-end testing.
- Modelled the final questionnaire chain as:
  - Immediate Post-task Questionnaire B
  - Writing Topic Familiarity
  - Participant Background Information
  - Study completion
- Completed an end-to-end simulated participant flow covering both writing tasks and both questionnaire stages.

This version established the full logical structure of the intended participant journey before live Qualtrics integration.

---

### Gamma 3.4.0 — Live Qualtrics Integration

Replaced the temporary questionnaire handoff architecture with the real Qualtrics workflow.

Key changes:

- Connected Task 1 completion to the real Qualtrics Survey A.
- Configured Survey A to return participants automatically to Gamma.
- Restored and re-authorised the participant into Task 2 Ready after the Survey A return.
- Connected Task 2 completion to the real final Qualtrics questionnaire chain.
- Preserved the intended participant flow:

  `Task 1`
  `→ Survey A`
  `→ Task 2`
  `→ Survey B / Familiarity / Background`
  `→ Study completion`

- Continued to pass only the minimum identifiers required for questionnaire matching, rather than exposing internal condition information or the Access Code.

Gamma 3.4.0 was the first version to implement the intended real-world writing–questionnaire workflow rather than a development simulation.

---

### Gamma 3.5.0 — Formal Participant Entry

Converted the development-style landing page into a formal participant-facing study entry page.

Key changes:

- Renamed the main interface to **Argumentative Writing System**.
- Added the full study title.
- Added a short study description.
- Added University of Melbourne ethics information:
  - Project ID Number
  - Supervisor name and email
  - Researcher name and email
- Added a mandatory participant acknowledgement covering:
  - Consent Form
  - Plain Language Statement (PLS)
  - Participant Instructions
  - voluntary participation
  - participant rights
  - right to withdraw
- Disabled Subject Code, Access Code and `Continue` until the acknowledgement checkbox is selected.
- Preserved the complete single-AC, two-task and Qualtrics workflow introduced in earlier Gamma versions.

This release moved Gamma from a development-oriented interface towards a participant-ready experimental platform.

---

### Gamma 3.5.1 — Participant Document Link

Improved access to participant information and consent materials.

Key changes:

- Converted the combined reference to the:
  - Consent Form
  - Plain Language Statement (PLS)
  - Participant Instructions

  into a direct hyperlink.

- Linked the acknowledgement text to the shared study-document folder.
- Opened the participant documents in a separate browser tab so the Gamma entry page remains available.

No experimental task logic or backend behaviour was changed.

---

### Gamma 3.5.2 — Accidental Leave Warning

Added protection against accidental page refresh, closure, or navigation during an active writing task.

Key changes:

- Added a browser `beforeunload` warning while a task is actively running.
- The warning becomes active only after the server has confirmed `start_task`.
- Participants receive the browser's standard leave-page warning if they attempt to:
  - refresh the page
  - close the tab/window
  - navigate away during an active task
- The safeguard is intended to reduce accidental loss of in-progress writing and process data.
- Existing Gamma sequence, Qualtrics, consent, AI, logging and ZIP-export behaviour remains unchanged.
