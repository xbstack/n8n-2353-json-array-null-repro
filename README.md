# n8n 2.35.3 `$json` mutating Array methods return `null`

Minimal reproduction and version matrix for [n8n issue #36540](https://github.com/n8n-io/n8n/issues/36540).

## Confirmed behavior

Using official Docker images with isolated Edit Fields branches:

| Expression | n8n 2.34.5 | n8n 2.35.3 |
| --- | --- | --- |
| `$json.data.sort()` | array | `null` |
| `$json.data.splice(0, 2)` | removed items | `null` |
| `$json.data.fill('X', 0, 2)` | mutated array | `null` |
| `$json.data.copyWithin(0, 2, 4)` | mutated array | `null` |
| `$json.data.reverse()` | reversed array | reversed array |
| `[...$json.data].sort()` | sorted array | sorted array |
| `$json.data.slice().sort()` | sorted array | sorted array |

The 2.35.3 copy-first variants for `sort`, `splice`, `fill`, and `copyWithin` were also verified and passed.

## Environment

- `n8nio/n8n:2.34.5` — Node.js `v24.18.0`
- `n8nio/n8n:2.35.3` — Node.js `v24.18.1`
- n8n CLI execution
- default SQLite
- macOS + Docker Desktop on an arm64 host
- one independent input branch per expression to avoid cross-field mutation

## Files

- `workflow-isolated.json` — authoritative version-comparison fixture
- `workflow-workarounds.json` — copy-first workaround fixture for 2.35.3
- `workflow.json` — initial combined-field control; not used as the authoritative matrix
- `results/verification.json` — normalized machine-readable results
- `version-matrix.md` — compact human-readable results

## How to rerun

Use a disposable n8n environment for each target version, import `workflow-isolated.json`, and execute the workflow. Repeat with `workflow-workarounds.json` on 2.35.3. Keep the versions isolated so database/runtime state from one run cannot affect the other.

The n8n CLI currently expects an imported workflow ID for `execute`; the repository intentionally does not claim that the deprecated `--file` flag is a stable one-command runner.

## Workaround

Copy the input array before calling a mutating method, for example:

```text
{{ [...$json.data].sort() }}
```

The same copy-first pattern was verified for `splice`, `fill`, and `copyWithin` on 2.35.3.

The upstream reporter also observed that `N8N_EXPRESSION_ENGINE=legacy` makes the same expressions work again on 2.35.3. That engine-toggle result is upstream evidence; it is not part of the local XBSTACK verification matrix in this repository.

## Evidence boundary

This repository confirms the observable regression between the two tested versions and the tested containment. It does **not** claim a specific internal root-cause commit unless n8n maintainers confirm one.

Full reproduction notes and production guidance:
https://www.xbstack.com/en/ai/n8n-2353-json-array-methods-return-null/?utm_source=github&utm_medium=referral&utm_campaign=n8n_2353_array_null_regression&utm_content=repository_readme
