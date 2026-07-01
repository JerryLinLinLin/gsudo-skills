# gsudo-skills

An AI Agent **skill** — [`gsudo`](skills/gsudo/SKILL.md) — for driving
[**gsudo**](https://gerardog.github.io/gsudo/), a `sudo` for Windows, from a shell. It's a cheatsheet +
bundled reference docs for running commands elevated (as admin) or elevating the current shell **in the
same console window**: single-command elevation, elevating CMD / PowerShell / WSL / Git-Bash, cutting
UAC popups with the credentials cache, running as System / TrustedInstaller / another user / a chosen
integrity level, elevating in a new window, the PowerShell `gsudo { ScriptBlock }` and `Invoke-gsudo` /
`$using:` syntax, self-elevating scripts, detecting elevation with `gsudo status`, and tuning behavior
via `gsudo config`.

```
gsudo-skills/
├── README.md                     ← you are here (setup / install)
├── LICENSE
└── skills/
    └── gsudo/
        ├── SKILL.md                  ← the cheatsheet entry point (invocation → cache → identities → PowerShell → config)
        └── reference/                ← verbatim upstream gsudo docs (authoritative fallback)
            ├── usage.md              ← full CLI options + configuration syntax
            ├── powershell.md         ← { ScriptBlock } / Invoke-gsudo / $using: elevation from PowerShell
            ├── credentials-cache.md  ← cache modes (Auto / Explicit / Disabled), pid & duration scoping
            ├── security.md           ← the "convenience feature, not a security boundary" reasoning
            ├── how-it-works.md       ← elevation modes: TokenSwitch / Attached / VT / Piped
            └── scripts-self-elevation.md  ← CMD/PowerShell templates to re-launch a script elevated
```

The skill **assumes gsudo is installed and on `PATH`** — which is exactly what the winget install
below gives you (`gsudo` and its `sudo` alias land on `PATH`). Install once with the guide below, then
use the skill.

---

## Get gsudo

gsudo is a single portable console app — no Windows service, no system changes beyond adding it to
`PATH`. It works on Windows 7 SP1 through Windows 11.

### Recommended: winget

```powershell
winget install gerardog.gsudo
```

The Chocolatey (`choco install gsudo`) and Scoop (`scoop install gsudo`) packages, and the `MSI` from
the [latest release](https://github.com/gerardog/gsudo/releases/latest), work too. **Open a new shell**
afterwards so the updated `PATH` is picked up.

### Verify the install

```powershell
# After install + a NEW shell:
gsudo --version         # -> gsudo v2.x ...
gsudo status            # -> current user, elevation state, integrity level, cache status
```

The installers create a **`sudo` alias** for gsudo, so `sudo notepad` works out of the box. (If
Microsoft's *Sudo for Windows* is also installed, `sudo` resolves to it first — see
[`skills/gsudo/SKILL.md`](skills/gsudo/SKILL.md) §13 to flip precedence.)

For the best experience in **PowerShell** and **Git-Bash/MSYS2**, add the small profile snippets from
SKILL.md §5 / §10 (module import for `gsudo !!` + tab-complete; the `.bashrc` wrapper for the cache).

---

## Install this skill into your AI agent (global / user space)

Claude Code, Codex CLI, and GitHub Copilot all read the **same `SKILL.md` Agent Skill format** from a
personal skills dir — installing is just dropping the `gsudo/` folder into each:

| Agent | Global skills dir (Windows) |
|---|---|
| Claude Code | `%USERPROFILE%\.claude\skills\` |
| OpenAI Codex CLI | `%USERPROFILE%\.codex\skills\` |
| GitHub Copilot | `%USERPROFILE%\.copilot\skills\` |

```powershell
$src = "C:\projects\gsudo-skills\skills\gsudo"    # adjust to where you cloned it
foreach ($a in '.claude', '.codex', '.copilot') {         # keep only the agents you use
  $dst = "$env:USERPROFILE\$a\skills"
  New-Item -ItemType Directory -Force $dst | Out-Null
  Copy-Item -Recurse -Force $src $dst
}
```

Each agent auto-discovers the skill by its `name`/`description` and loads it when a task matches (in
Claude Code, run `/skills` to confirm).

---

## Use responsibly

gsudo runs code with elevated privileges. Same-desktop elevation (gsudo, and Windows UAC itself) is a
**convenience feature, not a security boundary** — a compromised medium-integrity process on the same
desktop can hijack an elevation, and an active credentials cache widens that window. Only elevate code
you trust, keep the cache off unless you need it (`gsudo -k` ends all sessions), and prefer elevating
the single command that needs admin rather than a whole shell. See
[security.md](skills/gsudo/reference/security.md).

## Licensing

- This skill (the authored `skills/gsudo/SKILL.md` content) is released under the
  [MIT License](LICENSE).
- `skills/gsudo/reference/` contains documentation from the
  [gsudo project](https://github.com/gerardog/gsudo), redistributed under gsudo's original **MIT
  License** (© 2019 Gerardo Grignoli and contributors). See
  <https://gerardog.github.io/gsudo/> for the canonical source.
- gsudo itself is developed by Gerardo Grignoli and contributors under the MIT License and is **not**
  included here — install it via winget as shown above.
