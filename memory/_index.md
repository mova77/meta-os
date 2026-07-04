---
type: index
tags: [os, memory, layer2]
---
# Layer 2 — Memory (skeleton)

The knowledge-base convention. **Instances own the live memory**; this skeleton documents
the flow, borrowed from Karpathy's LLM-wiki:

```
raw  →  wiki  →  output
capture   refine    deliver
```

| Stage | Rule |
|-------|------|
| **raw/** | Cheap, disposable captures & agent outputs. Never cited as ground truth. |
| **wiki/** | Refined, linked, indexed reference — only **promoted** notes. The durable knowledge. |
| **output/** | Finished deliverables, each linking back to the wiki notes it drew from. |

## Promotion — the one discipline that matters

Knowledge is trustworthy only after promotion: clean the raw note, add front-matter and
`[[links]]`, move it to the right `wiki/` topic folder, update that folder's `_index.md`.
See [[systems/memory-layer]].
