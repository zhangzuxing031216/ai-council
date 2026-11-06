# AI-INSTALL.md · Installation Protocol for AI

> **Language / 语言**: [中文](AI-INSTALL.md) | English

> Goal: install this repository's `/council` command and 5+1 agents into the user's OpenCode configuration, and verify that they run.
> Applies to: Windows / Linux / macOS.
> Principle: gather evidence → back up → copy → configure models → verify → deliver; never overwrite an existing file of the same name without backing it up.

---

## 0. Your role

You are the install executor. You must:

1. **Gather evidence first**: OpenCode version, config directory, existing files, available models.
2. **One step at a time**: execute → paste the raw output → judge → next step.
3. **Do not fabricate**: do not call a command successful if you did not run it; do not call output normal if you did not see it.
4. **Do not overwrite**: back up files of the same name first.
5. **Do not guess models**: `provider/model` must come from `opencode models` or from explicit user input.
6. **Do not touch privacy**: do not write keys, tokens, or user data into the repository or logs.
7. **Keep it rollback-able**: leave a recovery method for every step.

---

## 1. Clarify 3 things before starting

If you can find it out from the system, do not ask.

1. Where is the OpenCode config directory? Default:
   - Linux/macOS: `~/.config/opencode/`
   - Windows: `%USERPROFILE%\.config\opencode\`
2. Which models does the user have available? Run `opencode models`, or ask the user for `provider/model`.
3. Do `commands/council.md` / `agents/council-*.md` already exist? If so, back them up first.

> If the user says "just set it up however", first get `quick` working with the current default model, then suggest a multi-model setup; do not blindly fill in model IDs that do not exist.

---

## 2. Phase 0 · Evidence

### 2.1 Version and directories

```bash
opencode --version
```

Windows PowerShell:

```powershell
opencode --version
$dst = "$env:USERPROFILE\.config\opencode"
Test-Path $dst
Get-ChildItem "$dst\commands","$dst\agents" -ErrorAction SilentlyContinue
```

Linux/macOS:

```bash
ls -la ~/.config/opencode/
ls -la ~/.config/opencode/commands/ 2>/dev/null
ls -la ~/.config/opencode/agents/ 2>/dev/null
```

### 2.2 Models

```bash
opencode models
```

Record the model IDs the user wants to use. **Do not** use the example names from this repository's README; they are not real models.

### 2.3 Existing files

```bash
# Linux/macOS
test -f ~/.config/opencode/commands/council.md && echo EXISTS || echo MISSING
ls ~/.config/opencode/agents/council-*.md 2>/dev/null || true
```

```powershell
# Windows
Test-Path "$dst\commands\council.md"
Get-ChildItem "$dst\agents\council-*.md" -ErrorAction SilentlyContinue
```

### 2.4 Decision

| Result | Branch |
|---|---|
| No config directory | Create it |
| A file of the same name already exists | Back up, then overwrite |
| No `opencode` command | Install OpenCode first, stop |
| No available models | Configure a provider first, stop |
| An older council version exists | Back up, preserve the user's customizations |

---

## 3. Phase 1 · Backup

```bash
STAMP=$(date +%F-%H%M%S)
mkdir -p ~/.config/opencode/backups
cp ~/.config/opencode/commands/council.md \
   ~/.config/opencode/backups/council.md.$STAMP 2>/dev/null || true
cp ~/.config/opencode/agents/council-*.md \
   ~/.config/opencode/backups/ 2>/dev/null || true
```

Windows:

```powershell
$stamp = Get-Date -Format 'yyyyMMdd-HHmmss'
$bak = "$dst\backups\$stamp"
New-Item -ItemType Directory -Force $bak | Out-Null
Copy-Item "$dst\commands\council.md" $bak -ErrorAction SilentlyContinue
Copy-Item "$dst\agents\council-*.md" $bak -ErrorAction SilentlyContinue
```

> Do not commit the backup directory to Git.

---

## 4. Phase 2 · Copy the files

### 4.1 Linux/macOS

```bash
mkdir -p ~/.config/opencode/commands ~/.config/opencode/agents
cp command/council.md ~/.config/opencode/commands/council.md
cp agents/council-architect.md \
   agents/council-coder.md \
   agents/council-critic.md \
   agents/council-vision.md \
   agents/council-scholar.md \
   agents/exec-verifier.md \
   ~/.config/opencode/agents/
```

### 4.2 Windows PowerShell

```powershell
$dst = "$env:USERPROFILE\.config\opencode"
New-Item -ItemType Directory -Force "$dst\commands","$dst\agents" | Out-Null
Copy-Item .\command\council.md "$dst\commands\council.md" -Force
Copy-Item .\agents\council-architect.md,
          .\agents\council-coder.md,
          .\agents\council-critic.md,
          .\agents\council-vision.md,
          .\agents\council-scholar.md,
          .\agents\exec-verifier.md `
          "$dst\agents\" -Force
```

### 4.3 Verify the files

```bash
ls -l ~/.config/opencode/commands/council.md
ls -l ~/.config/opencode/agents/council-*.md
```

Expected: 1 command + 5 council agents + 1 verifier.

---

## 5. Phase 3 · Configure models

### 5.1 Get it running first; it works without configuration

In the agent frontmatter, `model` is commented out and inherits the current model. Verify the pipeline first, then configure multiple models.

### 5.2 Configure one seat

Edit `~/.config/opencode/agents/council-architect.md`:

