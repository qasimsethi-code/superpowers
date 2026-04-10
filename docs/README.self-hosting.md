# Self-Hosting Superpowers

Self-hosting Superpowers is particularly useful for organizations with stringent security policies that require code to be reviewed before deployment, teams that need to pin to a specific version for compliance, or developers who want to maintain a customized fork with private skills.

## What "Self-Hosting" Means Here

Superpowers is a static collection of skill files — there is no server to run. "Self-hosting" means installing from **your own fork or internal mirror** of the repository instead of the public `obra/superpowers` repository.

This gives you:
- Full control over which skills are available to your agents
- The ability to add private, organization-specific skills alongside the core skills
- A pinned, audited version of the code that only changes when you approve updates
- Air-gapped operation with no runtime dependency on GitHub

## Setup

### 1. Fork or Mirror the Repository

**Public fork (GitHub):**

Use GitHub's fork button on https://github.com/obra/superpowers. Your fork URL will be:
```
https://github.com/YOUR-ORG/superpowers
```

**Private mirror (for air-gapped environments):**

```bash
# Clone as a bare mirror
git clone --mirror https://github.com/obra/superpowers.git superpowers.git

# Push to your internal git server
cd superpowers.git
git remote set-url --push origin https://git.internal.example.com/superpowers.git
git push --mirror
```

### 2. Install from Your Fork

Replace the upstream URL with your fork URL in all installation steps below.

---

## Platform Installation

### Claude Code

In Claude Code, install directly from your fork:

```bash
/plugin install superpowers@git+https://github.com/YOUR-ORG/superpowers.git
```

To pin a specific commit or tag:

```bash
/plugin install superpowers@git+https://github.com/YOUR-ORG/superpowers.git#v5.0.7
```

For an internal git server:

```bash
/plugin install superpowers@git+https://git.internal.example.com/superpowers.git
```

### OpenCode

In your `opencode.json` (global at `~/.config/opencode/opencode.json` or project-level):

```json
{
  "plugin": ["superpowers@git+https://github.com/YOUR-ORG/superpowers.git"]
}
```

To pin a specific tag:

```json
{
  "plugin": ["superpowers@git+https://github.com/YOUR-ORG/superpowers.git#v5.0.7"]
}
```

Restart OpenCode. The plugin installs automatically on launch.

### Codex

Clone from your fork instead of the upstream:

```bash
git clone https://github.com/YOUR-ORG/superpowers.git ~/.codex/superpowers

mkdir -p ~/.agents/skills
ln -s ~/.codex/superpowers/skills ~/.agents/skills/superpowers
```

**Windows (PowerShell):**

```powershell
git clone https://github.com/YOUR-ORG/superpowers.git "$env:USERPROFILE\.codex\superpowers"

New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
cmd /c mklink /J "$env:USERPROFILE\.agents\skills\superpowers" "$env:USERPROFILE\.codex\superpowers\skills"
```

### Gemini CLI

```bash
gemini extensions install https://github.com/YOUR-ORG/superpowers
```

---

## Adding Private Skills

Once you are working from your own fork, you can add organization-specific skills directly to the `skills/` directory alongside the upstream skills.

Create a new skill directory:

```bash
mkdir skills/my-org-workflow
```

Create `skills/my-org-workflow/SKILL.md` following the same structure as existing skills. The skill will be available to all agents that have your fork installed.

Keep private skills on a separate branch if you want to pull upstream updates cleanly:

```bash
# Track upstream
git remote add upstream https://github.com/obra/superpowers.git

# Pull updates onto your main branch
git fetch upstream
git merge upstream/main

# Keep private skills on a dedicated branch merged in
git checkout private-skills
git merge main
```

---

## Staying Up to Date

Periodically pull upstream changes into your fork:

```bash
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

Review the [CHANGELOG](../CHANGELOG.md) and [RELEASE-NOTES](../RELEASE-NOTES.md) before merging to understand what changed in core skills.

After updating the fork:

- **Claude Code / OpenCode:** The plugin re-installs from your fork on next launch.
- **Codex:** `cd ~/.codex/superpowers && git pull`

---

## Verifying the Installation

Start a new agent session and ask: `"Tell me about your superpowers"`. The agent should describe the available skills. If you added private skills, they should appear in the list.

---

## Troubleshooting

### Agent does not see private skills

Ensure your private skill directory contains a `SKILL.md` file with valid YAML frontmatter (a `name` and `description` field). Check an existing skill like `skills/brainstorming/SKILL.md` for the expected format.

### Plugin fails to install from internal git server

Verify that the git URL is accessible from the machine running the agent. For HTTPS with authentication:

```bash
git clone https://user:token@git.internal.example.com/superpowers.git
```

Store credentials using your system credential manager rather than embedding them in config files.

### Upstream changes conflict with private skills

Use a long-lived private branch that is rebased or merged onto the upstream `main` periodically. Keep private skills isolated to their own directories so upstream changes are unlikely to touch them.
