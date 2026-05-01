# Reusable workflows — input/output contracts

Each section gives the callable signature and a minimal caller example.
Authoritative source: the YAML files under `.github/workflows/_*.yml`.

---

## `_validate-iac.yml`

**Inputs:** `yaml-paths` (default `k8s/`), `yamllint-config`,
`kubeconform-args`, `include-workflows` (default `true`),
`gitleaks-permissions` (`read` | `write`).

**Caller example (ai-hosting):**

```yaml
jobs:
  validate:
    uses: lmcoelho/trakko-ci/.github/workflows/_validate-iac.yml@v1
    with:
      yaml-paths: "k8s/ open-brain/k8s/ server/k8s-manifests/per-namespace/"
```

---

## `_test-python.yml`

**Inputs:** `python-version` (required), `working-directory`,
`requirements-file`, `test-command`, `coverage-fail-under`,
`coverage-paths`, `lint-command`, `timeout-minutes`.

**Caller example (money-maker):**

```yaml
jobs:
  test:
    uses: lmcoelho/trakko-ci/.github/workflows/_test-python.yml@v1
    with:
      python-version: "3.13"
      coverage-paths: "src config"
      coverage-fail-under: 60
      lint-command: "ruff check src/ config/ tests/"
```

---

## `_test-node.yml`

**Inputs:** `node-version` (required), `working-directory`,
`install-command`, `build-command`, `test-command`, `typecheck-command`,
`timeout-minutes`.

**Caller example (vigilos):**

```yaml
jobs:
  test:
    uses: lmcoelho/trakko-ci/.github/workflows/_test-node.yml@v1
    with:
      node-version: "20"
      build-command: "npm run build --workspace=packages/pipeline-engine"
      typecheck-command: "npx tsc --noEmit"
      test-command: "npm test -- --coverage"
```

---

## `_test-integration.yml`

**Inputs:** `runtime` (`python`|`node`), `runtime-version`,
`working-directory`, `install-command`, `seed-command`, `test-command`,
`database-image`, `redis-image`, `database-name`, `database-user`,
`timeout-minutes`.

Sets `DATABASE_URL` and `REDIS_URL` env vars by default.

---

## `_security-python.yml`

**Inputs:** `python-version` (required), `working-directory`,
`requirements-file`, `run-gitleaks`, `run-pip-audit`,
`pip-audit-strict`, `run-bandit`, `bandit-paths`, `bandit-severity`,
`run-cyclonedx`, `sbom-output-path`.

**Caller example (game-fabric):**

```yaml
jobs:
  security:
    uses: lmcoelho/trakko-ci/.github/workflows/_security-python.yml@v1
    with:
      python-version: "3.12"
      bandit-paths: "core brain services config"
      run-cyclonedx: true
```

---

## `_security-node.yml`

**Inputs:** `node-version` (required), `working-directory`,
`run-gitleaks`, `run-npm-audit`, `npm-audit-level`, `accepted-vulns`
(comma-sep package names), `run-hardcoded-secret-scan`.

---

## `_security-image.yml`

**Inputs:** `image` (required, full ref), `run-trivy-os`,
`trivy-os-blocking`, `run-trivy-libs`, `trivy-libs-blocking`,
`run-syft-sbom`, `run-grype`, `grype-severity-cutoff`,
`grype-only-fixed`, `sbom-retention-days`.

**Caller example (money-maker):**

```yaml
jobs:
  scan:
    needs: build
    uses: lmcoelho/trakko-ci/.github/workflows/_security-image.yml@v1
    with:
      image: ghcr.io/lmcoelho/money-maker:${{ needs.build.outputs.image-tag }}
      grype-severity-cutoff: "high"
```

---

## `_build-image.yml`

**Inputs:** `image-name` (required, short name), `registry-owner`,
`context`, `dockerfile`, `build-args`, `platforms`, `push-latest`.

**Outputs:** `image-tag` (e.g. `sha-abc1234`), `image-ref` (full ref
without tag).

**Caller example (vigilos):**

```yaml
jobs:
  build-api:
    uses: lmcoelho/trakko-ci/.github/workflows/_build-image.yml@v1
    with:
      image-name: vigilos-api
      dockerfile: docker/Dockerfile.api
```

To consume the outputs in a sibling job:
`needs.build-api.outputs.image-tag`,
`needs.build-api.outputs.image-ref`.

---

## `_deploy-vps.yml`

**Inputs:** `environment` (required), `namespace` (required), `image`
(required, full ref), `deployments` (CSV `deploy/name:container`),
`cronjobs` (CSV), `apply-manifests-dir`, `verify-command`,
`rollback-on-failure`, `rollout-timeout`, `timeout-minutes`.

**Required secrets:** `TS_OAUTH_CLIENT_ID`, `TS_OAUTH_SECRET`,
`DEPLOY_HOST`, `DEPLOY_SSH_KEY`. Optional: `DEPLOY_USER` (default
`claude`).

**Caller example (atlas):**

```yaml
jobs:
  deploy:
    needs: [build-router, build-infomaniak]
    uses: lmcoelho/trakko-ci/.github/workflows/_deploy-vps.yml@v1
    with:
      environment: production
      namespace: atlas
      image: ghcr.io/lmcoelho/atlas-router:${{ needs.build-router.outputs.image-tag }}
      deployments: "atlas-router:router"
      apply-manifests-dir: k8s/atlas
      verify-command: |
        kubectl -n atlas exec deployment/atlas-router -- \
          python3 -c "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8080/healthz', timeout=5).read()"
    secrets: inherit
```

The `image` input is validated to match `ghcr.io/<owner>/<name>:sha-<7+hex>`
before any SSH command is constructed (defends against shell injection
in the heredoc segments).

`apply-manifests-dir` deliberately excludes `deployment*.yaml`,
`*deployment.yaml`, `*cronjob*.yaml`, and `*template*.yaml` from the
non-image-bearing apply step. Image refs flow through `set image` /
`patch` instead.

---

## Composite actions

### `actions/tailscale-ssh-bootstrap`

```yaml
- uses: lmcoelho/trakko-ci/actions/tailscale-ssh-bootstrap@v1
  with:
    oauth-client-id: ${{ secrets.TS_OAUTH_CLIENT_ID }}
    oauth-secret: ${{ secrets.TS_OAUTH_SECRET }}
    ssh-key: ${{ secrets.DEPLOY_SSH_KEY }}
    deploy-host: ${{ secrets.DEPLOY_HOST }}
```

### `actions/gitleaks-scan`

```yaml
- uses: lmcoelho/trakko-ci/actions/gitleaks-scan@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    enable-comments: "true"
```

### `actions/step-summary`

```yaml
- uses: lmcoelho/trakko-ci/actions/step-summary@v1
  with:
    job-name: my-job
    details: "- Anything: relevant"
```
