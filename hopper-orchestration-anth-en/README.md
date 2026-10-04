# hopper-orchestration-anth

**English** | [Português (Brasil)](../hopper-orchestration-anth-pt-br/README.md)

A Claude Code skill for coordinating one work stage with three Claude subagents (executor, reviewer, and validator) until every criterion is proven by evidence.

The orchestrator is the Claude session in which the user invokes the skill. It defines the stage, writes the missions, calls the agents, and decides returns, pauses, and acceptance. The executor produces the deliverable, the reviewer checks it in the files, and the validator proves the criteria that depend on something outside the deliverable, such as an installation or a running service. The workflow starts only when the user invokes the skill.

The plugin has three parts:

- the [SKILL.md](skills/hopper-orchestration-anth/SKILL.md), read only by the orchestrator;
- the [agent rules](skills/hopper-orchestration-anth/agent-rules.md), which the orchestrator copies unchanged into each mission;
- nine [subagent definitions](agents/), one per role (executor, reviewer, and validator) and effort (High, xHigh, and Max).

Browse the [document index](index.md) or see [how to contribute](../CONTRIBUTING.md).

## What the workflow requires

- A team panel, always shown before the first agent. For the executor, the reviewer, and the validator, the user clicks the model (Opus, Sonnet, or Fable) and the effort (High, xHigh, or Max). The previous choice appears first.
- Self-contained missions written by the orchestrator. Agents do not see the conversation with the user.
- An executor, a reviewer, and a validator in every stage. The reviewer and the validator leave only if the user waives them. More than one executor only for independent parts.
- Observable criteria, classified as local or target, an identified delivered version, and a single owner per modifiable item.
- Reports that start with a handoff with fixed fields, evidence in files, and an event log with real times.
- Graduated returns: a fix, then an exact criterion and, if the findings do not decrease, a pause for the user's decision.

The orchestrator keeps the main conversation's model and effort. It checks, in each agent's transcript, the model and effort used. Agents do not create other agents or use skills. Explaining or reviewing the skill does not invoke the workflow.

## Compatibility and limits

The skill is built for Claude Code, in the CLI and the app, on macOS and Linux. It depends on the Agent, SendMessage, and AskUserQuestion tools and on the subagents the plugin provides. Outside Claude Code, the workflow is not available.

The SKILL.md follows the [Agent Skills format](https://agentskills.io/specification) and sets `disable-model-invocation: true`: Claude does not invoke the skill on its own. Each agent receives only the tools of its role and inherits the session's permission mode. The skill recommends auto mode or the sandbox.

Discovering the skill does not prove that the full workflow works in your environment. Before starting agents, the orchestrator checks its own tools and the permission mode. Missing essentials require a user decision. The skill does not claim measured cost savings.

## Install

The plugin and the skill are named `hopper-orchestration-anth` in both languages. Choose one language only. The `hopper-orchestration-anth-en/` folder contains the complete English plugin: the skill in `skills/hopper-orchestration-anth/` and the subagents in `agents/`.

Clone the repository once:

```sh
mkdir -p "$HOME/.local/share"
git clone https://github.com/jonathanpah/hopper-orchestration-anth.git "$HOME/.local/share/hopper-orchestration-anth"
```

If the destination exists, inspect its origin and local changes before updating it.

### Claude Code

```sh
claude plugin marketplace add "$HOME/.local/share/hopper-orchestration-anth/hopper-orchestration-anth-en"
claude plugin install hopper-orchestration-anth@hopper-orchestration-anth
```

In a session that is already open, send `/reload-plugins` as a separate message to load the skill and the subagents. No `skillOverrides` entry is needed: the SKILL.md itself prevents automatic invocation.

### App and names

The qualified identifier is `hopper-orchestration-anth:hopper-orchestration-anth`: the first part identifies the plugin, and the second identifies the skill. The product adds this prefix. Invoke the skill with `/hopper-orchestration-anth:hopper-orchestration-anth`. The subagents appear as `hopper-orchestration-anth:executor-high`, `hopper-orchestration-anth:reviewer-xhigh`, and so on; only the skill's orchestrator uses them.

CLI installation does not prove that the plugin is in the app catalog. In the Claude app, check **Customize → Plugins → Yours** and **Skills → Yours**. If the plugin is missing there, add the package using Claude's plugin upload option and verify both lists. The ZIP must contain the files from the chosen language folder, including `.claude-plugin/plugin.json`, `skills/`, and `agents/`. Account and local installations are separate records; avoid two competing versions.

### Verification

Check the name, version, and content with `claude plugin list` and `claude plugin details hopper-orchestration-anth@hopper-orchestration-anth`: the skill and the nine agents must appear. Check the app separately. Installation does not run agents; testing the workflow requires invoking the skill with a bounded task. The [`evals/`](evals/) folder holds the official suite cases; to repeat them, run `claude plugin eval . --tag with-write --allow-tools Write Bash SendMessage` and `claude plugin eval . --tag without-write` in this folder.

## Use

Invoke `/hopper-orchestration-anth:hopper-orchestration-anth` and describe the authorized deliverable. For example:

> Orchestrate the fix of the shipping calculation in this project. The deliverable is the fixed code, with tests, in one commit; do not publish anything.

Before agents start, the orchestrator shows the team panel. For the executor, the reviewer, and the validator, click the model and the effort.

The skill remains in English, but agents communicate in the user's language. A task plan or a handoff alone does not authorize starting orchestration or expanding its scope.

## Records

Each stage creates a subfolder `docs-by-hopper-orchestration/YYYYMMDD-HHMMSS-<topic>` at the root of the project where the orchestration was invoked. It contains the missions (`mission-<role>.md`), the round reports (`r<round>-<role>.md`), the evidence (`evidence/`), `events.md`, and a final `summary.md`.

Dates and times use the system time zone, read at the moment of each record, always with the offset from UTC.

Keep run records outside this repository when possible. Do not submit credentials, private conversation histories, personal paths, or unredacted operational evidence with a contribution.

## Updates

Inspect local changes and update the checkout with `git pull --ff-only`. Then run `claude plugin marketplace update hopper-orchestration-anth` and `claude plugin update hopper-orchestration-anth@hopper-orchestration-anth`, and send `/reload-plugins` in open sessions. If you uploaded the plugin in the app, update the account package too. Verify content and discovery again.

To remove it, use `claude plugin uninstall hopper-orchestration-anth@hopper-orchestration-anth` and remove the account installation in the app, if present. Project records are preserved.

## Contribute

For suspected vulnerabilities, follow the [Security Policy](../SECURITY.md) and report privately before sharing details publicly.

Use [Issues](https://github.com/jonathanpah/hopper-orchestration-anth/issues) for reproducible problems and concrete proposals. Use [Discussions](https://github.com/jonathanpah/hopper-orchestration-anth/discussions) for questions, experiences, and ideas that are not yet a proposed change.

To propose an edit, fork the repository, create a branch, and open a pull request. Contributors can comment and suggest changes; the maintainer decides what is merged. See [CONTRIBUTING.md](../CONTRIBUTING.md) for scope, evidence, and review expectations and the [Code of Conduct](../CODE_OF_CONDUCT.md) for participation standards.

## License and maintainer

Copyright (c) 2026 Jonathan Honorio. Released under the [MIT License](../LICENSE).

Maintained by [Jonathan Honorio (@jonathanpah)](https://github.com/jonathanpah). This is an independent project, with no claimed affiliation with or endorsement by Anthropic.
