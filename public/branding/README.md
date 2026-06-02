# Branding assets for the secureIO whitelabel build

Files in this folder are served as `/branding/...` at runtime.

## Logo files

| File | Used when | env var |
|---|---|---|
| `secureio-logo-black.png` | Light theme (dark ink on light header) | `BRAND_LOGO_LIGHT` |
| `secureio-logo-white.png` | Dark theme (light ink on dark header) | `BRAND_LOGO_DARK` |

CSS swaps the two variants automatically based on the active theme. To
update either logo, replace the file with the new asset and keep the
same filename.

## Logo guidance

- **Format:** PNG with transparent background, or SVG (then also update
  the `BRAND_LOGO_*` env var to point at the `.svg` file).
- **Aspect ratio:** roughly 3:1 to 5:1 (the header renders the logo at
  28 px height; width scales proportionally).
- **Minimum height:** 112 px for crisp rendering on retina displays.
- **Maximum size:** ~400 px wide is fine.

## Where the logo links to

By default the logo links to <https://secure-io.de/>. To change, edit
`BRAND_LOGO_HREF` in `.github/workflows/deploy.yml`.
