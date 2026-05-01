# Secret rotation runbook

Every secret consumed by the reusable workflows is rotated on a fixed
quarterly cadence. Schedule a routine via `/schedule` to fire on the
first day of each quarter; the routine opens a rotation PR and a
checklist issue.

## Quarterly schedule

| Quarter | Secret | Repos affected |
|---|---|---|
| Q1 (Jan) | `DEPLOY_SSH_KEY` | game-fabric, vigilos, atlas, thundera, money-maker |
| Q2 (Apr) | `TS_OAUTH_SECRET` (shared client) + atlas's own | all 6 |
| Q3 (Jul) | `GHCR_PAT` | thundera, money-maker (cluster-side image pull) |
| Q4 (Oct) | `CI_EVENT_HMAC` (when atlas /ci/event lands; not in v1) | all 6 |

## Procedure: `DEPLOY_SSH_KEY`

```bash
# 1. Generate a new ed25519 keypair on the operator machine
ssh-keygen -t ed25519 -C "ci-deploy-$(date +%Y%m)" -f /tmp/new_deploy -N ''

# 2. Append the new public key to the VPS authorized_keys
ssh -i ~/.ssh/claude-key claude@100.100.223.46 \
  "echo '$(cat /tmp/new_deploy.pub)' >> ~/.ssh/authorized_keys"

# 3. Push the new private key as the new value of DEPLOY_SSH_KEY in every consumer repo
for repo in game-fabric vigilos atlas thundera money-maker; do
  gh secret set DEPLOY_SSH_KEY -R lmcoelho/$repo < /tmp/new_deploy
done

# 4. Smoke-test: trigger one deploy per repo via workflow_dispatch
for repo in game-fabric vigilos atlas thundera money-maker; do
  gh workflow run "Build & Deploy" -R lmcoelho/$repo
done

# 5. After every consumer has run a successful deploy with the new key,
#    remove the old public key from VPS authorized_keys
ssh -i ~/.ssh/claude-key claude@100.100.223.46 \
  "sed -i '/old-comment/d' ~/.ssh/authorized_keys"

# 6. Securely destroy the new private key on the operator machine
shred -vfz -n 3 /tmp/new_deploy /tmp/new_deploy.pub
```

## Procedure: `TS_OAUTH_SECRET`

Tailscale OAuth secrets cannot be re-read after creation; create a new
client first, then atomically swap.

```bash
# 1. In the Tailscale admin console, create a new OAuth client with:
#    - Devices Core: Write
#    - Tag: tag:ci
#    Note the new client_id and secret.

# 2. Push to every consumer repo (use the canonical names per CLAUDE.md)
for repo in ai-hosting vigilos game-fabric thundera money-maker; do
  gh secret set TS_OAUTH_CLIENT_ID -R lmcoelho/$repo -b "<new_client_id>"
  gh secret set TS_OAUTH_SECRET    -R lmcoelho/$repo -b "<new_secret>"
done

# atlas keeps its own dedicated client (per ai-hosting/CLAUDE.md). Rotate it
# the same way but using its own client/secret.
gh secret set TS_OAUTH_CLIENT_ID -R lmcoelho/atlas -b "<new_atlas_client_id>"
gh secret set TS_OAUTH_SECRET    -R lmcoelho/atlas -b "<new_atlas_secret>"

# 3. Trigger a workflow_dispatch on every repo to verify Tailscale auth.

# 4. Delete the old OAuth clients in the Tailscale admin console.
```

## Procedure: `GHCR_PAT`

Used as a long-lived classic PAT for cluster-side `docker-registry`
secret rotation in K3s (per existing project lesson — fine-grained
tokens didn't reliably work for GHCR pulls in K3s at the time of
writing).

```bash
# 1. Create a new classic PAT at github.com → settings → developer settings
#    → personal access tokens → tokens (classic). Scopes: read:packages,
#    write:packages. Expiry: 1 year.

# 2. Push to thundera + money-maker (both expect `GHCR_PAT`; rename
#    thundera's existing GHCR_PULL_TOKEN to GHCR_PAT)
gh secret set GHCR_PAT -R lmcoelho/thundera -b "<new_pat>"
gh secret delete GHCR_PULL_TOKEN -R lmcoelho/thundera || true
gh secret set GHCR_PAT -R lmcoelho/money-maker -b "<new_pat>"

# 3. Trigger a workflow_dispatch deploy on each. The deploy job will
#    create a new docker-registry secret in K3s with the new PAT.

# 4. After successful pulls, revoke the old PAT.
```
