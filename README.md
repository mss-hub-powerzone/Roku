# Roku — AAFES Channel Dashboard

Static dashboard for the Roku / Amazon Fire TV / Google streaming and TV-OS position in the
AAFES exchange, built by Military Sales & Service. Single self-contained page, no build step.

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

## Deploying to Cloudflare Pages

No build command and no build output directory — this is a plain static site.

```
Framework preset:        None
Build command:           (leave empty)
Build output directory:  /
```

Connect the repository in the Cloudflare dashboard (Workers & Pages → Create → Pages → Connect to
Git) and it will publish on every push to the default branch.

## Restricting access (not yet applied)

The dashboard carries non-public AAFES POS and inventory data. Before sharing the URL, put a
Cloudflare Zero Trust Access application in front of the Pages project:

1. Zero Trust → Access → Applications → **Add an application** → Self-hosted
2. Application domain: the Pages project hostname (and any custom domain)
3. Add a policy: **Action** Allow, **Include** → *Emails ending in* → `@roku.com`
4. Add a second Include rule in the same policy: *Emails ending in* → `@mssco.com`
5. Identity provider: One-time PIN is sufficient if neither domain is federated to your Zero Trust
   instance; otherwise use the configured IdP.
6. Save, then confirm in an incognito window that an unlisted address is refused.

Until that policy exists, treat the deployment URL as sensitive and do not circulate it.

---

Prepared by Military Sales & Service. Not for distribution outside Roku and MSS.
