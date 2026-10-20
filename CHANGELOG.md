# Changelog
Format: [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) | [SemVer](https://semver.org/)

A moved or renamed file listed in `anclas.json` is a major version: other repositories
consume those paths. A change to the content of one of those files is a minor version and
updates its hash in `anclas.json` in the same pull request.

## [Unreleased]
### Added
- `CHANGELOG.md`, `CITATION.cff`
- `anclas.json`: the 13 paths the agentic-sdlc-os ecosystem consumes, with their SHA-256
- `docs-check` workflow: anchors and territory on every pull request

## [1.0.0] - 2026-08-25
### Added
- Initial public release: 17 sections, four stack guides, OpenAPI templates and two
  worked services. Provenance in `docs/legal/VERSION_HISTORY.md`.
