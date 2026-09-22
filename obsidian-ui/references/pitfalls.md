# ObsidianUI pitfalls

Diagnosis-first. Every claim here was measured, not inferred — by parsing all 104
`/r/registry.json` items and by installing components into real projects. Where the
library already handles something, that is stated too, so you don't "fix" code that
is already correct.

Verified against: Next.js 16.3 (Turbopack), React 19.2.8, Tailwind v4,
shadcn CLI 4.21, three 0.186, @react-three/fiber 9.7, drei 10.7, gsap 3.15,
motion 13.4, Chrome 153. Counts are current as of the fetch date in
`agent-contract.md`; re-derive them from the registry if the catalogue grows.

## Symptom → cause

| Symptom | First thing to check |
|---|---|
| Component renders nothing / empty box | Hardcoded `/cdn/` demo media — §1 |
| Images 404 though the files installed | CLI wrote them to `src/public/` — §1 |
| `npx shadcn add` fails, 404 | Wrong slug, or URL not quoted — §2 |
| Literal `@ui/` folder appeared | Manual copy without alias resolution — §3 |
| Animation absent, no console error | Deleted the component's `.css` import — §4 |
| Hydration mismatch / `useRef` undefined | `"use client"` removed, or SSR'd WebGL — §5 |
| Two scrollbars, fighting scroll | Stacked scroll components / Lenis — §6 |
| Frame rate collapses | More than one WebGL canvas — §7 |
| Motion plays despite OS reduce setting | 22 components have no guard — §8 |
| Strict `tsc` / lint fails on new files | 23 components ship untyped `.jsx` — §10 |
| Build: "Functions cannot be passed to Client Components" | `as={Link}` from a Server Component — §5 |
| Image never fully visible / looks half-covered | decorative overlay loops forever — §12 |
| `npm install` ERESOLVE on a 3D component | React is 19.3+; fiber needs `<19.3` — §11 |

## 1. `/cdn/` demo media (most common blank render)

8 components hardcode ObsidianUI-hosted, **root-relative** paths in their demo
data. These resolve against *your* domain, where the files don't exist, so images
silently 404 and the component looks broken.

| component | `/cdn/` paths |
|---|---|
| `magnetic-image-trail` | 11 |
| `curved-plane` | 5 |
| `interactive-hover-slider` | 5 |
| `hover-img` | 3 |
| `dither-canvas` | 2 (video + poster) |
| `book-flip` | 2 (incl. `studio.hdr` env map) |
| `interactive-blur-reveal` | 2 (incl. a noise texture) |
| `grid-lift` | 1 (wordmark) |

Swap every one for the user's asset before calling the component done:

```bash
grep -rn '/cdn/' src/components/
```

Two caveats when replacing:

- `book-flip` needs a real **`.hdr`** environment map; a `.jpg` will not light the scene.
- `interactive-blur-reveal` and `magnetic-image-trail` include **noise/distortion
  textures** that are part of the effect, not content. Replacing them with a photo
  breaks the shader. Keep them, or regenerate equivalent noise.

Separately, `butterfly-trail-cursor` and `fractal-glass` load **absolute**
`https://www.obsidianui.dev/...` URLs (`meta.remoteAssets`), including a `.glb`
model. Those do render, but you're hotlinking a third party — self-host for
production or offline use.

`visitor-count` declares `meta.requiredEndpoints: ["/api/visitors"]`. You implement
that route; the registry does not supply it.

### The CLI puts `public/` assets in the wrong place

Verified on a default `create-next-app --src-dir` project. Installing
`folder-preview` writes its 5 avatars to:

```text
src/public/folder-preview/user1.svg     ← where the CLI put them
public/folder-preview/user1.svg         ← where Next.js serves from
```

The component requests `/folder-preview/user1.svg`, so **every asset 404s** and you
get another silent blank. Upstream's own guide says "a `public/` target is relative
to the project root" — the CLI does not honor that in a `--src-dir` layout.

Fix after any install that ships assets:

```bash
[ -d src/public ] && mkdir -p public && cp -R src/public/. public/ && rm -rf src/public
```

Confirmed: after moving, `/folder-preview/user1.svg` returns **200** on
`next start`. Affects `folder-preview` today — check any component whose manifest
has a `public/` target.

## 2. CLI install

```bash
# right
npx shadcn@latest add "https://www.obsidianui.dev/r/scroll-stack.json"
# wrong — unquoted, and a guessed slug
npx shadcn@latest add https://www.obsidianui.dev/r/scroll-stack.json?v=3
```

Slugs are not always what the display name suggests. Confirm against
`components.md`:

- "Hover Image" → `hover-img`
- "Hover Slider" → `interactive-hover-slider`
- "Marquee on SVG Path" → `svg-path-marquee`

