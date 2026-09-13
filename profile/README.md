# GatekeeperOS

GatekeeperOS is an independent project. It is not affiliated with or endorsed by the OpenClaw Foundation. OpenClaw is a trademark of its owner.

**capability-based access and deferred approvals for OpenClaw**

Stock OpenClaw, no fork, installed as plugins.

Each agent starts with access to nothing. You introduce a resource — a folder, a GitHub repo, an issue — by pasting its URL. From then on the agent reaches that resource only through a **gatekeeper**: a driver that logs every read, queues every write for your approval, and *simulates* the write locally so the agent keeps working while you approve later, in bulk, when it suits you.

No more choosing between "approve every single tool call" and `--dangerously-skip-permissions`.

**Status: GatekeeperOS rename; new-scope beta publication pending.** Kernel and filesystem driver accepted against a real Gateway in a VM; GitHub driver implemented and passing its integration suite, held from acceptance by one upstream logging issue. Details below — we'd rather you know exactly what's proven.

## How it works

- **Grants, not ACLs.** A grant is an opaque handle (`grant:7k3m9q2p`) the agent passes to gatekeeper tools. It never encodes the resource. Only an operator can create one; a URL pasted by anyone else is inert.
- **Reads are authorized and logged.** Every read goes through `authorizeObservation()` before data returns to the agent, and lands in an append-only audit log that never contains prompts, tokens, headers or bodies.
- **Writes are queued and simulated.** Every write goes through `submitAction()`, is journaled as pending, and is merged into the agent's subsequent reads as if it had happened. You apply, reject or revert it later — `gkos approvals apply 3`, or `/approvals apply 3` in a private operator chat.
- **Two gates, not one.** A host-level trusted tool policy denies any gatekeeper call without an active grant *before* any plugin hook runs; the kernel's `before_tool_call` then checks the grant belongs to this agent, session and audience. Gatekeepers can't register tools themselves and never see credentials in their results.
- **Upstream stays untouched.** Nothing in the `openclaw` package is patched or vendored. The upstream version is pinned in a lockfile and a conformance suite decides whether a release is compatible. (The one-command update/rollback pipeline is designed but not built yet — see status.)

## Repositories

| Repo | What it is | Review bar |
|---|---|---|
| [`gatekeeper-os`](https://github.com/gatekeeper-os/gatekeeper-os) | Kernel plugin, shared contracts, gatekeeper kit, `gkos` CLI, installer, conformance suite, and the `fs` and `github` reference gatekeepers | Kernel: every line reviewed |
| [`gatekeepers`](https://github.com/gatekeeper-os/gatekeepers) | Community gatekeepers — one folder per service — plus the `write-gatekeeper` skill any agent can follow to build one | Normal |

## What's proven, what isn't

| | |
|---|---|
| Kernel, `fs` driver, cells, installer, config reconcile, backups | **Accepted** — Phase 3: 78/78 conformance, 98/98 kernel-live checks, 23 real model turns on `openclaw@2026.9.2`; 410/410 host tests. Paste a `file://` URL → grant → `gk_fs_dir_list` → audit → `gkos grant revoke` removes the tool next turn; non-operators can't introduce. |
| Deferred approval + simulation | **Accepted** on the fs driver (pending writes appear in later reads; reject removes them). **Implemented and passing** on GitHub with a real Gateway and synthetic provider (100/103; comment pending → readback includes it → apply/reject/revert). |
| GitHub driver end-to-end against real GitHub | **Not yet.** Blocked on one upstream issue (below) plus a disposable test account. |
| Filesystem *writes* | **Simulate-only.** `gk_fs_file_write` queues and simulates; applying to disk is disabled until race-safe confinement is proven. You can inspect and reject, not apply. |
| Approvals TUI, blueprints, `gkos update`/rollback, MCP/HTTP drivers | **Not built.** Designed in `docs/implementation-plan.md`; placeholders only. |
| Real chat transports | Owner/observer logic proven with synthetic public-SDK ingress and the Control UI. Telegram deferred; Slack untested. |

**The upstream issue:** on unmodified OpenClaw 2026.9.2, when a native `requireApproval` is denied or has no approval route, the Gateway logs the tool's raw arguments before any plugin runs. Only the optional synchronous path is affected; it ships off by default. Reported to the OpenClaw maintainers.

## Try it (on a disposable machine)

The five packages `@gatekeeper-os/{shared,gatekeeper-kit,kernel,gatekeeper-fs,cli}`
are being prepared for `0.1.0-beta.2`; they are not published under this scope yet.
The repositories remain private pending the coordinated public release. No ClawHub
listing is claimed. After publication, use `@beta` explicitly; no stable release exists.

The following registry commands are for **after the new-scope beta is published**:

```sh
npm install --global @gatekeeper-os/cli@beta
gkos --version
gkos cell create evaluation --port 19100 --policy messaging
```

A clean-prefix CLI install/version smoke passed for the previous beta; npm-only fresh-VM acceptance of the new-scope beta is
still a separate gate. Keep the source installer as the VM evaluation route
(repository access required); core's `docs/vm-testing.md` has the exact commands.
All "what isn't proven" limitations above remain in force.

```bash
git clone https://github.com/gatekeeper-os/gatekeeper-os.git && cd gatekeeper-os
./installer/install.sh
gkos status --json
gkos grant add --agent <agent> file:///home/you/projects/some-folder
gkos audit tail --limit 20 --json
```

Or just paste that `file://` URL to your agent in a private chat. It can now list and read that one folder through `gk_fs_*` tools and nothing else.

## Lineage

The gatekeeper model — vendor → account → resource, approval queues, simulation, observer verification — is ported from [Cloudflare OS](https://github.com/cloudflare/cloudflare-os) (Apache-2.0) and adapted for a self-hosted, tool-calling runtime. Read their README for the original argument; it's the reason this project exists.

## What it does and doesn't defend against

It stops an agent from reaching resources it wasn't introduced to, from performing external side effects nobody reviewed, and from doing either silently. It does **not** eliminate prompt injection through a granted resource (it shrinks the blast radius to what was granted), it does not sandbox a malicious plugin (native plugins run in the Gateway process), and it does not provide hostile multi-tenant isolation inside one Gateway — for that, run separate cells, as OpenClaw itself recommends. Full write-up: [`docs/threat-model.md`](https://github.com/gatekeeper-os/gatekeeper-os/blob/main/docs/threat-model.md).

## Get involved

- Want a gatekeeper for a service we don't have? Open a [gatekeeper request](https://github.com/gatekeeper-os/gatekeepers/issues/new?template=gatekeeper-wanted.yml) or build one with the skill.
- Security issues: see [SECURITY.md](https://github.com/gatekeeper-os/.github/blob/main/SECURITY.md). Please don't open public issues for vulnerabilities.
- Code is MIT, like OpenClaw itself; the parts adapted from Cloudflare OS carry their Apache-2.0 notice (see `NOTICE`).
