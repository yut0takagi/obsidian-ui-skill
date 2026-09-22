---
name: obsidian-ui
description: "Build React/Next.js sites and pages using ObsidianUI (obsidianui.dev) — 41 animated components (buttons, menus, galleries, scroll/text/cursor animations, canvas and WebGL backgrounds) plus 63 shadcn primitives, installed as owned source via the shadcn CLI. Use when the user names ObsidianUI or obsidianui.dev, asks for a site/landing page built with it, wants one of its components (hover image, scroll stack, book flip, magnetic trail, art gallery, dither canvas …), or wants animated React UI and has this library available. 日本語トリガー: 「ObsidianUIで作って」「obsidianui のコンポーネントで」「あのUIライブラリでサイト作って」「アニメーション付きのReact UI」。"
---

# ObsidianUI

Component library of 41 animated React components served as a **shadcn-compatible
registry**. You install source into the project and own it — there is no runtime
package to import from, no `<ObsidianProvider>`, no theme object.

Registry: `https://www.obsidianui.dev/r/{name}.json` · Catalogue: `/llms.txt` ·
Upstream agent contract: `/agent-instructions.md` (MIT, GitLab `Atharvsinh-codez/ObsidianUI`)

## Pick the route first

| Situation | Route |
|---|---|
| Project already has `components.json` (shadcn) | **CLI install** — §2 |
| React project, no shadcn | **Init shadcn, then CLI** — §1 then §2 |
| No project yet | **Scaffold** — §1 |
| "Which component does X?" | `references/components.md`, don't guess names |
| Component installed but blank / broken / janky | `references/pitfalls.md` **before** editing source |
| Build a whole page or landing site | §4 composition rules |

**Never invent component names.** The catalogue is fixed at 41 documented
components. If the user asks for something not in `references/components.md`,
say so and offer the nearest real one — a wrong slug 404s the CLI.

## 1. Project setup

Tailwind **v4**, CSS-first. There is no `tailwind.config.js` content array to edit.

```bash
npx create-next-app@latest my-app --typescript --tailwind --eslint --app --src-dir --import-alias '@/*'
cd my-app
```

Adding Tailwind to an existing project:

```bash
npm install tailwindcss @tailwindcss/postcss postcss
```

```javascript
// postcss.config.mjs
export default { plugins: { "@tailwindcss/postcss": {} } };
```

```css
/* src/app/globals.css */
@import "tailwindcss";
```

Baseline deps and the `cn` helper every component expects at `@/lib/utils`:

```bash
npm install motion clsx tailwind-merge lucide-react
```

```typescript
// src/lib/utils.ts
import { ClassValue, clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

Then `npx shadcn@latest init` if `components.json` does not exist — the CLI
needs it to resolve `@ui/`, `@components/`, `@lib/`, `@hooks/` targets.

Optional, lets the short `@obsidian/name` form work:

```json
{ "registries": { "@obsidian": "https://www.obsidianui.dev/r/{name}.json" } }
```

## 2. Installing a component

```bash
npx shadcn@latest add "https://www.obsidianui.dev/r/<slug>.json"
```

Several in one command (preferred — one dependency resolution, one write):

```bash
npx shadcn@latest add "https://www.obsidianui.dev/r/hover-img.json" "https://www.obsidianui.dev/r/flip-text.json"
```

With the `registries` entry from §1 configured, the short form also works:

```bash
npx shadcn@latest add @obsidian/magnet-tabs
```

Quote the URL — unquoted `?`/`&` get eaten by the shell. Installing is a
**write to the user's project**: name the components before running it, and
prefer one command listing several slugs over an unattended loop.

Inspect before installing when you need deps or targets. Write to a file and read
it — piping `curl` into a parser is fragile, and some shells/proxies rewrite it:

```bash
curl -sL -o /tmp/c.json https://www.obsidianui.dev/r/scroll-stack.json
python3 -c "import json; d=json.load(open('/tmp/c.json')); print(d['dependencies'], [f['target'] for f in d['files']])"
# → ['gsap'] ['@components/block/scroll-stack.jsx', '@lib/effects/scroll-stack/styles.css']
```

Manual install, only when the CLI is unavailable — resolve targets through
`components.json` aliases (`@ui/x.tsx` → `src/components/ui/x.tsx`). Never
create a literal `@ui` directory. Copy **every** `files[]` entry, not just the
`.tsx`: CSS, hooks, shaders and `public/` assets are load-bearing.

### Post-install checklist

Run this every time. Each line catches a failure that is otherwise **silent** —
all four were reproduced on a clean Next.js 16 / React 19.2 project:

```bash
# 1. assets the CLI misplaced into src/public/ (they would 404) — pitfalls §1
[ -d src/public ] && mkdir -p public && cp -R src/public/. public/ && rm -rf src/public

# 2. ObsidianUI-hosted demo media that does not exist in this project — pitfalls §1
grep -rn '/cdn/' src/

# 3. compile + build
npx tsc --noEmit && npm run build

