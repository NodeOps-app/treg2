---
title: CreateOS Sandbox — own-key catalog and lifecycle tools
status: implemented locally; direct live calls passed; vendor review pending
sources:
  - src/treg/oauth_providers.py
  - src/treg/catalog/createos.yaml
  - src/treg/catalog/capabilities.yaml
  - src/treg/web/logos/createos.svg
  - tests/test_oauth_providers_m3.py
  - tests/test_key_providers.py
  - tests/test_createos.py
related:
  - architecture/catalog.md
  - architecture/auth-secrets.md
---

# CreateOS Sandbox

`CREATEOS` accepts a customer-issued API key in `X-Api-Key` and checks it with
`GET /v1/whoami`. An invalid key returned HTTP 401 with
`{"status":"fail","data":{"auth":"invalid api key"}}` on 2026-09-21. The key
is stored by treg in the customer's team and injected only when proxying calls.

`createos.yaml` catalogs two read-only lookups (shapes and root filesystems),
three account operations (list, get, and patch), and three actions: create a
sandbox, run one buffered command in it, and delete it. The lookups expose
current valid create inputs. PATCH changes ingress or idle auto-pause settings;
shape and rootfs are fixed at creation. CreateOS bills running compute to its
own customer account by vCPU and memory runtime.
These entries have no computable per-call treg price and are deliberately
own-key only. They must not reserve or settle a treg balance charge.

On 2026-09-21, a customer key passed `whoami` and direct live checks against
all eight CreateOS API operations. The first disposable sandbox passed create →
exec → delete; its command returned exit code 0 and `treg-ok`. A second check
passed shapes, rootfs, create → list → get → update → delete. PATCH changed
`auto_pause_after_seconds` to 120. Both deletions returned `destroying` and
were requested immediately after testing. A create attempt without `rootfs`
returned HTTP 400, so the catalog requires that field and successful checks
used `devbox:1`. A create attempt with a 24-character name returned HTTP 400;
the catalog now notes the observed 22-character maximum. treg's catalog proxy
was exercised separately with an in-process test for all eight endpoints. The
endpoints remain marked unverified until an
independent vendor review runs them through treg and checks the CreateOS usage
meter. No live response examples or secrets are stored in the repository.
