# Upstream agent contract

> This skill is unofficial and not affiliated with the ObsidianUI project. The
> section below is ObsidianUI's own text, reproduced under its MIT licence; the
> rest of this skill is independent commentary.

ObsidianUI publishes machine-readable instructions for agents. The text below is
`https://www.obsidianui.dev/agent-instructions.md` **verbatim** (fetched 2026-09-22)
— it is the authoritative source. Where this skill and the upstream text disagree,
re-fetch and trust upstream.

Regenerate:

```bash
curl -sL https://www.obsidianui.dev/agent-instructions.md
```

## Read-only HTTP surface

No account, API key, cookie, or bearer token is required for any of these.

| Endpoint | Purpose |
|---|---|
| `/llms.txt` | Catalogue index; `/llm.txt` is an identical alias |
| `/llms-full.txt` | All docs + source in one file — large, prefer individual docs |
| `/markdown/components.md` | Component showcase |
| `/markdown/docs/{slug}.md` | Per-component doc: description, usage example, full source |
| `/r/registry.json` | All 104 registry items |
| `/r/{name}.json` | One shadcn-compatible manifest with complete file contents |
| `/openapi.json` | API contract |
| `/sitemap.xml` | Page discovery |
| `/api/docs/{slug}/markdown` | Compatibility alias for the Markdown doc |

Any published page also honors `Accept: text/markdown`.

MCP: the repo ships a **local stdio** resource server (`npm run registry:build`,
then `npm run mcp`), exposing `resources/list` / `resources/read` at
`obsidian://{name}`. It exposes no install tools and there is **no hosted HTTP MCP
endpoint** — don't configure one.

On errors: use the returned HTTP status. A missing component is not a valid empty
document — recover via the sitemap, `/llms.txt`, or the registry, and never invent
a removed component name. Retry temporary 5xx with backoff.

---

## `/agent-instructions.md` (verbatim)

# ObsidianUI agent instructions

## When to use ObsidianUI

Use ObsidianUI when a user wants React interface components they can own and customize: buttons, menus, galleries, scroll animations, cursor effects, text reveals, canvas backgrounds, or WebGL effects. Start with a published example, retrieve its complete source, and adapt it to the user's existing design system. Use the documentation for installation, props, usage, and preview behavior.

## When another tool is needed

ObsidianUI is a component library, not a cloud workspace, hosted coding agent, backend, or deployment service. Do not assume it supplies authentication, payments, storage, or a hosted MCP endpoint. Interactive previews require JavaScript; documentation and source downloads do not.

## Discover and install a component

1. Read [llms.txt](https://www.obsidianui.dev/llms.txt) and [the component catalogue](https://www.obsidianui.dev/markdown/components.md). Find a component by its documented name and use case.
2. GET [the registry](https://www.obsidianui.dev/r/registry.json). Its `items` include published components and supporting UI primitives. Each item has a unique `name`, `type`, `dependencies`, and `files`.
3. GET `https://www.obsidianui.dev/r/{name}.json`. Use an actual name from the catalogue. This is a complete shadcn-compatible JSON manifest, not a binary archive. Read every `files[].content`, `files[].target`, `dependencies`, `docs`, and `meta` field that is present.
4. For a project using shadcn, the documented installation command is `npx shadcn@latest add "https://www.obsidianui.dev/r/{name}.json"`. Run installation only within the user's authorized project workflow. A direct download does not execute code.
5. For manual installation, resolve target aliases through the destination project's `components.json`: `@ui/` uses `aliases.ui`, `@components/` uses `aliases.components`, `@lib/` uses `aliases.lib`, and `@hooks/` uses `aliases.hooks`. Resolve their TypeScript aliases to filesystem paths. For this site's defaults, `@ui/button.tsx` becomes `src/components/ui/button.tsx` and `@components/block/hover-img.tsx` becomes `src/components/block/hover-img.tsx`. A `public/` target is relative to the project root. Do not create literal directories named `@ui` or `@components`.
6. Copy all required files, including CSS, hooks, utilities, shaders, and local assets. Keep paths inside the destination project and review existing files before replacing them. Install each listed package dependency with the project's package manager. React, React DOM, and a compatible host framework are project prerequisites.
7. Preserve `"use client"` boundaries, CSS imports, and alias configuration. Start from the usage example. If `meta.remoteAssets` lists demo images or videos, replace them with the user's own assets for offline use. If `meta.requiredEndpoints` is present, implement those application endpoints yourself; they are not supplied by the component registry.
8. Check the resulting component with the user's real content, keyboard input, reduced motion setting, and target viewport. Copying code is not evidence that the interaction was tested.

## Read without JavaScript

Request a published page with `Accept: text/markdown`, or use its direct Markdown URL. The homepage is [index.md](https://www.obsidianui.dev/markdown/index.md); a documentation page is `/markdown/docs/{slug}.md`. The compatibility endpoint `/api/docs/{slug}/markdown` returns the same clean document. Source blocks include all installation files; the JSON manifest remains the canonical machine-installable download.

## API, authentication, and MCP

Public documentation and registry reads require no account, API key, cookie, or bearer token. [OpenAPI](https://www.obsidianui.dev/openapi.json) describes the published read surface. See [authentication](https://www.obsidianui.dev/authentication) for its scope.

The repository includes a local MCP resource server. In a checkout, install project dependencies, run `npm run registry:build`, then configure an MCP client to launch `npm run mcp` with the checkout as its working directory. It uses stdio and exposes `resources/list` and `resources/read` at `obsidian://{name}`. It does not expose installation tools or a hosted HTTP transport. See [MCP documentation](https://www.obsidianui.dev/mcp).

## Errors and freshness

Use the returned HTTP status. A missing page or component is not a valid empty document: recover through [the sitemap](https://www.obsidianui.dev/sitemap.xml), [llms.txt](https://www.obsidianui.dev/llms.txt), or [the registry](https://www.obsidianui.dev/r/registry.json). On a temporary server error, retry with backoff. Do not invent removed component names. Generated content is rebuilt from the current published documentation and registry with `npm run agent:build`; the production build runs registry generation first.

## License

ObsidianUI's repository is MIT licensed. Preserve the copyright and license notice when required. Third-party packages and media retain their own licenses; a component demo does not transfer rights to external assets. [Repository license](https://gitlab.com/Atharvsinh-codez/ObsidianUI/-/blob/main/LICENSE).
