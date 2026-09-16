# GatekeeperOS

GatekeeperOS is an independent project. It is not affiliated with or endorsed by the OpenClaw Foundation. OpenClaw is a trademark of its owner.

**capability-based access and deferred approvals for OpenClaw**

Stock OpenClaw, no fork, installed as plugins.

Each agent starts with access to nothing. You introduce a resource — a folder, a GitHub repo, an issue — by pasting its URL. From then on the agent reaches that resource only through a **gatekeeper**: a driver that logs every read, queues every write for your approval, and *simulates* the write locally so the agent keeps working while you approve later, in bulk, when it suits you.

No more choosing between "approve every single tool call" and `--dangerously-skip-permissions`.

**Status: published beta.5; npm-only messaging-cell acceptance passed.**
All five packages are available at `0.1.0-beta.5`. Run
`20260916-220009-phase-3` passed on guest Node **22.22.3** and unmodified
OpenClaw **2026.9.2**, including an independently owned gatekeeper absent from
the kernel manifest. Real filesystem writes remain disabled; successful
approval effects were synthetic, not real connected-provider acceptance.

## How it works

- **Grants, not ACLs.** A grant is an opaque handle (`grant:7k3m9q2p`) the agent passes to gatekeeper tools. It never encodes the resource. Only an operator can create one; a URL pasted by anyone else is inert.
- **Reads are authorized and logged.** Every read goes through `authorizeObservation()` before data returns to the agent, and lands in an append-only audit log that never contains prompts, tokens, headers or bodies.
- **Writes are queued and simulated.** Every write goes through `submitAction()`, is journaled as pending, and is merged into the agent's subsequent reads as if it had happened. You apply, reject or revert it later — `gkos approvals apply 3`, or `/approvals apply 3` in a private operator chat.
- **Two gates, not one.** A host-level trusted tool policy denies any gatekeeper call without an active grant *before* any plugin hook runs; the kernel's `before_tool_call` then checks the grant belongs to this agent, session and audience. Kit-owned wrappers register the gatekeeper's exact manifest tools under its own identity; execution and per-grant narrowing remain kernel-owned.
- **Upstream stays untouched.** Nothing in the `openclaw` package is patched or vendored. The upstream version is pinned in a lockfile and a conformance suite decides whether a release is compatible. (The one-command update/rollback pipeline is designed but not built yet — see status.)

## Repositories

