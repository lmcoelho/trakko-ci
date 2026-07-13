# lmcoelho/trakko-ci — Trakko Pipeline v2

Central library of reusable GitHub Actions workflows + composite actions
shared across every `lmcoelho/*` (trakko) repository.

## Quick reference

Reusable workflows (call with `uses: lmcoelho/trakko-ci/.github/workflows/_<name>.yml@v1`):

| Name | Purpose |
|---|---|
| `_validate-iac.yml` | gitleaks + yamllint + kubeconform on `k8s/` and `.github/workflows/` |
| `_test-python.yml` | setup-python + pytest with optional coverage gate and lint |
| `_test-node.yml` | setup-node + npm install + test (and optional build/typecheck) |
| `_test-integration.yml` | adds Postgres + Redis services for integration tests |
| `_security-python.yml` | gitleaks + pip-audit + bandit + optional cyclonedx SBOM |
| `_security-node.yml` | gitleaks + npm audit + optional hardcoded-secret grep |
| `_security-image.yml` | Trivy (OS + libs) + Syft SBOM + Grype, all optional/blocking-configurable |
| `_build-image.yml` | buildx + GHCR + metadata-action + GHA layer cache, returns sha-tag |
| `_deploy-vps.yml` | Tailscale + SSH + apply manifests + set image + verify + rollback |

Composite actions (call with `uses: lmcoelho/trakko-ci/actions/<name>@v1`):

| Name | Purpose |
|---|---|
| `tailscale-ssh-bootstrap` | Connect to tailnet, install deploy SSH key, ssh-keyscan with retry |
| `gitleaks-scan` | gitleaks + Node-24 forcing + permission documentation |
| `step-summary` | Uniform pipeline summary block in `$GITHUB_STEP_SUMMARY` |

## Versioning

Callers reference `@v1`. See [`docs/version-policy.md`](docs/version-policy.md).

## Caller examples

See [`docs/reusable-workflows.md`](docs/reusable-workflows.md) for the input/output
contract of each workflow and a worked caller example.

## Audit context

This library was created as part of `/trakko-pipeline-audit` (see
`~/audit/trakko-pipeline-v2/04-design.md`). It collapses ~750–900 lines of
duplicated YAML across six trakko repos into a versioned, evidence-tested
common library.

## License

Internal use. Not currently licensed for redistribution outside lmcoelho.

<!-- Verified: Linear GitHub integration end-to-end test (TRA-83), 2026-07-13 -->
