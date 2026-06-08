# DA CLI Command Touch Scorecard

Date: June 7, 2026

Target repo:

```text
somarc/da-loops-tests@main
```

This scorecard records the first broad command-family touch pass against the DA loop lab. The intent is to measure whether each `da-cli` command family is reachable and useful as a loop primitive.

Scoring:

| Score | Meaning |
|---:|---|
| 0 | Not reachable or not useful in this lab yet |
| 1 | Reachable, but weak or mostly help-only |
| 2 | Useful primitive with some loop gaps |
| 3 | Loop-ready primitive |

## Summary

| Family | Touch Command | Exit | Score | Notes |
|---|---|---:|---:|---|
| `auth` | `da auth status --format json` | 0 | 2 | Useful auth preflight; human output ignored `--format json` in this pass. |
| `config` | `da config show --format json` | 0 | 2 | Resolved project `.da.json`; human output ignored `--format json`. |
| `status` | `da status --format json` | 0 | 3 | Strong agent envelope with `ok`, data, warnings, and `next`. |
| `resolve` | `da resolve <preview-url> --format json` | 0 | 3 | Strong agent envelope and executable `next` commands. |
| `content` | `da content list / --format json` | 0 | 3 | DA source list worked and returned hydrated fixture docs. |
| `preview` | `da preview status /contentbus.html --format json` | 0 | 3 | Read-only freshness proof for preview/live state. |
| `publish` | `da publish page /contentbus.html --format json` | 0 | 2 | Dry-run preflight is clear; no read-only `publish status` exists. |
| `deploy` | `da deploy page /contentbus.html --format json` | 0 | 2 | Dry-run preflight is clear, but deploy is broad and should be looped carefully. |
| `route` | `da route classify /contentbus --format json` | 0 | 3 | Correctly classified `contentbus`; `/missing` classified `orphan` with exit `2`. |
| `index` | `da index show --format json` | 0 | 2 | Needed `helix-query.yaml` fixture; now reachable for local index definitions. |
| `audit` | `da audit full /contentbus.html --format json` | 0 | 2 | Useful read-only quality pass; output is an array, not an agent envelope. |
| `block` | `da block inspect form --format json` | 0 | 2 | Built-in contracts are reachable; lab-authored `hero` has no built-in contract. |
| `migrate` | `da migrate import <lab-url> --format json` | 0 | 2 | Dry-run works; output is verbose and conversion creates nested `<main>` on an EDS source URL. |
| `pipeline` | `da pipeline scaffold release-candidate --format json` | 0 | 2 | Scaffold reachable; output is YAML despite `--format json`. |
| `code` | `da code verify /blocks/hero/hero.js --contains daLoopLab --format json` | 0 | 3 | Strong browser-facing public asset proof. |
| `design` | `da design audit /contentbus.html --format json` | 0 | 2 | Clean read-only result; output shape is useful but not envelope-based. |
| `commerce` | `da commerce --help` | 0 | 1 | Reachable, but not materially exercised by this non-commerce lab. |
| `job` | `da job list --format json` | 0 | 2 | Durable job surface reachable; no jobs exist yet in the lab. |
| `stardust` | `da stardust version --format json` | 0 | 1 | Reachable; local skill not installed and remote unavailable in this pass. |
| `site` | `da site freshness --fail-on preview-stale,live-stale,preview-missing,source-missing,probe-failed,unknown --format json` | 0 | 3 | Release-gate primitive passed with 6 fresh rows. |
| `skills` | `da skills list --format json` | 0 | 1 | Reachable, but output is human table despite `--format json`; usefulness still unclear. |
| `up` | `da up --help` | 0 | 1 | Help touched only; should be scored with an actual local server probe later. |

## Objective Results

Hydrated source routes:

- `/index.html`
- `/contentbus.html`
- `/blocks.html`
- `/hybrid.html`
- `/nav.html`
- `/footer.html`

Public reachability after hydration:

| Route | Preview | Live |
|---|---:|---:|
| `/` | 200 | 200 |
| `/contentbus` | 200 | 200 |
| `/blocks` | 200 | 200 |
| `/hybrid` | 200 | 200 |
| `/nav` | 200 | 200 |
| `/footer` | 200 | 200 |
| `/missing` | 404 | 404 |

Freshness gate:

```bash
da site freshness --fail-on preview-stale,live-stale,preview-missing,source-missing,probe-failed,unknown --format json
```

Result:

- 6 rows.
- All verdicts were `fresh`.

Codebus proof:

```bash
da code verify /blocks/hero/hero.js --contains daLoopLab --format json
da code verify /styles/styles.css --contains body --format json
```

Result:

- Both passed.

Route classification:

```bash
da route classify /contentbus --format json
da route classify /missing --format json
```

Result:

- `/contentbus`: `contentbus`
- `/missing`: `orphan`

## Findings

1. `publish` does not have a read-only `status` subcommand.
   - That is not necessarily wrong, but the loop pattern should use `preview status`, `site freshness`, or public live probes for publish verification.

2. `index` needed a lab fixture.
   - Added `helix-query.yaml` so `da index show` can be exercised.
   - A later pass should make `/query-index.json` reachable and test `index validate` plus `index query`.

3. `block inspect` is contract-name based, not page-path based.
   - `da block inspect hero` failed because `hero` is not a built-in contract.
   - `da block inspect form` passed.
   - Loop scoring for authored blocks should use `audit contracts --verify-code`, not only `block inspect`.

4. `audit full` is useful but not envelope-shaped.
   - Good read-only primitive, but agents need to infer pass/fail from row severities.

5. `pipeline scaffold release-candidate --format json` emits YAML.
   - This is probably acceptable for a scaffold, but it means `--format json` is not universal.

6. `skills list --format json` emits a human table.
   - This confirms the earlier suspicion that the skills surface is less loop-ready.

7. `migrate import` works as a dry-run, but importing an existing EDS page produced nested `<main>` markup.
   - That is a useful warning: migration loops should be tested against external source pages, not already-rendered EDS pages.

8. `commerce`, `stardust`, and `up` remain under-scored.
   - They are reachable, but this lab does not yet prove their operational value.

## Next Scorecard Pass

The next pass should add:

- A reachable `/query-index.json` target for `index validate` and `index query`.
- A form or video-hero fixture so `block inspect` can be paired with authored content.
- A pipeline YAML file that runs a safe DA-only site-readiness loop.
- A hidden trusted `--riverboat-gambler` YAML example that runs `git status`, `npm run lint`, `da site freshness`, and `da code verify`.
- An actual `da up` server probe against local and preview-fallback routes.
- A commerce fixture only if commerce remains in scope for loop-runtime scoring.