# 4. actually look at it
npm run dev
```

`tsc --noEmit` is **not** sufficient evidence: it skips the 23 `.jsx` components
entirely (measured — a component with 9 real type errors under `checkJs` passes
silently). A green build says nothing about whether the animation runs or the
images resolved. Verified case: a page can build, prerender **and** serve HTTP 200
while every image on it is broken.

## 3. Non-negotiables after install

- **Keep `"use client"`.** Every animated component is client-side. Dropping it in
  the App Router produces a hydration or `useRef`-of-undefined error.
- **Keep the CSS import.** Components marked *ships .css* in the reference are not
  pure-Tailwind; deleting the import silently removes the animation.
- **Replace `/cdn/` demo media.** 8 components hardcode ObsidianUI-hosted paths that
  do not exist in the user's project. Blank component = almost always this. See pitfalls.
- **Respect reduced motion.** Cursor, scroll and WebGL effects should no-op under
  `prefers-reduced-motion: reduce`. The library does not do this for you everywhere.
- **Expect `.jsx`, not `.tsx`.** 23 of 41 components ship untyped JSX with JSDoc
  `@param` annotations. They compile in a TS project (`allowJs: true`) and editors
  still surface prop hints, but they are **not** type-checked. Under
  `checkJs: true` or a strict lint gate they will error — convert the file to
  `.tsx` and type the props, or exempt the path. See pitfalls §10.
- **Import by the right shape.** `.tsx` components export both default and named;
  the 23 `.jsx` ones are **named-only**. `next/dynamic` on a `.jsx` component needs
  `.then((m) => m.Name)` or it fails typecheck.
- **Never pass `as={Link}` from a Server Component.** These are Client Components,
  so a component-valued prop crosses the RSC boundary and the build fails with
  "Functions cannot be passed directly to Client Components". Wrap it in a
  `"use client"` component. Wrapping in `<Link>` is not the fix — they render an
  `<a>` already. Pitfalls §5.
- **Check required props.** 5 components take a data prop with no default and
  render nothing without it: `masonry-grid` / `flow-scroll` / `horizontal-scroll` /
  `scroll-effect` (an `items`/`images` array) and `magnet-tabs` (`options`).
  `FlipText` requires `children`. An empty component is not always a broken one —
  read the usage example in the component's doc page first.
- **Test the interaction, not the compile.** Copying code is not evidence it works.
  Check real content, keyboard, and the target viewport.

## 4. Composing a page

Animation budget is the whole game — these components are individually heavy and
compound badly.

- **One hero effect per page.** WebGL/`three` components (Art Gallery, Book Flip,
  Fractal Glass, Curved Plane, Hover Slider) each own a canvas and a rAF loop.
  Two on one screen is a frame-rate problem, not a design choice.
- **At most one cursor effect per site**, mounted once in the layout. Two
  competing `mousemove` trails read as a bug.
- **Watch the engine split.** 22 components use `motion`, 12 use `gsap`. Mixing is
  fine but ships both (~47 KB + ~45 KB gzip); if everything you need is in one
  engine, stay there.
- **Prefer raw-three over fiber** when you only need one WebGL moment. The
  `@react-three` stack costs ~260 KB gzip vs ~55 KB for `three` alone — measured,
  pitfalls §5. `art-gallery` / `fractal-glass` / `curved-plane` /
  `interactive-hover-slider` are raw three; `book-flip` / `butterfly-trail-cursor`
  pull in fiber + drei.
- **Lazy-load below-the-fold WebGL**, but `ssr: false` cannot sit in a Server
  Component — it needs a `"use client"` wrapper, and `.jsx` components need
  `.then((m) => m.Name)`. Exact pattern in pitfalls §5.
- Scroll components assume they own the scroll container. Combining several, or
  adding Lenis smooth-scroll on top, needs deliberate nesting — see pitfalls.
- **Match the component to the job, not the mood.** Several animate over their own
  content indefinitely — `pixelated-carousel` keeps ~half its tiles opaque at all
  times (measured), so the image is never clean. Use those for atmosphere, and a
  plain `next/image` wherever the user has to actually see or compare something.
  Pitfalls §12 lists the sizing traps too (`masonry-grid` is `max-w-4xl` with no
  links; `trading-card` is fixed 300×400).

Suggested landing-page spine, ~3 installs: a text reveal for the headline, one
scroll animation for the body, one WebGL or canvas piece as the single hero moment.

This exact shape is verified to build and render (Next.js 16.3 / React 19.2):

```tsx
// src/app/page.tsx — stays a Server Component
import { FlipText } from "@/components/block/flip-text";
import { HoverImg } from "@/components/block/hover-img";
import { ArtGalleryLazy } from "@/components/lazy/art-gallery-lazy";

export default function Home() {
  return (
    <main>
      <FlipText>Ship better</FlipText>
      <HoverImg />
      <ArtGalleryLazy />
    </main>
  );
}
```

with the lazy wrapper from pitfalls §5. Remember `HoverImg`'s demo images are
`/cdn/` paths — replace them or the section renders blank.

## 5. When the site is the deliverable

If the user wants a finished page rather than wired components, still install real
ObsidianUI source — do not hand-write lookalike components and call them ObsidianUI.
Get it running (`npm run dev`) and verify in a browser before reporting done.

## References

- `references/components.md` — all 41 components: slug, deps, file count, per-component
  traps, and the dependency footprint table. Read before choosing.
- `references/pitfalls.md` — diagnosis-first list of the failures this library
  actually produces (blank renders, misplaced assets, RSC boundary errors, double
  scroll, WebGL context limits, decorative-vs-content mismatches, alias mistakes).
- `references/agent-contract.md` — upstream `/agent-instructions.md` verbatim, plus the
  read-only HTTP surface and licensing.
