---
name: commit-assistant
description: >
  Stage current work and create a git commit safely. Use when the user asks to stage changes and generate commit messages.
  The skill must: inspect git status, perform safety checks (secrets/binaries/large change), stage changes, analyze staged diff,
  generate exactly 3 distinct Conventional Commit message candidates, ask the user to select 1/2/3, then commit with the chosen message.
  Never push. Never commit without an explicit user selection.
---

# Commit Assistant (Production)

## Intent
Turn the current working tree changes into a single well-written commit with minimal risk:
- safe staging
- high-signal commit messages
- user-in-the-loop final decision

## Hard Constraints (Do NOT violate)
- Always run `git status -sb` first.
- If not in a git repository, stop.
- Never `git push`.
- Never commit unless user selects 1/2/3.
- If there are no staged changes after staging, stop and explain.
- Generate EXACTLY 3 commit message candidates (distinct).
- If suspicious secrets are detected, do NOT stage/commit automatically. Ask the user.

## Safety & Risk Checks (before staging)
1) Identify candidate files:
   - `git status --porcelain=v1`
2) Detect likely sensitive files by path/name patterns (case-insensitive). If any match, STOP and ask:
   - `.env`, `.env.*`, `*.pem`, `*.key`, `id_rsa*`, `*.p12`, `*.pfx`, `*.jks`
   - `credentials*`, `secret*`, `token*`, `apikey*`, `service-account*`
   - `config/*.json` or `*.yml` that look like credentials (use judgement; ask if unsure)
3) Detect large/binary additions:
   - Stage is not yet done, so check working tree size signals:
     - Any file extensions commonly binary: `png jpg jpeg gif webp pdf zip jar war exe dmg mp4 mov`
     - If large binaries appear, warn and ask whether to include them.
4) If change set is huge (many files), warn:
   - if `git status --porcelain` shows > 30 files changed, ask: "Commit as one? or split?"
   - Default behavior: still proceed as one commit if user confirms. If user doesn't confirm, stop.

## Staging Strategy
- Default: stage everything that is currently changed/new:
  - `git add -A`
- If the user previously requested partial staging, obey that.
- After staging, verify staged content:
  - `git diff --cached --stat`
  - If empty, stop.

## Diff Analysis (derive intent, type, scope)
Use staged diff to infer:
- Type (Conventional Commit):
  - `feat`: adds capability/endpoint/flow
  - `fix`: bug fix, incorrect behavior, edge-case handling
  - `refactor`: code movement/cleanup without behavior change
  - `docs`: docs/comments only
  - `test`: test changes only
  - `chore`: tooling, build, deps, formatting, housekeeping
  - `perf`: performance improvement
  - `ci/build`: CI/build pipeline changes
- Scope:
  - Prefer top-level module/folder name (e.g., `auth`, `api`, `excel`, `infra`, `cms`)
  - If unclear, omit scope.

Also summarize change intent in 2–5 bullets for yourself to craft messages:
- What changed
- Why (if inferable)
- Impact/risks

## Message Candidates (EXACTLY 3, distinct)
All candidates must:
- follow Conventional Commits: `type(scope): summary` or `type: summary`
- subject <= 72 chars
- be based on staged diff
- avoid vague "update", "misc", "changes"

Candidate strategies:
1) Conservative: broad, safe, minimal claim.
2) Balanced: most accurate type/scope; includes 1–3 bullet body if meaningful.
3) Specific: mentions key behavior change or important component; may include concise body.

## User Interaction
- Show the 3 options clearly numbered (1/2/3).
- Ask: "Choose 1, 2, or 3 (or say 'cancel')."
- Do not commit until explicit selection.

## Commit Execution
When user chooses:
- If subject only:
  - `git commit -m "<subject>"`
- If subject + body:
  - `git commit -m "<subject>" -m "<body lines...>"`
Then display:
- `git log -1 --oneline`
- `git status -sb`

## Output Format to User
- First show the staged summary (`git diff --cached --stat`).
- Then list 3 commit message candidates.
- Then ask for selection.

## Commands (canonical order)
1) `git status -sb`
2) `git status --porcelain=v1`
3) (safety checks; if needed ask user and stop)
4) `git add -A`
5) `git diff --cached --stat`
6) `git diff --cached`
7) propose 3 messages + ask selection
8) `git commit ...`
9) `git log -1 --oneline`
10) `git status -sb`
