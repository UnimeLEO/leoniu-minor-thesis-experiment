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

Gamma represents the next major development stage of the experiment platform.

Unlike Beta, where each task is managed as a separate assignment with its own Access Code, Gamma is designed around a participant-level sequence:

`one participant + one Access Code = Task 1 + Task 2`

The Gamma architecture therefore requires a new backend data model capable of storing both task conditions and task-level progress within a single participant sequence.

---

### Gamma 3.0.0 — Backend Database Foundation

The first Gamma backend implementation introduced a dedicated Cloudflare D1 SQLite database:

`minor-thesis-participants-gamma`

This database is fully separated from the existing Beta database so that Gamma development does not modify or destabilise the Beta assignment system.

The new database uses a participant-sequence model rather than Beta's task-level assignment model.

#### Core table

A new table was created:

`participant_sequences`

Each row represents one participant's complete two-task experimental sequence.

The table stores:

- unique internal sequence ID
- unique Subject Code
- unique Access Code
- Task 1 experimental condition
- Task 2 experimental condition
- overall participant-sequence status
- sequence creation time
- Task 1 start time
- Task 1 finish time
- Task 2 start time
- Task 2 finish time

The six valid experimental conditions remain:

`ah`, `ih`, `nh`, `al`, `il`, `nl`

Both `task_1_condition` and `task_2_condition` are constrained at the database level to these valid values.

#### Sequence status

The participant sequence currently supports four overall states:

- `unused`
- `used`
- `disabled`
- `withdrawn`

New sequences default to:

`unused`

Database-level `CHECK` constraints prevent invalid status values from being written.

#### Task-level progress timestamps

Unlike Beta, Gamma stores separate timestamps for each task:

- `task_1_started_at`
- `task_1_finished_at`
- `task_2_started_at`
- `task_2_finished_at`

This provides the database structure required for future participant-state routing and recovery.

For example, the backend will eventually be able to distinguish between states such as:

`not started`

`Task 1 started`

`Task 1 completed`

`Task 2 started`

`Task 2 completed`

without representing Task 1 and Task 2 as separate assignments.

#### Database uniqueness constraints

Both of the following fields are unique:

- `subject_code`
- `access_code`

This ensures that one Subject Code corresponds to one Gamma participant sequence and that one Access Code cannot be assigned to multiple participants.

This is a major structural difference from Beta, where the same participant normally had two database rows and two Access Codes.

#### Database indexes

Two indexes were added:

`idx_gamma_sequences_status`

for efficient status-based queries, and:

`idx_gamma_sequences_subject_access`

for efficient Subject Code + Access Code lookup during participant authentication.

#### Current implementation status

At this stage, the Gamma database schema has been created, but the full participant sequence logic has not yet been implemented.

The current backend milestone therefore establishes the persistence layer required for later development of:

- single-Access-Code authentication
- Task 1 / Task 2 routing
- task start and finish tracking
- participant progress recovery
- Qualtrics transition handling
- Gamma Admin assignment generation and management

The existing Beta database and Beta backend remain unchanged.

Beta therefore became the project's first mature **participant assignment and access-management architecture**, while preserving the experimentally validated Stable task engine.

It remains an important fallback implementation and development reference for later versions.
