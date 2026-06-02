# Branding assets for the secureIO whitelabel build

This folder is served as `/branding/...` at runtime.

## Required asset

| File | Used as |
|---|---|
| `secureio-logo.png` | Header logo, referenced by `BRAND_LOGO_PATH` in `.github/workflows/deploy.yml` |

## Logo guidance

- **Format:** PNG with transparent background (SVG also works if you also
  update `BRAND_LOGO_PATH` to point at the `.svg` file).
- **Aspect ratio:** roughly 3:1 to 5:1 (the header renders the logo at
  28 px height; width scales proportionally).
- **Dimensions:** at least 112 px tall for crisp rendering on retina
  displays; up to ~400 px wide is fine.
- **Contrast:** the logo sits next to the theme-toggle button on a dark
  or light header background, so a self-contained design that works on
  both themes is ideal (e.g. solid-colour or two-tone logo with enough
  contrast against both `#161a22` and `#f1f3f6`).

## Where the logo links to

By default the logo links to `https://secure-io.de/`. To change, edit
`BRAND_LOGO_HREF` in `.github/workflows/deploy.yml`.