```yaml
---
description: ...
mode: subagent
model: your-provider/your-model
permissions:
  ...
---
```

Replace `your-provider/your-model` with a real ID from `opencode models`.

### 5.3 Configure all of them

| Seat | File | Suggestion |
|---|---|---|
| Architect | `council-architect.md` | Strong reasoning / long context |
| Engineer | `council-coder.md` | Strong at coding |
| Critic | `council-critic.md` | A different vendor |
| Vision | `council-vision.md` | Multimodal |
| Scholar | `council-scholar.md` | Chinese / long context |
| Verifier | `exec-verifier.md` | Strict, cheap, stable |

### 5.4 What not to do

- Do not fill in model IDs that do not exist.
- Do not write keys into agent files.
- Do not give every seat the same model and still expect multiple perspectives.
- Do not assign specialist scripted tasks to a "cheap model".

### 5.5 Where model IDs come from

```bash
opencode models
```

If the user says "use my current default model", leave it commented out and fill in nothing.

---

## 6. Phase 4 · Restart and load

```bash
opencode service restart
```

On Windows, if there is no `service` subcommand, exit and reopen OpenCode.

Verify that `/council` appears in the command list:

```text
/council quick test
```

If the command does not appear:

1. The filename must be `council.md`.
2. The directory must be `commands/`, not `command/`.
3. Restart and try again.
4. Check the OpenCode logs / config syntax.

---

## 7. Phase 5 · Smoke test

### 7.1 quick

```text
/council quick Explain in one sentence whether this command works correctly.
```

Expected:

- Subagent invocations appear;
- A "seat / core claim" table appears at the end;
- An archive path is given (or it explicitly states that nothing was written to disk).

### 7.2 Default

```text
/council What are the first three things this project should do? Give priorities and risks.
```

Expected:

- Proposals from multiple seats;
- Anonymous peer review;
- A weighted conclusion;
- Disagreements / risks / acceptance criteria.

### 7.3 With an image (optional)

```text
/council Look at this image and point out color and hierarchy problems. [image]
```

Expect the vision seat to read the image; when there is no image, it breaks down the requirements.

### 7.4 Failure branches

| Symptom | Action |
|---|---|
| No `/council` | Check the path / restart |
| Command exists but no subagents | Check the agents files |
| Model does not exist | Edit the frontmatter or comment it out |
| Everything times out | Switch models / reduce seats / run `quick` first |
| Output has no table | Emphasize the template in the command; or switch to a stronger model |
| Subagents modify files | Check that `permissions` is set to deny |
| Archiving fails | Change `COUNCIL_DIR` to a writable directory |

---

## 8. Phase 6 · Acceptance

### 8.1 Files

```bash
test -f ~/.config/opencode/commands/council.md && echo COMMAND_OK
for f in architect coder critic vision scholar; do
  test -f ~/.config/opencode/agents/council-$f.md && echo "AGENT_OK $f"
done
test -f ~/.config/opencode/agents/exec-verifier.md && echo VERIFIER_OK
```

### 8.2 Configuration

- The `model` line either does not exist or holds a real ID.
- `mode: subagent` is present.
- `permissions` does not open up edit/shell (seats are read-only).
- The command frontmatter contains no nonexistent model.

### 8.3 Run

```text
/council quick Output a single line OK and say which seat you used.
```

### 8.4 Delivery report

```text
[Conclusion] Success / Partial success / Failure

[Environment]
- OS:
- OpenCode version:
- Config directory:

[Installation]
- command: path
- agents: 5 + verifier
- Backup: path

[Models]
- Architect:
- Engineer:
- Critic:
- Vision:
- Scholar:
- Verifier:

[Verification]
- /council quick: pass/fail
- Default mode: pass/fail
- Archive: path

[Unresolved / Risks]
- ...

[Rollback]
- Commands to restore the backup
```

---

## 9. Rollback

```bash
# Remove the new files
rm -f ~/.config/opencode/commands/council.md
rm -f ~/.config/opencode/agents/council-architect.md \
      ~/.config/opencode/agents/council-coder.md \
      ~/.config/opencode/agents/council-critic.md \
      ~/.config/opencode/agents/council-vision.md \
      ~/.config/opencode/agents/council-scholar.md \
      ~/.config/opencode/agents/exec-verifier.md

# Restore the old files from the backup
cp ~/.config/opencode/backups/council.md.<stamp> ~/.config/opencode/commands/council.md
```

Windows:

```powershell
Remove-Item "$dst\commands\council.md" -Force
Remove-Item "$dst\agents\council-*.md" -Force
Copy-Item "$bak\*" "$dst\commands\" -Force -ErrorAction SilentlyContinue
```

> Restart OpenCode after rolling back.

---

## 10. Prohibitions

1. Do not overwrite an existing user agent without backing it up.
2. Do not fill in model IDs that do not exist.
3. Do not write API keys into agents/commands/README.
4. Do not bring the user's private provider names or internal paths into a public repository.
5. Do not promise that "the council will definitely give the right answer".
6. Do not report "installation successful" before verifying.
7. Do not help the user use the council to make illegal or harmful decisions.

---

## 11. Quick command card

```bash
# Evidence
opencode --version
opencode models
ls ~/.config/opencode/commands ~/.config/opencode/agents

# Install
cp command/council.md ~/.config/opencode/commands/
cp agents/council-*.md agents/exec-verifier.md ~/.config/opencode/agents/

# Restart
opencode service restart

# Verify
/council quick test
```