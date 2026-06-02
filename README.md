# AI Security Landscape — secureIO whitelabel build

An open, community-maintained map of practical AI use cases in cybersecurity,
organised by ML / AI approach and security department.

**Live site:** <https://secureio-gmbh.github.io/ai-security-landscape/>

Each topic lives in a single Markdown file. Add one and the site picks it up
automatically. No JavaScript or build knowledge required to contribute.

## About this fork

This is the secureIO whitelabel build of the upstream community project
[Part-IO/ai-security-landscape](https://github.com/Part-IO/ai-security-landscape).
It mirrors upstream content one to one, adds the secureIO logo to the header,
and is published on secureIO's own GitHub Pages URL for conference and
marketing use.

A daily GitHub Action (`.github/workflows/sync-upstream.yml`) pulls upstream
changes into this fork automatically. Edits to topics should land in the
upstream repository.

## How it works

- **Content.** Every topic is a Markdown file in `src/content/topics/`.
- **Schema.** `src/content/config.ts` defines the required frontmatter
  fields (title, departments, ML types, maturity, factors, references,
  connections, optional hero image). The build fails if a topic violates
  the schema.
- **Rendering.** Astro builds a fully static site. No backend, no database.
- **Hosting.** GitHub Pages, deployed automatically by
  `.github/workflows/deploy.yml` on every push to `main`.
- **PR validation.** `.github/workflows/validate.yml` runs `astro check`
  and a full build on every pull request, so frontmatter errors and broken
  connection references fail before merge.
- **Whitelabel configuration.** `deploy.yml` sets `REPO_URL`,
  `BRAND_LOGO_LIGHT`, `BRAND_LOGO_DARK`, `BRAND_LOGO_HREF`, and
  `BRAND_LOGO_ALT` so the build points at this fork and renders the
  secureIO logo. Asset files live under `public/branding/`.

## QR code

`qr-code.png` in the repository root encodes the live URL above. Drop it
into slides or print materials.

## Run locally

```bash
npm install
npm run dev      # http://localhost:4321
npm run build    # static output in dist/
```

Node 18+ recommended.

## Contributing

Contributions go to the upstream repository:
<https://github.com/Part-IO/ai-security-landscape>. See its
[CONTRIBUTING.md](https://github.com/Part-IO/ai-security-landscape/blob/main/CONTRIBUTING.md)
for how to add a topic. Once merged upstream, the daily sync picks it up
here automatically.

## License

MIT, see [LICENSE](./LICENSE).
