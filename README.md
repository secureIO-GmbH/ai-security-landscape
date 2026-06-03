# AI Security Landscape

An open, community-maintained map of practical AI use cases in cybersecurity,
organised by ML / AI approach and security department.

**Live site:** <https://secureio-gmbh.github.io/ai-security-landscape/>

Each topic lives in a single Markdown file. Add one and the site picks it up
automatically. No JavaScript or build knowledge required to contribute.

`qr-code.png` in the repository root encodes the live URL above, ready to
drop into slides or print materials.

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

## Run locally

```bash
npm install
npm run dev      # http://localhost:4321
npm run build    # static output in dist/
```

Node 18+ recommended.

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md). The short version: copy
`src/content/topics/_template.md`, fill in the frontmatter, write a short
description, open a pull request. CI validates the schema for you.

## License

MIT, see [LICENSE](./LICENSE).
