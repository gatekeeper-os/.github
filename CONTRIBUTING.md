# Contributing

Thanks for looking. Two repos, two bars:

- **`openclaw-os`** (kernel, contracts, kit, CLI, conformance, reference gatekeepers) is held to the *kernel bar*: reviewers read every line, every exported member is doc-commented, and any change that touches how a grant is resolved or how a `gk_*` tool is registered gets a security-focused review first. Expect slower merges here; that's deliberate.
- **`gatekeepers`** (community drivers) is held to a normal bar, with one hard rule: every outside-world interaction must go through `authorizeObservation()` or `submitAction()`. The conformance check enforces it.

## Before you open a PR

1. Read `AGENTS.md` and `REVIEW.md` in the repo you're changing. They're short and they're the rules.
2. Run `pnpm build && pnpm test`. For gatekeepers, also `pnpm conformance --gatekeeper <vendor>`.
3. Never import from `openclaw/*` other than the documented `openclaw/plugin-sdk/*` subpaths. Never read upstream's SQLite. Never write under the upstream install root.
4. Never log secrets, prompts, headers, tokens, or request/response bodies. CI greps for it; reviewers grep harder.

## Building a gatekeeper

Follow `.agents/skills/write-gatekeeper/SKILL.md` in the `gatekeepers` repo. It has two STOP points where you (or your agent) present the tool surface for review before writing the implementation — the tool surface is the hard part to change later, so we review it first.

AI-assisted PRs are welcome. Say so in the description, and make sure a human has read the diff before you open it.

## License

By contributing you agree your contribution is licensed under MIT, the project's license.
