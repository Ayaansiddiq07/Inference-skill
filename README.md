# Inference Engineering

A portable Agent Skill for making measured architecture, performance, and production decisions
for generative AI inference systems. The skill uses the open `SKILL.md` format and can be used
with Codex, Claude Code, Google Antigravity, and other agents that support Agent Skills.

Use it when selecting models or hardware, diagnosing latency or throughput, optimizing LLM or
multimodal serving, benchmarking, scaling GPU services, or estimating inference cost. It provides
workload questions, decision workflows, optimization playbooks, and production-readiness checks.

## Use the skill

After installing it in an agent's global skills directory, you can use it in any codebase. Ask the
agent a relevant question, or name the skill explicitly, for example:

> Use the inference-engineering skill to plan a benchmark for our streaming model service.

You can also ask the agent to review an inference design, find a measured bottleneck, compare
serving options, or create a production-readiness checklist. The skill guides the analysis; it
does not replace workload measurements or current vendor documentation. Agents can select it
automatically for matching tasks; naming `inference-engineering` makes the request explicit.

## Agent support and install locations

The skill has no runtime dependencies. Install Git, then choose the directory for your agent and
scope. The path in each cell is the parent skills directory; add `inference-engineering` beneath
it. Documentation: [Codex](https://developers.openai.com/api/reference/cli),
[Claude Code](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview), and
[Google Antigravity](https://antigravity.google/docs/skills?app=antigravity-ide).

| Agent | Global (all projects) | Project-specific |
| --- | --- | --- |
| Codex | `~/.agents/skills/` | `.agents/skills/` |
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| Google Antigravity | `~/.gemini/config/skills/` | `.agents/skills/` |

Antigravity also supports the legacy project path `.agent/skills/`. Codex installations that
discover `~/.codex/skills/` can use it as their global parent directory.

All three agents use the same `SKILL.md` and `references/` files; only the parent install path
changes. The `agents/openai.yaml` file provides OpenAI-specific display metadata and is not needed
by Claude Code or Antigravity. Other agents may use this skill if they support the Agent Skills
format and load the folder's `SKILL.md` plus its relative references.

The destination folder must not already exist. If you already cloned this repository there, use
the update command below instead of cloning again. Back up any existing non-Git folder before
replacing it.

### Global install on Windows

Set `$skillRoot` to the global path from the table, then run:

```powershell
$skillRoot = Join-Path $env:USERPROFILE ".claude\skills" # Change for Codex or Antigravity.
$installPath = Join-Path $skillRoot "inference-engineering"
New-Item -ItemType Directory -Force $skillRoot | Out-Null
git clone https://github.com/Ayaansiddiq07/Inference-skill.git $installPath
```

For Codex, set the path to `.agents\skills`. For Antigravity, use `.gemini\config\skills`.

### Global install on macOS or Linux

```bash
skill_root="$HOME/.claude/skills" # Change for Codex or Antigravity.
mkdir -p "$skill_root"
git clone https://github.com/Ayaansiddiq07/Inference-skill.git \
  "$skill_root/inference-engineering"
```

For Codex, set `skill_root` to `$HOME/.agents/skills`. For Antigravity, use
`$HOME/.gemini/config/skills`. Start a new agent session after installation if the skill does not
appear. Confirm `SKILL.md` exists in the cloned directory, then ask the agent to use
`inference-engineering` explicitly.

## Update or pull the latest version

If you installed with Git, update the skill from the same directory you cloned it into. Set
`$skillPath` (PowerShell) or `skill_path` (macOS/Linux) to the corresponding global or
project-specific path from the table, then pull the latest `main` branch:

### Windows PowerShell

```powershell
$skillRoot = Join-Path $env:USERPROFILE ".claude\skills" # Change for your agent and scope.
$skillPath = Join-Path $skillRoot "inference-engineering"
git -C $skillPath pull --ff-only origin main
```

### macOS or Linux

```bash
skill_path="$HOME/.claude/skills/inference-engineering" # Change for your agent and scope.
git -C "$skill_path" pull --ff-only origin main
```

`--ff-only` avoids creating a merge commit. If you changed files locally and Git refuses the pull,
back up your changes, review the diff, and resolve or discard them before updating. Start a new
agent session if it does not reload the updated skill.

## Install in one project only

To limit the skill to one project, use the project-specific path from the table. Create its parent
directory, then clone the skill there. For Codex or Antigravity, use `.agents/skills`; for Claude
Code, use `.claude/skills`.

```bash
mkdir -p .claude/skills # Change to .agents/skills for Codex or Antigravity.
git clone https://github.com/Ayaansiddiq07/Inference-skill.git \
  .claude/skills/inference-engineering
```

In Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force ".claude\skills" | Out-Null # Change per the table.
git clone https://github.com/Ayaansiddiq07/Inference-skill.git `
  ".claude\skills\inference-engineering"
```

The project-local copy is useful when a team wants a reviewed, version-controlled skill alongside
its codebase. Follow your team's policy before committing assistant instructions to a project.

## Uninstall

Remove only the `inference-engineering` directory from the location where you installed it. Check
the path first and back up any local changes you want to keep.

### Windows PowerShell

```powershell
$skillRoot = Join-Path $env:USERPROFILE ".claude\skills" # Change for your agent and scope.
$skillPath = Join-Path $skillRoot "inference-engineering"
Remove-Item -LiteralPath $skillPath -Recurse
```

### macOS or Linux

```bash
rm -r "$HOME/.claude/skills/inference-engineering" # Change for your agent and scope.
```

For a project-specific install, remove the matching folder under `.agents/skills` or
`.claude/skills` in that project. This does not remove other skills or the agent itself.

## Repository contents

- `SKILL.md` — core workflow, guidance, and reference routing.
- `references/` — detailed guidance for hardware, optimization, modalities, software stack, and
  production operations.
- `agents/openai.yaml` — OpenAI-specific display metadata; Claude Code and Antigravity use the
  shared skill instructions without this file.

## License and attribution

Original material written for this skill is licensed under the
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). To reuse or adapt it, credit
Ayaan-Siddique (@Ayaansiddiq07) and acknowledge Philip Kiely's *Inference Engineering* (Baseten
Books, 2026) as the source inspiration. State whether you changed the material. The source
acknowledgment does not imply
endorsement or affiliation. Philip Kiely and Baseten Books did not grant this license; their book
and other third-party materials are not covered by it. See [LICENSE](LICENSE) for details.

Suggested attribution:

> Inference Engineering skill by
> [Ayaan-Siddique (@Ayaansiddiq07)](https://github.com/Ayaansiddiq07/Inference-skill), informed by
> [Philip Kiely's *Inference Engineering*](https://www.baseten.co/inference-engineering/book)
> (Baseten Books, 2026). Modified: [describe changes]. Licensed under
> [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

## Contributing

Keep guidance workload-driven and product-neutral. Verify changing model, hardware, framework, and
vendor details against current official documentation. Update the relevant reference file when
changing a topic, then check every internal link in `SKILL.md` still points to an existing file.
