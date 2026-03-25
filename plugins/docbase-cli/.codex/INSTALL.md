# Installing DocBase CLI Skill for Codex

## Prerequisites

- Git
- [DocBase CLI](https://www.npmjs.com/package/@krayinc/docbase-cli) installed and authenticated

## Installation

1. **Clone the repository:**

   ```bash
   git clone https://github.com/krayinc/docbase-marketplace.git ~/.codex/docbase-marketplace
   ```

2. **Create the skills symlink:**

   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/docbase-marketplace/plugins/docbase-cli/skills/docbase-cli ~/.agents/skills/docbase-cli
   ```

   **Windows (PowerShell):**

   ```powershell
   New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
   cmd /c mklink /J "$env:USERPROFILE\.agents\skills\docbase-cli" "$env:USERPROFILE\.codex\docbase-marketplace\plugins\docbase-cli\skills\docbase-cli"
   ```

3. **Restart Codex** to discover the skill.

## Updating

```bash
cd ~/.codex/docbase-marketplace && git pull
```

## Uninstalling

```bash
rm ~/.agents/skills/docbase-cli
rm -rf ~/.codex/docbase-marketplace
```
