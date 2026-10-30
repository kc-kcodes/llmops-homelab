# Daily workflow: AI agent on the lab server, IDE on the workstation

How I work with a coding agent (Claude Code) when the agent runs on a Linux lab server and the IDE runs on an older macOS workstation that can't run the agent locally.

## The picture

```
Workstation (IntelliJ)                     Lab server (Ubuntu)
──────────────────────                     ───────────────────
I type here      ── Cmd+Esc  /  SSH ──▶    agent works here
I review here    ◀────── git pull ──────   agent commits & pushes here
```

**Rule:** the agent's hands are on the lab server, my eyes are on the workstation. Git carries every change between them.

## How it's wired

- The IDE's Claude Code plugin runs a custom command instead of a local binary:

  ```bash
  ssh -t ubuntulab "cd ~/ckgit-prof/llmops-homelab && bash -lic claude"
  ```

- `ubuntulab` is an SSH host alias. No new credentials are involved.
- On the lab server, the repo lives under `~/ckgit-prof/`, so commits automatically use the professional identity (`includeIf`) and push with the professional SSH key (`github-pro`).
- The agent runs in **manual approval mode**: every file edit, command and push is approved by me.

## Daily cheat sheet

| I want to... | Do this |
|---|---|
| Start the agent | **Cmd+Esc** in IntelliJ (or the Claude icon) |
| Start from the latest code | Tell the agent: *"git pull first"* |
| Get the agent's work onto the workstation | Tell the agent: *"commit and push"*, then **Cmd+T** in IntelliJ |
| Stop the agent mid-task | **Esc** |
| Switch model | `/model` (lighter model for everyday work, stronger model for hard problems) |
| Leave the agent | `/exit`, or close the tab |

## When something looks wrong

Ask one question: **"Which machine has the latest change?"**

- The machine that **made** the change already has it, so `git pull` there says *Already up to date*.
- The **other** machine has to pull.

| Symptom | Likely cause | Fix |
|---|---|---|
| Agent's change not visible in the IDE | Workstation hasn't pulled | **Cmd+T** |
| Agent works from old code | Lab server hasn't pulled | Tell the agent *"git pull first"* |
| Merge conflict | Same file edited on both machines | Edit in one place at a time; resolve once, then commit |
| Commit shows a personal email | Repo outside `~/ckgit-prof/`, or a local `user.email` override | `git config --show-origin user.email` |
| Pull Requests panel says "Bad credentials" | IDE's GitHub **API** login is stale. Git over SSH is unaffected | Ignore, or re-add the account under Settings → Version Control → GitHub |

## Habits

1. **Pull before starting**, on both sides.
2. **One editor at a time per file**: either the agent or me, not both.
3. **Small commits with clear messages.** The history is part of the portfolio.
4. **Review every agent action.** Approve deliberately; don't enable auto-accept in the IDE.

## Why this design

| Choice | Reason |
|---|---|
| Agent on the lab server | The workstation's OS is too old for the agent CLI, and the server has the GPU and the lab workloads anyway |
| Git as the sync layer | Auditable history, no file-sharing mounts or kernel extensions on the workstation |
| SSH alias in the IDE command | Reuses existing keys; no extra secrets stored in the IDE |
| Manual approval mode | Human-in-the-loop for every action, the same standard as a production change |
