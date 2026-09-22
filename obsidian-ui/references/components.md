# ObsidianUI component reference

Generated from `/r/registry.json` and `/markdown/docs/{slug}.md`. 41 documented components.
The registry holds 104 items total — the remaining 63 are stock shadcn primitives
(button, dialog, input, select …) served under the same `/r/{name}.json` scheme.

Install any row:

```bash
npx shadcn@latest add "https://www.obsidianui.dev/r/<slug>.json"
```

**deps** omits `clsx` / `tailwind-merge`, which almost every component pulls in through `cn`.
**files** is how many files the manifest writes — anything above 1 means helpers, CSS, or assets
come along, so don't hand-copy just the `.tsx`.

Notes decode as:

- **ships .css** — a real stylesheet is written and imported; Tailwind classes alone won't reproduce it.
- **ships assets** — files land in `public/`.
- **`/cdn/` demo media** — the usage example points at ObsidianUI-hosted paths that **do not exist in your project**. Swap them for your own or the component renders blank. See `pitfalls.md`.
- **remote assets** — loads absolute `https://www.obsidianui.dev/...` media at runtime (offline-hostile).
- **needs API route** — you implement the endpoint yourself.

## Buttons, menus & cards

| component | slug | deps | files | notes |
|---|---|---|---|---|
| **Arrow Fill Button** | `arrow-fill-button` | — | 3 | ships .css |
| **Folder Preview** | `folder-preview` | `motion` | 7 | ships assets |
| **Hover Image** | `hover-img` | `gsap` | 2 | ships .css, `/cdn/` demo media |
| **Masonry Grid** | `masonry-grid` | `motion` | 1 | — |
| **Pixelated Carousel** | `pixelated-carousel` | `motion` | 1 | — |
| **Apple Spotlight** | `apple-spotlight` | `lucide-react`, `motion` | 2 | — |
| **Circle Menu** | `circle-menu` | `lucide-react`, `motion` | 2 | — |
| **Magnet Tabs** | `magnet-tabs` | `motion` | 1 | — |
| **Split Showcase** | `split-showcase` | `motion` | 2 | — |
| **Trading Card** | `trading-card` | `motion` | 1 | — |
| **Jelly Loader** | `jelly-loader` | `motion` | 1 | — |
| **OTP Input** | `otp-input` | `lucide-react`, `motion` | 2 | — |

- **Arrow Fill Button** — A rounded button with an expanding color fill and a sliding arrow on hover or keyboard focus.
- **Folder Preview** — An interactive 3D folder that opens to reveal image contents.
- **Hover Image** — A stunning hover-based image preview component. When users hover over project titles, a smooth mouse-following thumbnail appears showcasing the corresponding image. Perfect for portfolios, project…
- **Masonry Grid** — A responsive masonry-style image grid with hover animations and lightbox support.
- **Pixelated Carousel** — An image carousel with a unique pixelated transition effect. Images dissolve into pixels and reassemble for the next slide.
- **Apple Spotlight** — A macOS-style spotlight search component with animated shortcuts and search results. Features a beautiful blur effect, animated shortcut buttons, and live search results.
- **Circle Menu** — A radial navigation menu that expands in a circular pattern with smooth spring animations and hover effects.
- **Magnet Tabs** — Animated tab navigation with magnetic hover effect and smooth indicator transitions.
- **Split Showcase** — A split showcase component with two interactive partner cards separated by a dotted divider. Features outward spring shift on hover, expanding rounded corners, and smooth Apple-style transitions.
- **Trading Card** — A 3D interactive trading card with perspective transforms that respond to mouse movement. Features smooth reveal animations and a premium holographic feel.
- **Jelly Loader** — A beautiful loading animation with stacked, rotating elliptical shapes that create a mesmerizing jelly-like effect. Uses a gradient color palette from light pink to deep magenta.
- **OTP Input** — An animated 6-digit OTP verification input with success/error states and smooth animations.

## Text animation

| component | slug | deps | files | notes |
|---|---|---|---|---|
| **Flip Text** | `flip-text` | — | 2 | — |
| **Rectangular Text Reveal** | `rectangular-text-reveal` | `gsap` | 2 | — |
| **Text Fill Animation** | `text-fill-animation` | `gsap` | 3 | ships .css |
| **Text Stream** | `text-stream` | `gsap` | 3 | ships .css |
| **Draggable Marquee** | `draggable-marquee` | `gsap` | 2 | ships .css |

- **Flip Text** — An animated text component where each character flips and rotates on hover. Creates a playful, interactive typography effect.
- **Rectangular Text Reveal** — Colored rectangles sweep across each line before revealing the text underneath.
- **Text Fill Animation** — A scroll-driven color sweep brings text into focus one character at a time.
- **Text Stream** — A continuous vertical text stream changes speed and direction with your scroll.
- **Draggable Marquee** — A continuous image marquee with drag momentum and a seamless looping track.

## Scroll animation

| component | slug | deps | files | notes |
|---|---|---|---|---|
| **Flip Scroll** | `flip-scroll` | `motion` | 1 | — |
| **Flow Scroll** | `flow-scroll` | `motion` | 1 | — |
| **Glowing Scroll Indicator** | `glowing-scroll-indicator` | `motion` | 1 | — |
| **Horizontal Scroll** | `horizontal-scroll` | `motion` | 1 | — |
| **Marquee on SVG Path** | `svg-path-marquee` | `motion` | 1 | — |
| **Parallax Gallery** | `parallax-gallery` | `gsap`, `motion` | 1 | — |
| **Scroll Effect** | `scroll-effect` | `motion` | 3 | ships .css |
| **Scroll Stack** | `scroll-stack` | `gsap` | 2 | ships .css |
| **SVG Pixel Reveal** | `svg-pixel-reveal` | `gsap` | 2 | ships .css |

