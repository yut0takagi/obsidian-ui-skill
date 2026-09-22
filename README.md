# obsidian-ui — a Claude Code skill for ObsidianUI

Build React/Next.js sites with [ObsidianUI](https://www.obsidianui.dev/) — 41
animated components installed as source you own, via the shadcn CLI.

Unofficial and independent. Not affiliated with the ObsidianUI project.

ObsidianUI already publishes [`agent-instructions.md`](https://www.obsidianui.dev/agent-instructions.md)
for agents, and this skill keeps that file verbatim. What it adds is the part the
docs don't cover: **the failure modes, measured on real installs.** Most of them
are silent — the build passes, the page returns 200, and the component is broken.

## Install

```bash
git clone <this-repo> obsidian-ui-skill
ln -s "$PWD/obsidian-ui-skill/obsidian-ui" ~/.claude/skills/obsidian-ui
```

Project-scoped instead: symlink into `.claude/skills/` in your repo. Restart
Claude Code — skills load at session start.

Then just ask for what you want: *"build a landing page with ObsidianUI"*,
*"add the scroll-stack component"*. The skill's description routes on
`obsidianui.dev`, component names, and animated-React-UI requests.

## Contents

| File | What it holds |
|---|---|
| `SKILL.md` | Routing, setup, install commands, post-install checklist, composition rules |
| `references/components.md` | All 41 components: slug, deps, file count, per-component traps |
| `references/pitfalls.md` | 13 sections of measured failure modes, symptom-first |
| `references/agent-contract.md` | Upstream `agent-instructions.md` verbatim + the read-only HTTP surface |

## What it catches

A sample of things verified by installing components and looking at the result:

- **8 components hardcode `/cdn/` demo paths** that 404 in your project. The most
  common cause of a blank component. Confirmed live: page builds, returns 200, every
  image `naturalWidth === 0`.
- **The CLI writes `public/` assets to `src/public/`** in a `--src-dir` project, so
  they never serve — contradicting upstream's own docs. Reproduced on two clean projects.
- **23 of 41 components ship untyped `.jsx`** (with JSDoc). `tsc --noEmit` passes by
  skipping them; `checkJs: true` surfaced 9 errors in one file.
- **`ssr: false` can't sit in a Server Component**, and `.jsx` components are
  named-export-only, so `next/dynamic` needs `.then((m) => m.Name)`.
- **`as={Link}` from a Server Component fails the build** — a component-valued prop
  can't cross the RSC boundary.
- **`@react-three/fiber` pins React `<19.3`.** Fine on today's `create-next-app`
  (19.2.x); ERESOLVE if you've upgraded.
- **`pixelated-carousel` keeps ~half its tiles opaque forever**, so the image is
  never clean. A hero effect, not a product gallery.
- **22 of 41 animated components have no `prefers-reduced-motion` guard** — including
  all 6 cursor effects. The other 18 already handle it, and the skill says which, so
  you don't patch working code.
- **Chrome caps live WebGL contexts at 16** and evicts the oldest; the practical
  limit is frame budget, not the cap.

## Provenance

Counts come from parsing all 104 `/r/registry.json` items. Behavioural claims were
measured on Next.js 16.3 / React 19.2.8 / Tailwind v4 / shadcn CLI 4.21 /
three 0.186 / fiber 9.7 / drei 10.7 / gsap 3.15 / motion 13.4 / Chrome 153.

Where the library already does the right thing, the skill says so — the goal is
fewer wrong edits, not more.

## Staying current

Nothing auto-updates. The catalogue can grow, and versions move:

```bash
curl -sL https://www.obsidianui.dev/agent-instructions.md   # refresh the contract
curl -sL -o /tmp/registry.json https://www.obsidianui.dev/r/registry.json
```

If counts in the docs disagree with the registry, trust the registry and open an
issue or PR. Upstream is authoritative for install mechanics; this skill is
authoritative only for the failures it measured.

## Licence

MIT — see [`LICENSE`](LICENSE).

[`NOTICE.md`](NOTICE.md) covers the third-party side: ObsidianUI's own MIT notice for
the reproduced agent instructions, and the fact that demo media on obsidianui.dev
(images, `.glb` models, HDR maps) is **not** MIT. Swap those for your own assets.
