# n8n `$json` mutating Array methods regression matrix

Verified locally on 2026-08-19 with official `n8nio/n8n` Docker images and SQLite defaults.

Input for every isolated branch:

```json
{"data":["Mango","Apple","Kiwi","Orange","Blueberry","Banana","Peach","Grape","Pineapple","Strawberry"]}
```

| Expression | n8n 2.34.5 | n8n 2.35.3 | Result |
| --- | --- | --- | --- |
| `$json.data.sort()` | sorted array | `null` | regression confirmed |
| `$json.data.splice(0, 2)` | `["Mango","Apple"]` | `null` | regression confirmed |
| `$json.data.fill("X", 0, 2)` | array beginning with `X, X` | `null` | regression confirmed |
| `$json.data.copyWithin(0, 2, 4)` | mutated array | `null` | regression confirmed |
| `$json.data.reverse()` | reversed array | reversed array | unaffected control |
| `[...$json.data].sort()` | sorted array | sorted array | workaround confirmed |
| `$json.data.slice().sort()` | sorted array | sorted array | workaround confirmed |

The first combined-field workflow is kept as an additional behavioral observation: on 2.34.5, multiple mutating expressions in one Edit Fields node can mutate the shared input array while assignments are evaluated, so the isolated matrix is the authoritative comparison.
