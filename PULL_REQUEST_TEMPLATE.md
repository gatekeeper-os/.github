## What

<!-- one or two sentences -->

## Why

<!-- the problem, or a link to the issue -->

## Security checklist

- [ ] No new path resolves a grant or registers a `gk_*` tool outside `Kernel.resolveGrant` / `registerGatekeeperTools` (or this PR doesn't touch the kernel)
- [ ] Every new outside-world interaction goes through `authorizeObservation()` or `submitAction()` (or this PR doesn't touch a gatekeeper)
- [ ] Nothing new is logged that could contain a secret, prompt, header, token, or body
- [ ] No new import from `openclaw/*` beyond documented `plugin-sdk` subpaths
- [ ] `pnpm build && pnpm test` pass locally

## AI assistance

<!-- If an agent wrote or reviewed part of this, say which part. A human has read the full diff: yes / no -->
