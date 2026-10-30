# LLMOps Home Lab

Running a large language model like any other production service: deployed, monitored, secured, and tested.

This repo documents my move from DevOps / SRE / security into **AI Platform and LLMOps engineering**, built one stage at a time on a home lab (Ubuntu + NVIDIA GPU) with real configs, runbooks and results.

> **Why this repo exists:** most AI demos stop at "it answers a question." Production needs more: uptime, latency SLOs, GPU capacity, access control, audit logs, and automated checks that the *answers* are still good. That's the gap this lab works through.

---

## Progress

| Stage | Topic | Status | Proof |
|---|---|---|---|
| 0 | Secure dev environment (multi-account Git over SSH) | 🟡 In progress | [Stage 0](#stage-0--secure-dev-environment) |
| 1 | AI fundamentals + Python for ops | 🟡 In progress | [Stage 1](#stage-1--ai-fundamentals--python-for-ops) |
| 2 | Serve a local LLM (Ollama on GPU) | ⬜ Planned | — |
| 3 | Observability: tokens/sec, latency, GPU metrics | ⬜ Planned | — |
| 4 | Run the model on Kubernetes | ⬜ Planned | — |
| 5 | Security: auth proxy, rate limits, prompt audit log | ⬜ Planned | — |
| 6 | Evals: automated answer-quality tests in CI | ⬜ Planned | — |
| 7 | Runbooks, architecture diagram, final write-up | ⬜ Planned | — |

Legend: ✅ done · 🟡 in progress · ⬜ planned

---

## Lab environment

| Component | Detail |
|---|---|
| Workstation | macOS |
| Lab server | Ubuntu with NVIDIA GPU |
| Change policy | OS auto-update disabled; NVIDIA driver packages held (`apt-mark hold`); every install reviewed with `apt list --upgradable` first; no blanket `apt upgrade` |
| Install approach | Single binaries and containers preferred, so nothing touches the GPU driver |
| Editor | vi |

---

## Stage 0 — Secure dev environment

**Goal:** work with two GitHub accounts (learning and professional) from multiple machines without ever pushing to the wrong account or leaking a personal email.

**What I built**
- One SSH key **per account, per device**. No private key is ever copied between machines. If a device is lost, I revoke only that device's keys.
- SSH host aliases so each account always uses its own key:

```
Host github.com github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal
    IdentitiesOnly yes

Host github-pro
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_pro
    IdentitiesOnly yes
```

- A **repo-scoped deploy key** for an automation repo, giving access to one repository only instead of a whole account.
- Commit identity switches by folder with `includeIf`. The personal account is the global default, and everything under the professional folder is overridden with the kc-kcodes GitHub noreply email:

```ini
[user]
    name = <personal>
    email = <personal email>

# must come after [user] so it overrides it
[includeIf "gitdir:~/ckgit-prof/"]
    path = ~/.gitconfig-pro

# any kc-kcodes URL automatically uses the professional SSH key
[url "git@github-pro:kc-kcodes/"]
    insteadOf = git@github.com:kc-kcodes/
    insteadOf = https://github.com/kc-kcodes/
```

**Verification**

```bash
ssh -T git@github-personal   # authenticates as the personal account
ssh -T git@github-pro        # authenticates as kc-kcodes
git config user.email        # inside a repo under ~/ckgit-prof → kc-kcodes noreply address
```

**Checklist**
- [x] macOS: SSH aliases, per-folder identity, URL rewrite. First commit to this repo authored as kc-kcodes
- [x] Ubuntu lab: SSH aliases for both accounts (existing repo-scoped deploy key kept)
- [x] Ubuntu lab: per-folder identity (`~/.gitconfig-pro`, `includeIf`, URL rewrite), same folder and file names as macOS
- [x] Ubuntu lab: removed `credential.helper = store` (plain-text token storage) and moved owned repos from HTTPS to SSH
- [ ] VS Code: confirm commits and pushes use the right account
- [ ] IntelliJ: confirm commits and pushes use the right account

**Lessons learned**
- GitHub ignores the SSH `User` field for identity. It always expects `git` and identifies you **by the key**. A stray `User <name>` line only works by accident.
- SSH uses the **first matching value** for each option. A duplicate `Host github.com` block higher in the file silently overrides a corrected one lower down.
- `IdentitiesOnly yes` stops SSH from offering every key it can find, which is the usual cause of "pushed to the wrong account."
- Ubuntu skips private keys with loose permissions. Use `chmod 600` on keys and config, and `700` on `~/.ssh`.
- `includeIf "gitdir:..."` only applies **inside a git repo**, and the path needs a trailing `/`. Running `git config user.email` in the parent folder still shows the global value, and that's expected.
- An `includeIf` placed above `[user]` gets overridden by it. Git applies settings top to bottom, so the include goes last.
- A repo's own `.git/config` beats the global `includeIf`. If a repo has a local `user.email`, it quietly wins. Find the source with `git config --show-origin user.email` and remove it with `git config --local --unset user.email`.
- `credential.helper = store` writes GitHub tokens to `~/.git-credentials` in plain text. Rule of thumb: repos you **push** to use SSH; repos you only **pull** from (public third-party code) can stay on HTTPS with no stored credentials.
- Before changing anything, audit first: list every local repo's remote (`find ~ -path "*/.git/config" -exec grep -H "url =" {} \;`) so a cleanup doesn't break existing work.
- Claude Code runs on the lab server; IntelliJ on the Mac pulls changes via Git.

**Security takeaways:** least privilege (deploy keys), per-device credentials (simple revocation), no personal data in public commit history.

---

## Stage 1 — AI fundamentals + Python for ops

**Goal:** speak the vocabulary and read and modify the Python that runs AI infrastructure.

- [ ] AI vocabulary with ops examples: tokens, context window, temperature, RAG, agents, MCP
- [ ] Anthropic Academy: Claude Code course
- [ ] freeCodeCamp Python (v9): loops and sequences
- [ ] Ops script: service health check with retries (`for`, `while`, `break`, `continue`)
- [ ] Ops script: log scanner (slicing, `enumerate`, `.splitlines()`)

*Notes and scripts will be added here as each item is completed.*

---

## Stages 2–7

Each stage is filled in as it's completed, using the same structure: **Goal → What I built → Verification → Lessons learned**.

---

## About

DevOps / SRE engineer with ~20 years across AWS, CI/CD, Kubernetes and identity governance, now focused on operating AI systems safely in production.
