# Roku — AAFES Channel Dashboard

Static dashboard for the Roku / Amazon Fire TV / Google streaming and TV-OS position in the
AAFES exchange, built by Military Sales & Service. Single self-contained page, no build step.

Prepared for **Harris Atran**, Roku (hatran@roku.com). MSS point of contact for the Roku account
is **Jon Barber**, VP of Marketing (jon.barber@mssco.com). Both are shown in the page footer.

The page renders **dark by default**; a Light / Dark toggle sits in the tab bar and the choice is
remembered per browser in `localStorage` under `roku-dash-theme`.

## Contents

| Path | What it is |
|---|---|
| `index.html` | The dashboard. All CSS, JS and chart SVG are inline; the only external requests are Google Fonts. |
| `_headers` | Cloudflare Pages response headers — `noindex`, nosniff, frame and referrer policy. |

## Data

| | |
|---|---|
| **Source** | AAFES Atlas Inventory Trend Report (ITBIS), Dept 1097 "TV Video", all stores, all suppliers |
| **Extract** | `Dept 1097 TV - Sept 1, 2026.xlsx` — 59,841 item × location rows × 106 fiscal weeks |
| **Metrics** | Units, Sales $, EoP Store On Hand Units, EoP Store Extended Retail $ |
| **Current period** | Rolling 52 weeks, W/E 2025-09-06 → 2026-08-29 |
| **Comparison** | W/E 2024-09-07 → 2025-08-30 |
| **On-hand as of** | 2026-08-29 |

Figures are AAFES only. NEX, MCX, CGX and VCS are not in this extract.

### Treatments worth knowing

- **On-hand excludes drop-ship and returns pseudo-locations.** Raw on-hand nets negative because
  `ECOMM DROP SHIP FULF` carries a −6,251 unit balance for vendor drop-ship, where AAFES owns no
  inventory. The feed reports *store* on-hand only — DC and ecommerce inventory is not visible.
- **TV operating system is only assigned where AAFES states it in the item description.** TCL,
  Hisense and Element sets whose descriptions name no platform are reported as unknown rather than
  assigned one. Roku TV and Fire TV figures are firm; the Google TV figure is a floor.
- **Screen size** is parsed from the item description and, where absent, from the model number
  (Samsung UN/QN/MRN, LG OLED/UA/UT, Sony XR/KD, Hisense and TCL series codes). 92.5% of
  department dollars resolve to a TV set with a parsed size.
- **Region** is derived from the AAFES district code — four-digit districts beginning 5, 6, 7 or 9
  are OCONUS; districts described as ECOM are online. Hawaii and Alaska count OCONUS.

### Verification

Department totals, unit totals, ecommerce dollars and all three brand totals were recomputed
directly from the source workbook by an independent script and reconcile exactly.

## Deploying

No build step — this is a plain static page.

This is served by a **Cloudflare Worker** with static assets, named `roku` — *not* a Pages
project, and **not** connected to Git. Pushing to GitHub does not deploy it. Deploy explicitly,
from a directory holding just `index.html` and `_headers`:

```
npx wrangler deploy --name roku --assets <dir> --compatibility-date 2026-09-03
```

`npx wrangler deployments list --name roku` shows what is live; `npx wrangler rollback --name roku`
reverts. Do **not** use `wrangler versions upload` to stage a preview: a preview hostname is not
covered by the Access application below, and this page carries non-public AAFES data.

## The PowerZone tab

The **PowerZone** tab is shared with the Apple Command Center and is deliberately identical on
both — the same nine AAFES departments, the same fixed department colours, the same footnote. It
is **not maintained in this repo**. The component, the weekly payload and the injector live in
the `aafes-weekly-recap` repo under `PowerZone Tab Handoff`, and everything between the
`PZ:BEGIN` / `PZ:END` markers in `index.html` is generated. Edit the component, then re-run:

```
python "PowerZone Tab Handoff/inject_powerzone_tab.py"
```

It is idempotent, so a second run is a no-op; `--check` reports without writing.

Those figures are AAFES **chain-wide** across every door, not store 1010 — the source workbook's
Overview sheet records `Stores: All`. The footnote on the tab says so; do not reword it without
checking that week's Overview sheet.

## Access

**Applied.** A Cloudflare Zero Trust Access application, `roku - Cloudflare Workers`, has gated
this Worker since 1 September 2026, with two allow policies:

| Policy | Include |
|---|---|
| Email domain: roku.com | `@roku.com` |
| Email domain: mssco.com | `@mssco.com` |

Session 24h. Verify at any time — an unauthenticated request must answer 302 to
`mss-hub-pages.cloudflareaccess.com`, never 200:

```
curl -s -o /dev/null -w "%{http_code} %{redirect_url}\n" -I https://roku.jon-barber.workers.dev
```

Confirmed 302 on 8 September 2026, after the PowerZone deployment.

---

Prepared by Military Sales & Service. Not for distribution outside Roku and MSS.
