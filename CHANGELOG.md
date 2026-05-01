# Changelog

All notable changes to this library are documented here. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [v1.0.0] — 2026-05-01

### Added
- 9 reusable workflows: `_validate-iac`, `_test-python`, `_test-node`,
  `_test-integration`, `_security-python`, `_security-node`,
  `_security-image`, `_build-image`, `_deploy-vps`.
- 3 composite actions: `tailscale-ssh-bootstrap`, `gitleaks-scan`,
  `step-summary`.
- `smoke-test.yml` self-test workflow.
- Documentation: `docs/reusable-workflows.md`, `docs/version-policy.md`,
  `docs/secret-rotation-runbook.md`.

### Notes
- Initial release. Adopted by ai-hosting / vigilos / game-fabric / atlas /
  thundera / money-maker as part of the Trakko Pipeline v2 migration.
