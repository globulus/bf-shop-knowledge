# Shopfront knowledge base

Everything we know about what Shopfront should do: client notes, the PRD built
from them, open questions, decisions and the backlog. Managed with Globuli.

## How it works

1. After a client visit, call or email, write a note in `inputs/` (copy
   `templates/note.md`). Free text is fine. Add attachments under
   `inputs/attachments/<note-id>/`.
2. Push a branch and open a PR. The synthesizer turns the note into PRD changes
   and questions; the critic checks the result. Both commit to your PR.
3. Review the PR (your note and the agents' reading of it) and merge.
4. When a requirement is good to build, set `status: ready` in a PR. CI checks
   the Definition of Ready.

Notes are never edited after they are merged. To correct one, write a new note
with `corrects: <note-id>`.

If the project takes its PRDs from Linear (`intake.linear` in `globuli.yaml`),
Globuli writes each issue that is ready, or carries the intake label, to
`inputs/LIN-<TEAM>-<n>.md` on a notes PR of its own: the issue's title, state,
labels and link as front matter, its description as the body. Requirements
cite it as `LIN-<TEAM>-<n>`. Unlike a note, Globuli rewrites the file when the
issue changes, on the open PR or on a new one; change the issue in Linear,
not the file. Globuli never writes to Linear.

With `intake.figma` as well, the frames of the Figma designs an issue links to
come with it as PNG images in `inputs/attachments/LIN-<TEAM>-<n>/`, listed in
the file's "Designs" section. Globuli replaces them when the issue's Figma
links change. When Figma can't be asked, Globuli waits for it, up to three
hours, before taking the issue in with a note instead of the images.

## Layout

| Path | What | Owner |
|---|---|---|
| `inputs/` | Client notes and attachments (append-only); `LIN-*.md` from Linear, their designs in `attachments/LIN-*/` | People; Globuli for `LIN-*` |
| `prd/overview.md` | Vision, client, users, goals, non-goals, module map | Maintainer (agents propose) |
| `prd/releases/<version>.md` | What a release is meant to achieve | Maintainer (agents propose) |
| `prd/modules/<module>.md` | Narrative per module, links to its requirements | Stage 1 agents |
| `prd/requirements/` | `FR-NNNN.md`, `NFR-NNNN.md` | Stage 1 agents; status gates by maintainer |
| `glossary.md` | Croatian term ↔ English term ↔ code name | Stage 1 agents (reviewed in PRs) |
| `roles.md` | User roles and what each may do per module | Stage 1 agents |
| `stakeholders.md` | Client people and what they can decide | Stage 1 agents |
| `questions/` | `Q-NNNN.md`, one per question; the visit agenda is generated from them | Stage 1 agents, orchestrator |
| `decisions.md` | `DEC-NNNN` project decisions | Whoever records them |
| `backlog/` | `BL-NNNN.md` deferred findings and ideas | Orchestrator |
| `checklists/` | Domain checklists the critic applies on top of Globuli's generic one | Maintainer |
| `templates/` | Templates for all of the above | Globuli |

IDs follow Globuli's `docs/ids.md`. Requirement statuses and the Definition of
Ready follow Globuli's `docs/requirement-lifecycle.md`.
