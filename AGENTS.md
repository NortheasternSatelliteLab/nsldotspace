## Project layout

Astro site with MDX pages, Tailwind v4, deployed to Cloudflare Workers. `README.md` has the full guide for
content updates; the essentials:

- One file per URL in `src/pages/`. Most are `.mdx` using `layouts/PageLayout.astro`, with frontmatter
  `title`, `description`, `heading`, `lead`. `index.astro` and `contact.astro` are plain Astro.
- Every component is re-exported from `src/components/index.ts`, so pages use one import line.
  Each component's props are documented in a comment at the top of its file.
- Content that lives outside the page text: nav, contact email, social and join links in
  `src/components/site/links.ts`; team blurbs in `src/components/site/teams.ts`; home-page project
  statuses (`work`) and press list (`press`) at the top of `src/pages/index.astro`; shared text in `src/partials/`.
- Colors are a fixed palette in `src/styles/global.css` (`ink`, `prussian`, `dusk`, `slate`, `clay`,
  `alice`, `white`, `line`). Default Tailwind colors don't exist, so don't use classes like `text-red-500`.
- In MDX, leave a blank line between an opening HTML/component tag and Markdown inside it, or the
  Markdown won't be parsed.
- The contact and mailing-list forms have no backend. They open a `mailto:` link.
- Run `pnpm build` to check changes. There is no test suite.
- When people, schedule, or project details change, check for duplicates across `index.astro`,
  `this-semester.mdx`, `people.mdx`, and `projects.mdx` and update them together.

## Development

When starting the dev server, use background mode:

```
astro dev --background
```

Manage the background server with `astro dev stop`, `astro dev status`, and `astro dev logs`.

## Images

- Raster images (jpg/png/webp): use `<Image>` / `<Picture>` from `astro:assets`.
- SVGs (logos, wordmarks, illustrations): use them as SVG components
  (`import Logo from '../assets/brand/nsl-logo-light.svg'` → `<Logo class="h-16 w-auto" />`,
  adding `aria-hidden="true"` when decorative or `role="img" aria-label="…"` otherwise).
  Do not pass SVGs to `<Image>` — with `imageService: 'compile'`, the Cloudflare adapter's dev
  image endpoint rejects SVG with `400 Unsupported format: svg`, so they break under `astro dev`
  even though builds work.

## Documentation

Full documentation: https://docs.astro.build

Consult these guides before working on related tasks:

- [Adding pages, dynamic routes, or middleware](https://docs.astro.build/en/guides/routing/)
- [Working with Astro components](https://docs.astro.build/en/basics/astro-components/)
- [Using React, Vue, Svelte, or other framework components](https://docs.astro.build/en/guides/framework-components/)
- [Adding or managing content](https://docs.astro.build/en/guides/content-collections/)
- [Adding styles or using Tailwind](https://docs.astro.build/en/guides/styling/)
- [Supporting multiple languages](https://docs.astro.build/en/guides/internationalization/)