| Repo | What it is | Review bar |
|---|---|---|
| [`gatekeeper-os`](https://github.com/gatekeeper-os/gatekeeper-os) | Kernel plugin, shared contracts, gatekeeper kit, `gkos` CLI, installer, conformance suite, and the `fs` and `github` reference gatekeepers | Kernel: every line reviewed |
| [`gatekeepers`](https://github.com/gatekeeper-os/gatekeepers) | Community gatekeepers — one folder per service — plus the `write-gatekeeper` skill any agent can follow to build one | Normal |

## What's proven, what isn't

| | |
|---|---|
| Kernel, `fs` driver, cells, installer, config reconcile, backups | **Source-install evidence** — Phase 3: 78/78 conformance, 98/98 kernel-live checks, 23 model turns on `openclaw@2026.9.2`; 410/410 host tests. Kernel-live used the **full profile**, not the shipped messaging baseline. This did not verify messaging end to end. |
| Packed model-turn gate | **Verified beta.5 candidate on Node22.22.3** — shipped messaging baseline, no-grant OS tools, independent gatekeeper ownership/backstop, grants, synthetic approvals and native denials. Separate from npm-only acceptance. |
| npm-only end-to-end checkpoint | **PASS beta.5**, run `20260916-220009-phase-3` on Node22.22.3 / OpenClaw2026.9.2: registry identities/tags/times/archive SHA5125/5, actual cell creation and selector, install-policy14/14, kernel-live114/114, selected conformance38/38, owner-only72/72, independent approvals30/30; 25 local model turns /39 requests. No product source checkout/build/patch. |
| Deferred approval + simulation | **Source-install/full-profile evidence** on the fs driver (pending writes appear in later reads; reject removes them); beta.5 additionally passed npm-only messaging-baseline apply/reject with recorded synthetic effects and audit. **Implemented and passing** on GitHub with a real Gateway and synthetic provider (100/103; comment pending → readback includes it → apply/reject/revert). |
| GitHub driver end-to-end against real GitHub | **Not yet.** Blocked on one upstream issue (below) plus a disposable test account. |
| Filesystem *writes* | **Simulate-only.** `gk_fs_file_write` queues and simulates; applying to disk is disabled until race-safe confinement is proven. You can inspect and reject, not apply. |
| Approvals TUI, blueprints, `gkos update`/rollback, MCP/HTTP drivers | **Not built.** Designed in `docs/implementation-plan.md`; placeholders only. |
| Real chat transports | Beta.5 npm-only owner-only audience checks passed72/72 using synthetic public-SDK ingress. Real Telegram remains deferred; real Slack acceptance is not claimed. |

**The upstream issue:** on unmodified OpenClaw 2026.9.2, when a native `requireApproval` is denied or has no approval route, the Gateway logs the tool's raw arguments before any plugin runs. Only the optional synchronous path is affected; it ships off by default. Reported to the OpenClaw maintainers.

## Try it (on a disposable machine)

The five packages `@gatekeeper-os/{shared,gatekeeper-kit,kernel,gatekeeper-fs,cli}`
are published at beta.5; `@beta` selects that line. Use Node22.22.3+ on a disposable
evaluation machine. No ClawHub listing is claimed; no stable release exists.

```sh
npm install --global @gatekeeper-os/cli@beta
gkos --version
gkos cell create evaluation --port 19100 --policy messaging
```

Beta.5 passed the snapshot-based npm-only checkpoint against the published artifacts.
The independent synthetic gatekeeper demonstrated apply/reject effects and audit;
real filesystem writes and real connected-provider/channel acceptance remain out
of scope. Earlier failed runs remain recorded as failures in core's
[PROGRESS](https://github.com/gatekeeper-os/gatekeeper-os/blob/main/plans/PROGRESS.md).

```bash
git clone https://github.com/gatekeeper-os/gatekeeper-os.git && cd gatekeeper-os
./installer/install.sh
gkos status --json
gkos grant add --agent <agent> file:///home/you/projects/some-folder
gkos audit tail --limit 20 --json
```

In the source-install/full-profile acceptance fixture, a `file://` URL introduced by
the operator enabled scoped `gk_fs_*` reads. The shipped messaging baseline is also
covered by the passing beta.5 npm-only checkpoint above; this does not enable real filesystem writes.

## Lineage

The gatekeeper model — vendor → account → resource, approval queues, simulation, observer verification — is ported from [Cloudflare OS](https://github.com/cloudflare/cloudflare-os) (Apache-2.0) and adapted for a self-hosted, tool-calling runtime. Read their README for the original argument; it's the reason this project exists.

## What it does and doesn't defend against

It stops an agent from reaching resources it wasn't introduced to, from performing external side effects nobody reviewed, and from doing either silently. It does **not** eliminate prompt injection through a granted resource (it shrinks the blast radius to what was granted), it does not sandbox a malicious plugin (native plugins run in the Gateway process), and it does not provide hostile multi-tenant isolation inside one Gateway — for that, run separate cells, as OpenClaw itself recommends. Full write-up: [`docs/threat-model.md`](https://github.com/gatekeeper-os/gatekeeper-os/blob/main/docs/threat-model.md).

## Get involved

- Want a gatekeeper for a service we don't have? Open a [gatekeeper request](https://github.com/gatekeeper-os/gatekeepers/issues/new?template=gatekeeper-wanted.yml) or build one with the skill.
- Security issues: see [SECURITY.md](https://github.com/gatekeeper-os/.github/blob/main/SECURITY.md). Please don't open public issues for vulnerabilities.
- Code is MIT, like OpenClaw itself; the parts adapted from Cloudflare OS carry their Apache-2.0 notice (see `NOTICE`).
