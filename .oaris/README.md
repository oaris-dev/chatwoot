# .oaris — Oaris Edition deployment config

This directory holds the Oaris Edition deployment configuration for the **public**
`oaris-dev/chatwoot` fork.

- `coolify-deployment.yaml` — production compose Coolify deploys from. Env values are
  placeholders (`${...}`); safe to be public.

## Internal docs live offline

Internal planning docs and runbooks (project roadmap, deployment guides, backup plans)
contain infra-specific detail — hostnames, S3 bucket paths, server topology — and are
**intentionally not committed** to this public fork.

They are kept offline under `.oaris/local/` (gitignored). Keep it that way: do not commit
hostnames, bucket paths, or other infra specifics into this repository.
