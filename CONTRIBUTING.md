# Contributing to hopper-orchestration-anth

Contributions can improve the skill's clarity, identify workflow failures, verify behavior in Claude Code, or improve the documentation. English documentation is in `hopper-orchestration-anth-en/` and Brazilian Portuguese documentation is in `hopper-orchestration-anth-pt-br/`. Shared GitHub policies and templates remain at the repository root. Skill bodies are in `hopper-orchestration-anth-en/skills/hopper-orchestration-anth/` and `hopper-orchestration-anth-pt-br/skills/hopper-orchestration-anth/`. Each language folder is a complete plugin: the skill, the agent rules, and nine agent definitions. Keep both versions equivalent; one release covers both languages.

## Start with the problem

For suspected vulnerabilities, follow the [Security Policy](SECURITY.md) and report privately. Do not include security-sensitive details in public issues, discussions, or pull requests before coordinating disclosure.

Use a bug report for a reproducible failure and a proposal for a specific change. Use [Discussions](https://github.com/jonathanpah/hopper-orchestration-anth/discussions) for questions or early ideas. Search existing issues before opening another report about the same behavior.

For changes to orchestration policy, explain the current rule, the problem it creates, the desired behavior, evidence, and tradeoffs. A proposal is not approval to change the project's policy. Small documentation corrections can go directly to a pull request.

## Preserve the design

- Keep the skill focused on what to do. Put installation commands and details in the README.
- Preserve explicit activation, user-defined team configuration, scope boundaries, modifiable-item ownership, evidence requirements, and recorded decisions unless the maintainer approves a policy change.
- Keep the plugin self-contained. `SKILL.md`, the agent rules, and the agent definitions must not depend on a separate `AGENTS.md` or this contribution guide to perform the workflow.
- Keep the skill, the agent rules, and the nine agent definitions equivalent in both languages. Update both languages together when changing a rule. Missions copy parts of the agent rules unchanged, so keep their headings stable.
- Do not add fixed model versions, arbitrary duration limits, or consumption caps as incidental edits.
- Distinguish observed behavior from documentation claims, assumptions, and proposed behavior. Do not claim performance or cost improvements without comparable measurements.
- Use plain language in English and Portuguese. Do not add comments to code examples or configuration files, or hidden HTML comments to Markdown. Put explanations in visible documentation.

## Open a pull request

1. Fork the repository to your GitHub account. A fork is your own copy that you can edit.
2. Create a branch in your fork for one coherent change.
3. Edit the files and commit the change with a descriptive message.
4. Open a pull request from your branch to this repository's `main` branch. Link the relevant issue, if one exists.
5. Explain the problem, resulting behavior, verification, and limitations. Respond to review on the same branch.

A pull request proposes a change; it does not change this repository until the maintainer merges it. You can also contribute by reviewing proposals, reproducing reported failures, or supplying evidence without changing files.

## Verify proportionately

Check that the changed instructions remain coherent, links resolve, and YAML or JSON examples parse. Run `claude plugin validate --strict` on the changed language folder. For a translation or wording change, compare the meaning and obligations before and after. A wording-only correction does not require rerunning an unaffected orchestration.

For workflow changes, record a small relevant scenario and its observable outcome. Useful cases include a request that must not activate the skill, a missing tool, a target criterion marked as local, a correction returned by the reviewer, and an unchanged version delivered again. Select cases affected by your change; do not run unrelated work just to complete a checklist.

For a compatibility claim, identify the Claude Code version (CLI or app), the operating system, the requested and confirmed models and efforts, what you observed, and limitations. Keep separate claims for parsing, discovery, invocation policy, and actual workflow behavior. Passing one does not establish the others.

Use disposable resources for experiments. Share a concise, redacted reproduction or synthetic example. Never submit passwords, access tokens, private keys, personal data, private chat history, or unredacted production records.

## Review and releases

Jonathan Honorio maintains the project and decides whether a change is accepted. Reviews consider the stated problem, evidence, consistency with the design, and maintenance cost. Review does not imply a promised response time or acceptance.

Accepted changes are merged into `main`. A release identifies a selected repository version and describes its changes and known limitations. Installing or updating remains a user's decision; an open pull request or discussion is not a released change.

By submitting a contribution, you agree that your contribution is provided under this repository's [MIT License](LICENSE). Follow the [Code of Conduct](CODE_OF_CONDUCT.md).
