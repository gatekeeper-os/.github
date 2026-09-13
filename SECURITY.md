# Security policy

GatekeeperOS is a security layer, so we treat reports seriously and respond quickly.

## Reporting a vulnerability

Please **do not** open a public issue for anything that could be a vulnerability.

Use GitHub's private vulnerability reporting on the affected repository ("Security" tab → "Report a vulnerability"). If that isn't available to you, email the maintainers at the address on the org profile with the subject line `[GatekeeperOS security]`.

Include: the affected package and version, the OpenClaw version you ran against, steps to reproduce, and what an attacker gains. A minimal proof of concept is ideal; a full exploit is not needed.

## What counts

Anything that violates the project's stated invariants is in scope:

- An agent reaching a resource without an active grant (any path that bypasses `Kernel.resolveGrant` or the trusted tool policy).
- A side-effecting action performed without going through `submitAction()` and a human decision.
- Credentials, tokens, prompts, headers, or request/response bodies appearing in tool results, audit logs, or OS logs.
- A non-operator being able to introduce a resource or decide an approval.
- A gatekeeper asserting its own ambience (becoming available to an agent without operator configuration).
- Escaping a `gatekeeper-fs` root (symlink, `..`, encoding tricks).
- Simulation drift that lets an agent observe a side effect as applied when it was rejected.

Prompt injection *through* a granted resource is a known limitation, not a vulnerability, unless it lets the agent exceed the grant.

## What to expect

Acknowledgement within 3 days; a fix or mitigation plan within 14 days for confirmed issues; credit in the release notes if you want it. We'll ask you to keep the report private until a fix ships, and we'll coordinate disclosure timing with you.

## Supported versions

The latest minor release of each `@gatekeeper-os/*` package, against the OpenClaw versions listed in its `openclaw.compat.pluginApi` range.
