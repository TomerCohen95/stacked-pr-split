# stacked-pr-split

[עברית](README-he.md)

Split a large change into small, dependent pull requests with your coding agent. Each PR focuses on one part of the change and includes a short Before / After explanation for reviewers.

**[Download the skill as a ZIP](https://github.com/TomerCohen95/stacked-pr-split/releases/latest/download/stacked-pr-split.zip)**

## What it does

- Reads the change and proposes layers: foundations, core logic, integration, and UI.
- Presents the split plan and waits for approval before building the stack.
- Builds a stack with `gh stack`, with each PR based on the one below it.
- Adds a Before / After explanation and review guidance to each PR.
- Falls back to a clean commit series or dependent PRs when stacks are unavailable.

## Install

Download and extract the ZIP linked above. It contains a folder named `stacked-pr-split`.

Copy that folder to your agent's skills directory:

| Agent | Folder location |
| --- | --- |
| Claude Code, all projects | `~/.claude/skills/stacked-pr-split/` |
| Claude Code, one project | `.claude/skills/stacked-pr-split/` |
| GitHub Copilot, one project | `.github/skills/stacked-pr-split/` |
| Another agent with skill support | Its skills directory, in a folder named `stacked-pr-split` |

Make sure `SKILL.md` sits directly inside the `stacked-pr-split` folder and your agent recognizes the skill. Depending on the agent, you may need to start a new session or reload its skills.

For GitHub stacks, you need GitHub CLI 2.90.0 or later, Git 2.20 or later, push access to the repository, and the `gh-stack` extension:

```bash
gh auth login
gh extension install github/gh-stack
```

You can also clone the repository directly into your skills directory. For example, for a personal Claude Code installation:

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/TomerCohen95/stacked-pr-split.git ~/.claude/skills/stacked-pr-split
```

## Use

Open your code project in your agent and ask:

> Use stacked-pr-split to split the changes on my current branch into small PRs. Show me the plan before making changes.

The skill creates a backup branch, proposes a split, and waits for your approval before building the stack. See [SKILL.md](SKILL.md) for the full workflow.

## Docs

- [GitHub stacked PR quickstart](https://docs.github.com/en/pull-requests/get-started/stacked-prs-quickstart)
- [Claude Code skills](https://code.claude.com/docs/en/skills)
- [GitHub Copilot skills](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills)
