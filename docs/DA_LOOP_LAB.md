# DA Loop Lab

This repository is the controlled test platform for `da-cli` loop-runtime work.

The goal is not visual polish. The goal is route coverage and repeatable CLI behavior across contentbus, codebus, hybrid, stale, missing, and verification surfaces.

## Target

```json
{
  "org": "somarc",
  "repo": "da-loops-tests",
  "branch": "main"
}
```

Preview:

```text
https://main--da-loops-tests--somarc.aem.page/
```

Live:

```text
https://main--da-loops-tests--somarc.aem.live/
```

## Hydrated Baseline

The repo is connected to AEM Code Sync and codebus assets are reachable.

Observed public baseline after fixture hydration on June 7, 2026:

| Route | Preview | Live | Meaning |
|---|---:|---:|---|
| `/` | 200 | 200 | Root contentbus route reachable |
| `/contentbus` | 200 | 200 | Explicit contentbus route reachable |
| `/blocks` | 200 | 200 | Block contract route reachable |
| `/hybrid` | 200 | 200 | Ownership edge-case route reachable |
| `/nav` | 200 | 200 | Shared nav route reachable |
| `/footer` | 200 | 200 | Shared footer route reachable |
| `/missing` | 404 | 404 | Intentional orphan/missing route |
| `/styles/styles.css` | 200 | Not checked | Codebus reachable |
| `/blocks/hero/hero.css` | 200 | Not checked | Block asset reachable |
| `/blocks/hero/hero.js` | 200 | Not checked | Block JS verification marker reachable |

Local `da status` resolves the project target from `.da.json`.

## Intended Route Matrix

| Route | Desired State | Purpose |
|---|---|---|
| `/` | DA-authored content route | Basic contentbus route for status, preview, publish, freshness |
| `/contentbus` | DA-authored content route | Explicit contentbus classification target |
| `/blocks` | DA-authored page using blocks | Block inspect and audit contracts target |
| `/hybrid` | DA-authored page with same-path repo asset experiment | Hybrid/ownership edge-case target |
| `/missing` | Not authored | Orphan/missing route target |
| `/nav` | DA shared doc | Shared document freshness target |
| `/footer` | DA shared doc | Shared document freshness target |
| `/styles/styles.css` | Repo-owned CSS | Codebus status/sync/verify target |
| `/blocks/hero/hero.css` | Repo-owned block CSS | Block code asset verification target |
| `/blocks/hero/hero.js` | Repo-owned block JS | Block code asset verification target |

## Seed Content

Seed fixtures live under `da-loop-fixtures/content/`.

Dry-run upload:

```bash
da content put-tree da-loop-fixtures/content --format json
```

Committed upload after reviewing preflight:

```bash
da --commit content put-tree da-loop-fixtures/content --yes
```

Preview after upload:

```bash
da --commit preview pages /index.html /contentbus.html /blocks.html /hybrid.html /nav.html /footer.html --yes
```

Publish after preview:

```bash
da --commit publish pages /index.html /contentbus.html /blocks.html /hybrid.html /nav.html /footer.html --yes
```

## Reachability Checks

Read-only public checks:

```bash
curl -I https://main--da-loops-tests--somarc.aem.page/
curl -I https://main--da-loops-tests--somarc.aem.live/
curl -I https://main--da-loops-tests--somarc.aem.page/styles/styles.css
curl -I https://main--da-loops-tests--somarc.aem.page/blocks/hero/hero.css
```

Read-only `da-cli` checks:

```bash
da status --format json
da route classify / --format json
da route classify /contentbus --format json
da route classify /missing --format json
da site freshness --format json
da code verify /styles/styles.css --contains "body" --format json
da code verify /blocks/hero/hero.js --contains "decorate" --format json
```

## Loop Surfaces To Score Here

Use this repo to evaluate:

- `auth`: expired/valid auth preflight behavior.
- `config`: project `.da.json` resolution.
- `resolve` and `status`: agent entrypoint shape.
- `route`: contentbus/codebus/orphan classification.
- `content`: `put-tree`, `get`, `list`, `tree`, `versions`.
- `preview` and `publish`: async mutation plus freshness proof.
- `site freshness`: release-gate verdicts.
- `code`: status, sync, wait, verify.
- `audit` and `block`: contract and block verification.
- `pipeline`: safe DA-only loops first, then trusted gambler loops.

## First Loop Candidate

Start with a small site-readiness loop:

```bash
da status --format json
da site model --format json
da site freshness --fail-on preview-stale,live-stale,preview-missing,source-missing,probe-failed,unknown --format json
da audit contracts --prefix / --verify-code --format json
da code verify /styles/styles.css --contains "body" --format json
```

This should become the first objective scorecard pass before new first-class loop implementation.
