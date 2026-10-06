# Northeastern Satellite Lab website

Source for [nsl.space](https://nusatlab.space). It's an [Astro](https://docs.astro.build) site with MDX pages, Tailwind CSS v4, and Cloudflare Workers hosting. Nearly all content is plain text inside page files, so most updates don't need real programming.

## Running it locally

You need Node 22.12 or newer and [pnpm](https://pnpm.io).

```sh
pnpm install
pnpm dev        # http://localhost:4321, reloads as you save
pnpm build      # production build into dist/
pnpm preview    # serve the production build locally
```

Run `pnpm build` before you push. It catches broken imports and bad component props that the dev server may not.

## How the site is laid out

```text
src/
  pages/        One file per URL. This is where almost all edits happen.
  components/   Reusable building blocks (Section, Card, Position, ...).
  layouts/      Page shells: BaseLayout (head, nav, footer) and PageLayout (adds the title block).
  partials/     Chunks of text reused on more than one page.
  assets/       Images and fonts that Astro processes (photos/, missions/, brand/, fonts/).
  styles/       global.css, which holds the color palette and Tailwind setup.
public/         Files served as-is (favicon).
astro.config.mjs  Site URL, fonts, redirects.
wrangler.jsonc    Cloudflare deploy config.
```

### Pages and URLs

A file in `src/pages/` becomes a URL by its path:

| File | URL |
| :--- | :--- |
| `src/pages/index.astro` | `/` |
| `src/pages/about.mdx` | `/about` |
| `src/pages/people.mdx` | `/people` |
| `src/pages/projects.mdx` | `/projects` |
| `src/pages/projects/dewsat.mdx` | `/projects/dewsat` |
| `src/pages/this-semester.mdx` | `/this-semester` |
| `src/pages/contact.astro` | `/contact` |

Most pages are `.mdx`: Markdown with the ability to drop in components. The home page and contact page are `.astro` files because they have more logic. The old `/programs` URL redirects to `/projects` (set in `astro.config.mjs`).

Every `.mdx` page starts with frontmatter that feeds `PageLayout`:

```mdx
---
layout: ../layouts/PageLayout.astro   # relative path, one more ../ for files in a subfolder
title: People                          # browser tab and search results
description: "One sentence for search results."
heading: People                        # the big h1 on the page
lead: "The intro paragraph under the heading."
---

import { Section, Roster, Position } from '../components';
```

Every component is exported from `src/components/index.ts`, so one import line covers them all.

### Basic building blocks

`<Section title="..." sub="..." id="...">` is a titled band of the page. Everything on a page sits inside one. The `id` makes it linkable, like `/projects#core`.

`<Grid cols={2}>` holds `<Card>`s or `<ImageCard>`s. Each component has a comment at the top of its file listing its props, so check there first.

Plain Markdown (paragraphs, lists, links) works anywhere in an `.mdx` file. Wrap it in `<div class="prose-nsl text-alice/90">` if it needs the site's body-text styling. Leave a blank line between the opening tag and your Markdown, or MDX won't parse it as Markdown.

## Common updates

### New semester: the schedule

Edit `src/pages/this-semester.mdx`.

1. Update `title`, `description`, `heading` and the text inside frontmatter.
2. Change the `<Session day time where>` rows in "Weekly meetings".
3. Replace the `<Event when title>` entries under "Workshops, talks, and events".
4. Update the "New here?" paragraph if the Intro to Satellites time or room changed. The same time and room also appear in `src/pages/index.astro` in the hero text.

### New officers or leads

Edit `src/pages/people.mdx`. Each group is a `<Roster>` of `<Position role name />` entries.

- Leave off `name` (or write `name="TBD"`) for an open role.
- `deputy="..."` adds a deputy under the lead.
- `partner` marks someone from a partner lab.
- `href` links the name to a project page.
- For two people, put both in `name`: `name="Noah Typrin, Landon Bayer"`.

When a project lead changes, also update the `meta="Lead: ..."` line on that project's card in `projects.mdx`.

### Projects

The list lives in `src/pages/projects.mdx`, in three sections:

- Flagship: `<ImageCard>` with an SVG from `src/assets/missions/`. These link to their own page.
- Core and rising: plain `<Card>`s. Change `tag` to update the status ("In development", "Up next").

To add a flagship project page, copy `src/pages/projects/proves-atlas.mdx` or `dewsat.mdx` to a new file in the same folder. Each uses a `<Mission>` component, which takes `name`, `status`, a `scale` list of key numbers, and `collaborators`, with the body text inside it. Then add an `<ImageCard>` for it on the Projects page and a `href` on the lead's `<Position>` on the People page. To remove a project, delete its card and its page, and search the repo for links to its URL.

Home page status lines for the four flagship projects are in the `work` list at the top of `src/pages/index.astro`. Update them when a project hits a milestone.

### News and press

The "In the news" list is the `press` array at the top of `src/pages/index.astro`. Add an object with `title`, `outlet` (include the month and year, like `Northeastern Global News, March 2026`), and `href`. Newest goes first.

### Navigation, email, social and sign-up links

All in `src/components/site/links.ts`:

- `navLinks` is the top nav. Adding a page doesn't add it to the nav automatically.
- `contactEmail` is an object with `text` (what visitors see, written with `{at}` and `{dot}`), `user` and `domain`. It's split up on purpose so the full address never appears in the HTML for spambots to scrape. Use `<Email />` to show the address as a link, and join `user` and `domain` in a script if you need the real address. Never put the full address in an `href`. It's used in the footer, the contact page and both forms.
- `socialLinks` are the Instagram and LinkedIn links on the contact page.
- `joinLinks` are the NUEngage, Discord and interest-form links shown on This Semester. Check these every semester, since Discord invites and Google Forms get replaced.

The four team descriptions on the home page come from `src/components/site/teams.ts`. The longer versions are in `src/pages/about.mdx`.

### Text shared across pages

`src/partials/` holds MDX snippets that are imported into more than one page: `what-is-nsl.mdx`, `what-is-a-cubesat.mdx` and `alumni-employers.mdx`. Edit them once and every page that imports them updates.

### Quick facts and partner lists

Both are in `src/pages/index.astro`: the "Quick facts" numbers, and the `<Org>` entries under "Who we work with".

## Images

- Photos (jpg, png, webp): put them in `src/assets/photos/` and use `<Image>` from `astro:assets`. Astro resizes them and generates responsive sizes. Always write real `alt` text, and credit the photographer in the caption when there is one. `proves-atlas.mdx` has working examples.
- SVGs (logos, mission art, diagrams): import them as components, never through `<Image>`:

  ```mdx
  import Logo from '../assets/brand/nsl-logo-light.svg';
  <Logo class="h-16 w-auto" aria-hidden="true" />
  ```

  `<Image>` fails on SVGs under `pnpm dev` with `400 Unsupported format: svg`, even though the build works.
- Diagrams drawn in code are in `src/components/diagrams/`.

## Styling

Tailwind utility classes go straight in the markup. The palette is defined in `src/styles/global.css` and replaces Tailwind's defaults, so only these color names exist: `ink`, `prussian`, `dusk`, `slate`, `clay`, `alice`, `white`, and `line`. Something like `text-red-500` won't work. Headings use the Science Gothic font (`font-display`), body text uses Roboto, and fonts are set up in `astro.config.mjs`.

## Forms

The contact form and mailing list form have no backend. Submitting opens the visitor's email app with a pre-filled message to the contact address, which the form script assembles from `contactEmail.user` and `contactEmail.domain`. Real sign-ups need a service added later (Cloudflare Workers can handle that).

## Deploying

The site is deployed as a Cloudflare Worker named `nsldotspace` (see `wrangler.jsonc`), serving the static build from `dist/`. I didn't find a CI config in the repo, so check with the current web lead how deploys run today. Deploying by hand is `pnpm build` followed by `pnpm wrangler deploy`, which needs access to the club's Cloudflare account.

## Working on this with Claude Code

`CLAUDE.md` and `AGENTS.md` hold the project notes for AI coding tools, including the SVG rule above and how to run the dev server in the background.

## Links

- [Astro docs](https://docs.astro.build)
- [MDX in Astro](https://docs.astro.build/en/guides/integrations-guide/mdx/)
- [Tailwind CSS docs](https://tailwindcss.com/docs)
