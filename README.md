<p align="center"><img src="docs/banner.svg" alt="missions: Preflight mission reports: Go / No-Go, phase by phase" width="100%"></p>

# missions

Mission reports from **TAEM** preflight runs. Each file is the record of one mission: the pad check, the controllers' findings, and the Go / No-Go gate decision that came out of it.

| Mission | Review | Date | Verdict |
|---|---|---|---|
| [fabric-forge-blog](fabric-forge-blog.md) | TAEM Shakedown Review: fabric-forge/blog | 2026-03-19 | GO — unanimous, all 13 controllers cleared |
| [fabric-forge-social-ui](fabric-forge-social-ui.md) | TAEM Shakedown Review: fabric-forge/social-ui | 2026-03-19 | GO — 12/13 controllers cleared, 1 Ollama timeout (not a code finding) |
| [fabric-forge-social](fabric-forge-social.md) | TAEM Shakedown Review: fabric-forge/social | 2026-03-19 | GO — 12/13 controllers cleared, 1 Ollama timeout (not a code finding) |
| [git-steer-post-remediation](git-steer-post-remediation.md) | TAEM Post-Remediation Review: ry-ops/git-steer | 2026-03-18 | GO — all Cycle 1 blocking findings resolved. Mission LANDED. |
| [git-steer-remediation-cycle-1](git-steer-remediation-cycle-1.md) | TAEM Remediation — Cycle 1 | — | NO-GO (SECINSP, TRC, PRB) |
| [git-steer](git-steer.md) | TAEM Full Shakedown Review: ry-ops/git-steer | 2026-03-17 | HOLD — strong foundation, actionable findings below |
| [open-claw](open-claw.md) | TAEM Full Shakedown Review: openclaw/openclaw | 2026-03-17 | GO — impressive security posture, advisories below |

See [taem](https://github.com/TAEM-DEV/taem) for how missions run, and [mc-state](https://github.com/TAEM-DEV/mc-state) for the state they leave behind.

<!-- org-footer -->
---

<p align="center"><sub>Part of <a href="https://github.com/TAEM-DEV">TAEM</a> · mission control preflight for software integration · built by <a href="https://github.com/ry-ops">ry-ops</a></sub></p>