- **Flip Scroll** — A scroll-triggered flip animation where elements rotate and transform as you scroll. Creates a dynamic, engaging scroll experience.
- **Flow Scroll** — A smooth scroll-driven animation where elements flow and transform as you scroll through the page. Creates a cinematic experience.
- **Glowing Scroll Indicator** — An animated scroll progress indicator with glowing bars that light up as you scroll.
- **Horizontal Scroll** — A smooth horizontal scrolling section that transforms vertical scroll into horizontal movement. Perfect for portfolios, galleries, and feature showcases.
- **Marquee on SVG Path** — Images follow a looping SVG curve with drag momentum and scroll-responsive speed.
- **Parallax Gallery** — A framed gallery with rotating photographs, moving side thumbnails, and scroll snapping.
- **Scroll Effect** — A stacking card scroll effect where cards stack on top of each other as you scroll.
- **Scroll Stack** — Successive cards scale into focus and fade as the next section takes their place.
- **SVG Pixel Reveal** — An SVG pixel filter dissolves into a crisp photograph as it enters the scroll area.

## Cursor effects

| component | slug | deps | files | notes |
|---|---|---|---|---|
| **Butterfly Trail Cursor** | `butterfly-trail-cursor` | `@react-three/drei`, `@react-three/fiber`, `motion`, `three` | 2 | remote assets |
| **Colorful Cursor Aura** | `colorful-cursor-aura` | `gsap`, `motion` | 2 | — |
| **Interactive Arrows** | `interactive-arrows` | `@radix-ui/react-select`, `lucide-react`, `motion` | 9 | — |
| **Magnetic Image Trail** | `magnetic-image-trail` | — | 2 | `/cdn/` demo media |
| **Mask Cursor Effect** | `mask-cursor-effect` | `motion` | 2 | — |
| **Rope Cursor** | `rope-cursor` | `gsap`, `motion` | 2 | — |

- **Butterfly Trail Cursor** — Animated butterflies lift away from the pointer with soft wing motion and fading trails.
- **Colorful Cursor Aura** — Three colored masks follow the pointer through bold typography with staggered easing.
- **Interactive Arrows** — Six canvas arrow and line patterns respond to the pointer with rotation, spacing, and opacity.
- **Magnetic Image Trail** — A cluster of images follows your cursor with magnetic momentum and a diagonal orbit.
- **Mask Cursor Effect** — A cursor-following mask effect that reveals hidden content on hover. Creates an engaging reveal experience.
- **Rope Cursor** — A smooth segmented rope follows the pointer with progressively delayed motion.

## Canvas backgrounds

| component | slug | deps | files | notes |
|---|---|---|---|---|
| **Dither Canvas** | `dither-canvas` | — | 2 | `/cdn/` demo media |
| **Dotted Grid** | `dotted-grid` | — | 2 | — |

- **Dither Canvas** — A video becomes a bright blue and cyan dither texture on white, with fluid distortion that follows your pointer.
- **Dotted Grid** — An animated dot field transforms between geometric shapes and responds to your cursor with a glowing trail.

## WebGL & 3D

| component | slug | deps | files | notes |
|---|---|---|---|---|
| **Book Flip** | `book-flip` | `@react-three/drei`, `@react-three/fiber`, `maath`, `three` | 6 | `/cdn/` demo media |
| **Curved Plane** | `curved-plane` | `gsap`, `three` | 3 | `/cdn/` demo media |
| **Fractal Glass** | `fractal-glass` | `three` | 3 | remote assets |
| **Grid Lift** | `grid-lift` | — | 3 | `/cdn/` demo media |
| **Hover Slider** | `interactive-hover-slider` | `gsap`, `three` | 3 | `/cdn/` demo media |
| **Interactive Blur Reveal** | `interactive-blur-reveal` | — | 2 | `/cdn/` demo media |
| **Art Gallery** | `art-gallery` | `three` | 4 | — |

- **Book Flip** — A tactile 3D book with bending pages, realistic light, and interactive page turns.
- **Curved Plane** — An image carousel whose edges curve and stretch with your drag and scroll velocity.
- **Fractal Glass** — Glass strips refract an image with fractal distortion and pointer-driven parallax.
- **Grid Lift** — A fine grid lifts into a dimensional text or SVG mask around your pointer.
- **Hover Slider** — An editorial list reveals a curved image stack with elastic image transitions.
- **Interactive Blur Reveal** — A frosted image becomes clear beneath a fluid cursor trail, with noise distortion and subtle grain.
- **Art Gallery** — A lensed photo grid you can drag through, with barrel distortion and infinite tiled studies.

## Dependency footprint

Across the 41 documented components:

| package | used by | pulls in |
|---|---|---|
| `motion` | 22 | Framer Motion successor |
| `gsap` | 12 | own timeline engine |
| `three` | 6 | ~600KB, WebGL context |
| `lucide-react` | 4 | icon set |
| `@react-three/drei` | 2 | three helpers |
| `@react-three/fiber` | 2 | React renderer for three |
| `@radix-ui/react-select` | 1 | headless select |
| `maath` | 1 | math helpers for three |

`motion` and `gsap` are **both** present in the catalogue and they do not share a timeline.
Mixing many components can ship two animation engines — check `pitfalls.md` before combining.