Never invent a name to recover from a 404. Re-read `/llms.txt` or
`/r/registry.json`.

## 3. Manual install and aliases

Targets use `@ui/`, `@components/`, `@lib/`, `@hooks/`. These are **registry
placeholders**, resolved through the destination project's `components.json`
aliases — not TypeScript paths to copy literally.

```text
@ui/button.tsx              → src/components/ui/button.tsx
@components/block/hover-img.tsx → src/components/block/hover-img.tsx
@lib/utils.ts               → src/lib/utils.ts
public/folder-preview/*.svg → public/folder-preview/*.svg   (project root)
```

If a directory named `@ui` exists, the resolution step was skipped — that is the bug.

Copy **every** `files[]` entry. `folder-preview` writes 7 files (5 are SVGs in
`public/`); `interactive-arrows` writes 9. Copying only the `.tsx` yields missing
imports or a component with no content.

## 4. Stylesheets are load-bearing

These ship a real `.css` file and import it:

`arrow-fill-button`, `hover-img`, `text-fill-animation`, `text-stream`,
`draggable-marquee`, `scroll-effect`, `scroll-stack`, `svg-pixel-reveal`

Removing the import to "convert to pure Tailwind" compiles clean and kills the
animation. Keyframes and clip-paths live in that file.

## 5. Client boundaries

Every documented component already ships `"use client"` — verified across all 41.
Don't add it; don't remove it.

Real SSR risk is mounting WebGL in a server-rendered tree. Lazy-load it — but the
obvious one-liner **does not build** in the App Router:

```tsx
// ❌ page.tsx is a Server Component — Turbopack build error:
// "`ssr: false` is not allowed with `next/dynamic` in Server Components."
const ArtGallery = dynamic(() => import("@/components/block/art-gallery"), { ssr: false });
```

`ssr: false` must live inside a Client Component. Add a one-line wrapper — and note
the `.then()`, because **`.jsx` components have no default export** (§10):

```tsx
// src/components/lazy/art-gallery-lazy.tsx
"use client";
import dynamic from "next/dynamic";

export const ArtGalleryLazy = dynamic(
  () => import("@/components/block/art-gallery").then((m) => m.ArtGallery),
  { ssr: false }
);
```

```tsx
// src/app/page.tsx — stays a Server Component
import { ArtGalleryLazy } from "@/components/lazy/art-gallery-lazy";
export default function Home() { return <ArtGalleryLazy />; }
```

### Passing `as={Link}` from a Server Component fails the build

Several components (`arrow-fill-button`, others that render an element you can
swap) accept an `as` prop. They are Client Components, so handing them a
component from a Server Component crosses the RSC boundary with a function:

```
Error: Functions cannot be passed directly to Client Components unless you
explicitly expose it by marking it with "use server".
  {as: function i, href: "/shop", ...}
```

Reproduced on Next.js 16.3 by rendering `<ArrowFillButton as={Link} href="/shop">`
from `app/page.tsx`. Do the wiring in a Client Component instead:

```tsx
// src/components/site/cta-button.tsx
"use client";
import Link from "next/link";
import { ArrowFillButton } from "@/components/block/arrow-fill-button";

export function CtaButton({ href, children }: { href: string; children: string }) {
  return <ArrowFillButton as={Link} href={href}>{children}</ArrowFillButton>;
}
```

Wrapping the button in a `<Link>` instead is not the fix: these buttons render an
`<a>` by default, and an `<a>` inside an `<a>` is invalid HTML. Use `as`, from the
client side.

Export shape splits cleanly by extension, verified across 10 installs:

| files | export |
|---|---|
| `.tsx` (18 components) | default **and** named — either import works |
| `.jsx` (23 components) | **named only** — `dynamic()` needs `.then((m) => m.Name)` |

Verified end to end on Next.js 16.3 + React 19.2: this wrapper pattern compiles,
prerenders, and the canvas initializes live (`isContextLost() === false`).

This also keeps the animation engine out of the initial bundle. Measured with
esbuild `--bundle --minify`, React externalized (three 0.186, fiber 9.7, drei 10.7,
gsap 3.15, motion 13.4):

| import | minified | gzip |
|---|---:|---:|
| `motion/react` (motion + scroll hooks) | 140 KB | **47 KB** |
| `gsap` + `ScrollTrigger` | 114 KB | **45 KB** |
| `three` alone | 245 KB | **55 KB** |
| `three` + `@react-three/fiber` + `@react-three/drei` | 948 KB | **260 KB** |

The `@react-three` stack is the outlier — roughly 5× the gzip cost of either
2D engine. That is `butterfly-trail-cursor` and `book-flip`. The four raw-three
components (`curved-plane`, `fractal-glass`, `interactive-hover-slider`,
`art-gallery`) cost far less because they skip fiber and drei.

