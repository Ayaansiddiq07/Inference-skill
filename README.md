# Inference Engineering

An installable Codex skill for making measured architecture, performance, and production
decisions for generative AI inference systems.

Use it when selecting models or hardware, diagnosing latency or throughput, optimizing LLM or
multimodal serving, benchmarking, scaling GPU services, or estimating inference cost. It provides
workload questions, decision workflows, optimization playbooks, and production-readiness checks.

## Use the skill

The skill works across codebases when installed in your global Codex skills directory. In Codex,
ask a question that matches its scope, such as:

> Use the inference-engineering skill to plan a benchmark for our streaming model service.

You can also ask Codex to review an inference design, find a measured bottleneck, compare serving
options, or create a production-readiness checklist. The skill guides the analysis; it does not
replace workload measurements or current vendor documentation.

## Install globally

The skill has no runtime dependencies. Install Git first, then clone this repository into
`~/.agents/skills/inference-engineering`. This is the documented user-wide
[Codex skills location](https://developers.openai.com/api/reference/cli). If your Codex setup is
configured to discover `~/.codex/skills`, you can use that location instead. Use a destination
folder that does not already exist; back up any existing skill folder before replacing it.

### Windows PowerShell

```powershell
$skillsPath = Join-Path $env:USERPROFILE ".agents\skills"
$skillPath = Join-Path $skillsPath "inference-engineering"
New-Item -ItemType Directory -Force $skillsPath | Out-Null
git clone https://github.com/Ayaansiddiq07/Inference-skill.git $skillPath
```

### macOS or Linux

```bash
mkdir -p "$HOME/.agents/skills"
git clone https://github.com/Ayaansiddiq07/Inference-skill.git \
  "$HOME/.agents/skills/inference-engineering"
```

Restart Codex after installation if the skill does not appear. Confirm the installation by checking
that `SKILL.md` exists in the cloned directory and ask Codex to use the skill explicitly.

## Update or pull the latest version

If you installed with Git, pull updates in the skill directory:

### Windows PowerShell

```powershell
$skillPath = Join-Path $env:USERPROFILE ".agents\skills\inference-engineering"
git -C $skillPath pull --ff-only origin main
```

### macOS or Linux

```bash
git -C "$HOME/.agents/skills/inference-engineering" pull --ff-only origin main
```

`--ff-only` avoids creating a merge commit. If you changed files locally and Git refuses the pull,
back up your changes, review the diff, and resolve or discard them before updating. Restart Codex
after updating so it reloads the skill.

## Install in one project only

To limit the skill to a single project, clone it into that project's `.agents/skills` directory
instead of your home directory:

```bash
mkdir -p .agents/skills
git clone https://github.com/Ayaansiddiq07/Inference-skill.git \
  .agents/skills/inference-engineering
```

In Windows PowerShell, create the parent directory first:

```powershell
New-Item -ItemType Directory -Force ".agents\skills" | Out-Null
git clone https://github.com/Ayaansiddiq07/Inference-skill.git `
  ".agents\skills\inference-engineering"
```

The project-local copy is useful when a team wants a reviewed, version-controlled skill alongside
its codebase. Follow your team's policy before committing assistant instructions to a project.

## Uninstall

Remove the cloned skill directory. This does not remove Codex or change other skills.

### Windows PowerShell

```powershell
$skillPath = Join-Path $env:USERPROFILE ".agents\skills\inference-engineering"
Remove-Item -LiteralPath $skillPath -Recurse
```

### macOS or Linux

```bash
rm -r "$HOME/.agents/skills/inference-engineering"
```

Back up any local changes you want to keep before uninstalling. To remove a project-local copy,
delete that project's `.agents/skills/inference-engineering` directory instead.

## Repository contents

- `SKILL.md` — core workflow, guidance, and reference routing.
- `references/` — detailed guidance for hardware, optimization, modalities, software stack, and
  production operations.
- `agents/openai.yaml` — display metadata for compatible skill interfaces.

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
