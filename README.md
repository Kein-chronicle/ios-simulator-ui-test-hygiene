# ios-simulator-ui-test-hygiene

**Keep iOS simulator UI tests and crash-fuzz runs from dying on a laptop: the 16 GB memory wall and the crash-report dialog that never goes away.**

[![test](https://github.com/Kein-chronicle/ios-simulator-ui-test-hygiene/actions/workflows/test.yml/badge.svg)](https://github.com/Kein-chronicle/ios-simulator-ui-test-hygiene/actions/workflows/test.yml) ![license](https://img.shields.io/badge/license-MIT-blue.svg)

## Why

"Stopped because the system is running low on memory" on a 16 GB Mac is a resource problem, not a test problem. And a fuzz gate that deliberately kills your app triggers a crash dialog that blocks the run.

> **Status: doc-only.** Specific to macOS and the iOS simulator toolchain as observed; versions change.

## Install

**As a Claude Code skill** (Claude uses it when the task matches the skill description):

```sh
git clone https://github.com/Kein-chronicle/ios-simulator-ui-test-hygiene.git ~/.claude/skills/ios-test-hygiene
```

For a single project, clone into `<project>/.claude/skills/ios-test-hygiene` instead, then start a new Claude Code session.

**Just read it:** the whole skill is [SKILL.md](SKILL.md). It is plain Markdown; copy any part into your own `CLAUDE.md` or prompts.

**Requires:** nothing to read it; Python 3.10+ only to run `validate.py`.

## What it covers

- Memory wall on 16 GB Macs: what to measure, what to close, how to chunk the run.
- The crash dialog that will not dismiss: which process owns it and what to disable without losing the crash logs.

## Limits

- Specific to macOS and the iOS simulator toolchain as observed; versions change.

## Related projects

These solve neighbouring problems; use them when they fit better.

- **Xcode test plans and `-parallel-testing-enabled`**: the official knobs for splitting and parallelizing runs

## More from the same collection

Small, tested tools that grew out of running AI agents and game/app pipelines for months:

- [**safe-delete-sole-copy-check**](https://github.com/Kein-chronicle/safe-delete-sole-copy-check): Before you delete a project folder: is any of it the only copy? Checks the live remote, unpushed commits and tags, untracked and ignored files, and verifies copies by hash.
- [**exif-orientation-guard**](https://github.com/Kein-chronicle/exif-orientation-guard): Find and fix images whose stored size differs from their displayed size because of an EXIF orientation tag.
- [**macos-tcc-automation-permissions**](https://github.com/Kein-chronicle/macos-tcc-automation-permissions): Make unattended macOS automation survive permission resets: AppleScript -1743/-1728, launchd jobs, screen recording, and real vs synthetic clicks.

## Development

```sh
python3 validate.py
```

This only validates the skill file (frontmatter and referenced paths). See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT. See [LICENSE](LICENSE).

If this saved you a debugging session, a star helps other people find it. Bug reports and failing test cases are even better.

<sub>Keywords: ios, xcode, xcodebuild, ui-testing, simulator, fuzzing, macos, claude-skills</sub>
