---
name: hopper-orchestration-anth
description: Coordinates one work stage with three Claude subagents (executor, reviewer, and validator), each with the model and effort the user chooses, until every criterion is proven by evidence. Use when the user invokes this skill to orchestrate a deliverable with agents.
compatibility: Requires Claude Code with the hopper-orchestration-anth plugin, which provides the subagents, and the Agent, SendMessage, and AskUserQuestion tools.
disable-model-invocation: true
---

# hopper-orchestration-anth

The reader of this file is the **orchestrator**: the Claude session in which the user invoked the skill. Agents do not read this file. Each agent reads only its own mission, which the orchestrator writes. The terms and rules that go into missions are in `${CLAUDE_SKILL_DIR}/agent-rules.md`; the orchestrator reads that file in full before the preparation.

## 1. Who takes part

| Participant | Who it is | What it does | Name in files |
| --- | --- | --- | --- |
| User | The person who invoked the skill | Authorizes the stage, chooses models and efforts, and makes the decisions outside the authorization | — |
| Orchestrator | The Claude session that reads this skill | Defines the stage, writes the missions, calls the agents, decides returns, pauses, and acceptance, and talks to the user. Does not produce, review, or validate | `orchestrator` |
| Executor | Subagent | Produces the deliverable, verifies it, and fixes it | `executor`, `executor-2`… |
| Reviewer | Subagent | Checks the deliverable in the files: requirements, errors, and omissions | `reviewer` |
| Validator | Subagent | Checks the type of each criterion and proves the target criteria | `validator` |

Agents do not talk to the user or to each other. Everything goes through the orchestrator.

## 2. Markers and files

Text between `< >` is a marker: the orchestrator replaces it with the real value.

| Marker | Meaning | Example |
| --- | --- | --- |
| `<role>` | Agent name in files | `reviewer` |
| `<effort>` | `high`, `xhigh`, or `max` | `xhigh` |
| `<model>` | `opus`, `sonnet`, or `fable` | `sonnet` |
| `<round>` | Round number | `2` |
| `<topic>` | Stage topic, in up to three hyphen-separated words | `login-fix` |
| `<stage-folder>` | `docs-by-hopper-orchestration/<YYYYMMDD-HHMMSS>-<topic>/`, at the project root, with the date and time the stage started | — |
| `<description>` | Short description of a piece of evidence | `tests` |
| `<agent-id>` | Identifier the Agent tool returns when it starts the agent | — |
| `<project>` and `<session>` | Project folder and orchestrator session under `~/.claude/projects/` | — |

Files of each stage, all inside `<stage-folder>`:

| File | Who writes | Who reads | Content |
| --- | --- | --- | --- |
| `mission-<role>.md` | Orchestrator | The agent in that role | The agent's mission (section 6) |
| `r<round>-<role>.md` | The agent | Orchestrator and the following agents | The round report, which starts with the handoff |
| `evidence/r<round>-<role>-<description>` | The agent | Orchestrator and the following agents | Command output and other proof |
| `events.md` | Orchestrator | Orchestrator and user | One line per stage event (section 10) |
| `summary.md` | Orchestrator | User and the next stage | The stage result (section 10) |

## 3. When the orchestrator uses this skill

Only when the user invokes the skill. Explaining, reviewing, or planning with the skill does not start orchestration.

## 4. Preparation: what the orchestrator does before starting agents

1. **Reads the previous stage.** If there is one, the orchestrator reads its `summary.md` and does not redo work already accepted.
2. **Defines the stage:** deliverable, criteria with the type of each one, limits, and the owner of each modifiable item.
3. **Checks its own tools:** Write, Bash, Agent, and SendMessage, loading the deferred ones. If any is missing, it tells the user and stops.
4. **Checks the session's permission mode.** Agents inherit this mode. The orchestrator recommends auto mode, which reviews the agents' actions, or the sandbox. If the session runs without permissions (bypass), it warns the user before starting agents and follows the user's decision.
5. **Sizes the team.** The reviewer and the validator leave the team only if the user waives them. If the team seems too large or too small for the deliverable, the orchestrator recommends another one to the user, for example without a validator when there is no target criterion. More than one executor only for independent parts. In all of this, the user's decision prevails.
6. **Shows the panel.** Before any call to the Agent tool, the orchestrator asks the user, for each role on the team, in the order executor, reviewer, and validator: first the model, then the effort. It asks one question at a time, with clickable options (AskUserQuestion tool). Without that tool, the orchestrator shows the options numbered and asks for the number. The user's previous choice appears first. The executor's choice applies to all executors.
   - Models: Opus (`opus`), Sonnet (`sonnet`), and Fable (`fable`).
   - Efforts: High (`high`), xHigh (`xhigh`), and Max (`max`).
7. **Creates the stage folder and writes one mission per agent** (section 6).
8. **Starts the executor** (section 5).

Preparing does not authorize executing. No plan or handoff expands what the user authorized.

## 5. How the orchestrator starts and resumes agents

The plugin provides one subagent definition per role and effort, with the type `hopper-orchestration-anth:<role>-<effort>` (example: `hopper-orchestration-anth:reviewer-xhigh`). The definition fixes the effort and the tools:

- executor: Read, Write, Edit, Bash, Grep, and Glob;
- reviewer and validator: Read, Write, Bash, Grep, and Glob;
- no agent has Agent or Skill: agents do not create agents or use skills.

**To start an agent,** the orchestrator:

1. calls the Agent tool with the agent type, `model` equal to the chosen `<model>`, and background execution;
2. sends only this request: "Read and carry out `<stage-folder>/mission-<role>.md`. Round `<round>`.", plus the path of the latest report, when there is one. Nothing from the conversation with the user goes into the request;
3. waits for the completion notice, without checking on the agent while it works;
4. checks the model and effort used in the agent's transcript, `~/.claude/projects/<project>/<session>/subagents/agent-<agent-id>.jsonl`, in the `"model"` and `"effort"` fields. If it cannot confirm them, it records that and tells the user.

**To resume the same agent** for a correction, the orchestrator calls the SendMessage tool with the `<agent-id>` and gives the new round and the path of the report that prompted the resumption. The resumed agent keeps its history and model. If resumption fails, the orchestrator starts another agent in the same role, with the mission and the latest report.

**The orchestrator does not:**

- use agents for its own work: it writes the stage folder, the missions, `events.md`, and `summary.md` itself;
- start agents from the terminal or create programs to control them;
- change the model or effort on its own.

## 6. How the orchestrator writes the mission

The mission is the only text the agent receives; the agent does not see the conversation with the user, so that review and validation do not inherit conclusions. The orchestrator saves each mission in `<stage-folder>/mission-<role>.md` with these items, in this order:

1. the agent's role and its name in files (`executor`, `executor-2`, `reviewer`, or `validator`);
2. objective, deliverable, and criteria, each criterion with its type;
3. the stage folder, the specification, the team roles, and the decisions already made, with their reasons;
4. the user's and the project's instructions that apply to the stage, including the language;
5. the accesses, the limits, and the modifiable items owned by the agent; with more than one executor, which one joins the parts;
6. copied unchanged from `agent-rules.md`: the parts "Terms", "Rules for all agents", the part for the agent's role, "Handoff", and "Communication".

If a decision changes criteria, the type of a criterion, or the limits, the orchestrator updates the affected missions and records the decision, with its reason, in item 3 of each one.

## 7. What the orchestrator does when it receives a handoff

The orchestrator checks that the handoff has every field of the "Handoff" part of `agent-rules.md`. If any is missing, it asks the same agent to complete it; this does not count as a return. Then it follows the table below. In it, "calls" means starting the agent in round 1 and resuming the same agent in later rounds.

| Situation | What the orchestrator does |
| --- | --- |
| One executor finished its part, and another executor joins the parts | Calls the executor that joins the parts |
| The executor finished the deliverable | Calls the reviewer |
| The reviewer approved, and the team has a validator | Calls the validator |
| The reviewer approved, and the team has no validator | Accepts the stage (section 9) |
| The validator approved | Accepts the stage (section 9) |
| The reviewer or the validator returned the deliverable | Handles the return (section 8) |
| The validator disagreed with the type of a criterion | Decides the type, records it, updates the missions, and resumes the validator with only the target criteria not yet checked |
| The agent is blocked, needs context, or finished with reservations | Resolves it, if it is within the authorization; otherwise, pauses and takes it to the user (section 9) |

## 8. How the orchestrator handles returns

The orchestrator counts the stage's returns and acts as follows:

- **First return:** resumes the executor and asks it to fix the findings in the report that returned the deliverable.
- **Second return:** before resuming the executor, defines the exact criterion the next round must meet and includes it in the request.
- **Third return onward:** compares the total of open findings, the reviewer's and the validator's combined, with that of the previous return. If it decreased, records the decision to continue, defines the exact criterion, and resumes the executor. If it did not decrease, pauses and takes to the user the history, the likely cause, and a recommendation.
- **The executor delivers again, unchanged, the version already returned:** the orchestrator does not call the reviewer, because there is no change to check. It counts a new return, with the same open findings, and follows the rules above.
- **The same impediment again, with nothing changed:** pauses and takes it to the user.

The orchestrator records each decision in `events.md`, with its reason. It sets no deadline. It does not accept or cancel the stage just because failures repeat.

## 9. Acceptance, pause, and closing

- **Acceptance:** the orchestrator accepts the stage when every criterion has evidence from the delivered version and there is no open finding. Suggestions do not block acceptance.
- **Pause:** when the stage depends on a user decision, the orchestrator stops and gives the user the partial result, the impediment, and the recommended next action. The modifiable items stay with their owners. With the user's answer, the orchestrator resumes the stage with the same agents.
- **Closing:** after acceptance, or when the user decides to close the stage, the orchestrator writes `summary.md`, releases the modifiable items, and deletes the temporary files it created itself.

## 10. How the orchestrator records the stage

**`events.md`:** the orchestrator writes one line per event, with the date, time, and time zone read at the moment of recording. Events: stage defined; team chosen; agent started or resumed, with type, model, and `<agent-id>`; agent finished, with confirmed model and effort; handoff received; return; decision, with its reason; pause; acceptance. On each line, the orchestrator states whether the action was decided, attempted, or done, and records as done only what it confirmed. It also records failures.

**`summary.md`:** when closing the stage, the orchestrator writes: result; delivered version; each criterion with the path to its evidence; team, with role, model, and effort; user decisions, in the user's words; limitations; pending items; next step.

After an interruption or compaction, the orchestrator rereads this skill and continues from the last confirmed event in `events.md`.

## 11. How the orchestrator talks to the user

The orchestrator follows the "Communication" part of `agent-rules.md`. In addition, it:

- reports progress only when the result, the impediment, or the needed decision changes;
- does not ask again for what the user already confirmed, except the panel.