Drei is also imported by name, so the real figure depends on which helpers a
component pulls; treat 260 KB as the ceiling for a full fiber+drei page, not a
fixed tax.

## 6. Scroll ownership

6 components register GSAP **ScrollTrigger**: `rectangular-text-reveal`,
`text-fill-animation`, `parallax-gallery`, `scroll-stack`, `svg-pixel-reveal`,
plus `draggable-marquee` (registerPlugin only).

Consequences:

- Adding Lenis or any smooth-scroll library requires wiring it to ScrollTrigger
  (`ScrollTrigger.update` on Lenis scroll, and `scrollerProxy` for a custom
  container). Without that, triggers fire at the wrong offsets.
- `horizontal-scroll`, `flip-scroll`, `flow-scroll` and `scroll-stack` each assume
  they own the vertical scroll of their section. Nesting two is a layout decision
  you must make explicitly; they will not negotiate.
- Pinned sections need a real height. Inside a `height: 100%` / flex parent that
  collapses, the pin jumps.

## 7. WebGL budget

`three` appears in 6 components — `butterfly-trail-cursor`, `book-flip` (both via
`@react-three/fiber`), and `curved-plane`, `fractal-glass`,
`interactive-hover-slider`, `art-gallery` (raw three).

All six **do** call `dispose()`, and the raw-three ones **do**
`cancelAnimationFrame` — teardown is handled; you don't need to patch it.

The real constraint is concurrency: each owns a GL context and a render loop.
Browsers cap live contexts per page and evict the **oldest** when you exceed it.

Measured in Chrome 153: the cap is **16**. Acquiring the 17th silently kills
context #1 — `gl.isContextLost()` flips to true, the canvas goes blank, and
nothing throws. Requesting 40 leaves exactly the newest 16 alive.

You will not hit 16 with ObsidianUI components alone, so the practical rule is
about frame budget, not the cap: each one runs its own `requestAnimationFrame`
loop, so two full-viewport WebGL pieces on one screen compete for the same
16 ms. Keep it to **one per page** and lazy-load it (§5). If you ever do see a
3D component blank out with a clean console, contexts are the first suspect.

Mixing `@react-three/fiber` components with raw-three components is fine —
separate canvases — but `three` must be a single version in the tree. All registry
deps are **unpinned**, so a stale lockfile can pair a new `@react-three/fiber`
with an old `three` and throw on import.

## 8. Reduced motion

18 of 41 components respect `prefers-reduced-motion`. **22 animated ones do not**,
including every cursor effect:

`folder-preview`, `hover-img`, `masonry-grid`, `pixelated-carousel`,
`apple-spotlight`, `circle-menu`, `magnet-tabs`, `split-showcase`, `trading-card`,
`jelly-loader`, `otp-input`, `flip-scroll`, `flow-scroll`,
`glowing-scroll-indicator`, `horizontal-scroll`, `svg-path-marquee`,
`parallax-gallery`, `butterfly-trail-cursor`, `colorful-cursor-aura`,
`interactive-arrows`, `mask-cursor-effect`, `rope-cursor`

For anything full-screen or cursor-following, add a guard:

```tsx
const reduced = useReducedMotion();        // from "motion/react"
if (reduced) return <>{children}</>;       // or render the static frame
```

CSS-level fallback:

```css
@media (prefers-reduced-motion: reduce) {
  .obsidian-effect { animation: none; transition: none; }
}
```

## 9. Cursor effects specifically

All 6 clean up their listeners (`removeEventListener` verified), so they don't leak.
Remaining judgment calls:

- Mount **one**, in the layout — not per page. Two trails look like a bug.
- They assume a pointer. On touch, they are dead weight; gate on
  `matchMedia("(pointer: fine)")`.
- Several draw a fixed full-viewport overlay. If it swallows clicks, the fix is
  `pointer-events: none` on the canvas, not restructuring your page.

## 10. `.jsx` in a TypeScript project

23 of the 41 components ship **`.jsx`**, not `.tsx`:

`arrow-fill-button`, `rectangular-text-reveal`, `text-fill-animation`,
`text-stream`, `draggable-marquee`, `svg-path-marquee`, `parallax-gallery`,
`scroll-stack`, `svg-pixel-reveal`, `butterfly-trail-cursor`,
`colorful-cursor-aura`, `interactive-arrows`, `magnetic-image-trail`,
`rope-cursor`, `dither-canvas`, `dotted-grid`, `book-flip`, `curved-plane`,
`fractal-glass`, `grid-lift`, `interactive-hover-slider`,
`interactive-blur-reveal`, `art-gallery`

They are not type-hostile — props carry JSDoc annotations, so editors still
autocomplete:

```jsx
/** @param {{bgColor?: string, cards?: Array<{id: string|number, title: string}>, ...}} props */
export function ScrollStack({ bgColor = "bg-white", cards = [], ... }) {
```

What this means in practice:

- `create-next-app --typescript` sets `allowJs: true`, so **they build as-is**.
- They are **not** type-checked. `tsc --noEmit` passes by ignoring them.
- With `checkJs: true`, or an ESLint config that bans `.jsx`, or a CI gate
  requiring full type coverage, they **fail**. Either rename to `.tsx` and
  translate the JSDoc into real prop types, or exempt the path deliberately.
- Don't report "fully typed" after installing these. It isn't true.

Measured on a clean install of `scroll-stack` alone (strict, React 19 types):
`tsc --noEmit` → **0 errors**; adding `checkJs: true` → **9 errors** in that one
file (`TS7005` implicit `any[]`, `TS2339` on the `scroller` ref union, `TS2353`
on a CSS custom property, `TS2322` ref assignment). The JSDoc is close but not
strict-clean, so budget real work if you convert.

## 11. `@react-three/fiber` and the React 19.3 peer range

Not a problem on a default scaffold today — but a sharp edge worth knowing before
you debug it blind.

`@react-three/fiber@9.7.0` declares a **bounded** React peer range:

```json
"peerDependencies": { "react": ">=19 <19.3", "react-dom": ">=19 <19.3", "three": ">=0.156" }
```

Verified both sides:

- `create-next-app@latest` currently installs React **19.2.8**, which is inside
  that range — `npm install @react-three/fiber @react-three/drei three` succeeds
  with no flags. **This is the normal path and it works.**
- On a project already upgraded to React **19.3+**, the same install **fails**:

```
npm error code ERESOLVE
npm error Found: react@19.3.0
npm error   peerOptional expo@">=43.0" from @react-three/fiber@9.7.0
```

The `expo` line is a red herring — expo is an optional peer. The real conflict is
`react: ">=19 <19.3"`. `@react-three/drei` is innocent (it accepts `^19`); fiber
alone reproduces it.

If you hit it, two fixes, both verified:

```bash
# A — recommended: keep React inside fiber's range
npm install react@19.2 react-dom@19.2

# B — keep React 19.3+, override resolution
npm install --legacy-peer-deps @react-three/fiber @react-three/drei three
```

Prefer **A**. `--legacy-peer-deps` disables peer checking for the whole install,
not just this package, and must be repeated for every later install and in CI
(`npm ci`) — or pinned into `.npmrc`, where it silently affects the whole project.

This only concerns `butterfly-trail-cursor` and `book-flip`. The four **raw-three**
components (`curved-plane`, `fractal-glass`, `interactive-hover-slider`,
`art-gallery`) depend on `three` alone, install on any React version, and cost
~205 KB less gzip (the 260→55 KB gap in §5). If you need a WebGL moment rather
than a React scene graph, prefer those.

The shadcn CLI runs the install itself, so on an affected project
`npx shadcn add .../book-flip.json` surfaces the same ERESOLVE. Fix the React
version first, then add the component.

## 12. Decorative components are not content components

Some components animate *over* their own content forever. That reads as craft in
a hero and as a defect anywhere the content is the point.

**`pixelated-carousel`** is the clearest case. Measured while it ran: of 42 tiles,
**19–24 stayed opaque at every sample over 4 seconds**, because the overlay
animates with `repeat: Infinity`. The image underneath is never cleanly visible.

That is fine for a hero dissolve. It is wrong for anything a user needs to
inspect — a product photo, a chart, a document preview, an avatar. On a product
page it hides the thing being sold, which no amount of styling fixes.

Before reaching for one of these, ask what the image is *for*:

| the image is… | use |
|---|---|
| atmosphere, above the fold | `pixelated-carousel`, `dither-canvas`, WebGL pieces |
| something the user must see clearly | a plain `next/image` + your own thumbnails |
| something the user must compare | your own gallery; none of these ship one |

Related sizing traps found in the same pass:

- `pixelated-carousel` renders `h-full w-full` — the **parent** must have a height,
  or it collapses to nothing.
- `masonry-grid` hard-codes `max-w-4xl` and renders no links, so it cannot serve as
  a catalogue grid without editing the source.
- `trading-card` is a fixed **300×400** tilt card. A spotlight treatment, not a
  grid cell.

Editing these is expected — the registry installs source you own. Keep a one-line
comment saying what you changed and why, since the file no longer matches upstream.

## 13. Licensing

Repo is MIT — preserve the notice where required. Third-party packages and **demo
media** keep their own licenses: a demo does not grant rights to the images, the
`.glb`, or the HDR. Another reason §1 is mandatory before shipping.
