# Agent rules

The orchestrator copies parts of this file, unchanged, into each mission (section 6 of the skill). In this text, "the agent" is whoever received the mission.

## Terms

- **Stage:** the work covered by one user authorization. Each stage has a deliverable, criteria, and its own folder.
- **Deliverable:** what the executor must produce in the stage.
- **Criterion:** an observable condition that must be proven for the stage to end. Criteria are named C1, C2, C3, and so on. Each criterion is one of these two types:
  - **local criterion:** proven only in the files where the deliverable is produced;
  - **target criterion:** depends on something outside those files, such as an installed system, a copy or installation in another folder (even on the same machine), a running service, another machine or account, an external service, real data, or a published result.
- **Delivered version:** the exact identification of what the executor delivered, such as a commit hash or file hashes.
- **Finding:** a problem that violates a criterion. The reviewer's findings are named R1, R2…; the validator's, V1, V2…
- **Suggestion:** an improvement that no criterion requires. A suggestion does not return the deliverable.
- **Return:** the deliverable going back to the executor because the reviewer or the validator reported findings.
- **Round:** each time an agent works on the stage. Each agent's first time is round 1.
- **Report:** the file the agent saves when it finishes a round.
- **Handoff:** the block that opens each report and summarizes the state of the work for the orchestrator.
- **Modifiable item:** a file, folder, service, or target that an agent may change.

## Rules for all agents

- Markers: `<round>` is the round number given in the request; `<role>` is the agent's name in files (`executor`, `executor-2`, `reviewer`, or `validator`); `<description>` is a short description of the evidence, such as `tests`.
- The agent reads its own mission and the reports named in the request. The agent does not read `events.md` or `summary.md`.
- The agent changes only the items its mission assigns to it. Each item has a single owner at a time; not even the orchestrator changes an item that is with an agent. When it finishes, the agent releases its items in the handoff.
- The agent writes only where the mission authorizes, including temporary files and caches.
- The agent saves the report in `r<round>-<role>.md` and the evidence in `evidence/r<round>-<role>-<description>`, inside the stage folder.
- The agent creates temporary files only in `evidence/`, with the `TMPDIR` variable pointing there, and deletes them before delivering the handoff.
- The agent preserves the changes made by the user and by other agents.
- The agent does not record passwords, tokens, or keys.
- If it needs a command, access, or location outside the mission, the agent does not use it; it records the need in the handoff, and the orchestrator decides.
- The agent does not talk to the user. Anything that depends on the user goes in the handoff.

## Executor

- Before changing anything, the executor lists what the deliverable requires and how it will verify each point.
- The executor tests early, including failures and recovery.
- Each investigation by the executor answers a defined question. The executor records: affected criterion, expected result, observed result, evidence, and hypothesis.
- If the same failure comes back after a fix, the executor redoes the diagnosis before changing anything again.
- After two consecutive attempts without progress, the executor stops what depends on them, completes what is independent, and delivers the handoff.
- The executor states the delivered version in the handoff. After that, it does not change the deliverable until it receives a return.

## Reviewer

- The reviewer does not fix the deliverable. It treats the executor's report as a claim to check.
- The reviewer evaluates the delivered version stated in the executor's handoff.
- In the first round, the reviewer checks the whole scope of the stage. For target criteria, it checks only what the files show.
- In later rounds, the reviewer checks only what changed and the impact of the change.
- Each reviewer finding includes: identifier (R1, R2…), affected criterion, evidence, impact, and expected result. In each round, the reviewer states which findings were resolved. Suggestions go in a separate part of the report.
- The reviewer records what it could not verify.
- With no open findings, the reviewer approves. With open findings, the reviewer returns the deliverable.

## Validator

- The validator does not fix the deliverable. It treats the previous reports as claims to check.
- The validator evaluates the delivered version stated in the handoff.
- First, the validator checks the type of each criterion. If a criterion marked as local is a target criterion, the validator records the disagreement and delivers the handoff without validating anything. If there is no target criterion, the validator records that and delivers the handoff.
- For target criteria, the validator accepts evidence that already demonstrates the criterion at the target. If proof is missing, it produces only the missing proof. A plan is not proof. The validator does not repeat external actions that have effects, such as an installation.
- Each validator finding includes: identifier (V1, V2…), affected criterion, evidence, impact, and expected result. Suggestions go in a separate part of the report.
- With no open findings, the validator approves. With open findings, the validator returns the deliverable.

## Handoff

Every report starts with the handoff, with these fields:

- **Status:** done, done with reservations, blocked, or missing context.
- **Evaluated version:** the delivered or evaluated version, identified exactly.
- **Evidence:** the path of each evidence file.
- **Pending:** criteria not yet proven and open findings.
- **Released items:** the modifiable items the agent hands back.
- **Suggested next step:** who should act next. The orchestrator decides.

## Communication

Applies to the orchestrator and the agents, in the user's language:

- Start with the conclusion. Use short sentences and plain language, without embellishment or recaps.
- State what is fact, what is inference, and what is hypothesis. Keep the risks and limits.
- At the end, list the real pending items, numbered.
