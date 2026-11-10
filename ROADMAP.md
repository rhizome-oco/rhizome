# Hypha - the first milestone of Rhizome

**Hypha reads real Rust code, gives every definition a fingerprint computed from its structure and
its dependencies, keeps those definitions in one shared graph, and can turn a chosen piece back into
working code.** The last part is not optional: a store that only ingests has not proven that its
representation holds enough to compile. The capstone is **graft**.

**The graph is the source of truth.** What is stored, hashed, compared and edited is resolved
structure. Text appears once on the way in (a person wrote it) and once on the way out, at a single
marked boundary where the compiler - today a black-box oracle with only a text socket - is called.

## The parts

| crate | what it is |
|---|---|
| `hypha-extract` | the only part that knows Rust exists as source: rust-analyzer's semantic layer, pinned, behind a stable `SymbolGraph` boundary |
| `hypha-graph` | the graph's types and the Merkle hashing; language-agnostic |
| `hypha-store` | content-addressed storage: one instance per machine |
| `hypha-daemon` | the long-lived process that keeps the analysis warm - the first instance of the later genome process |
| `hypha-graft` | the closure walk, the verbatim emit, the oracle call |
| `hypha-cli` | `hy`, a thin client |

Identity: `H(node) = blake3(postcard(hash form) ++ sorted references)` - local names replaced by
positions, cycles condensed and hashed as one unit. Beside the hash form each node keeps its verbatim
source span; graft emits those original bytes and reconstructs nothing. The store lives at
`~/.local/share/hypha/store`; a project holds only a `hypha.lock`, a small pointer file.

## Eight phases, and what each proves

| phase | proves |
|---|---|
| 0 | the environment is reproducible before any logic exists |
| 1 | the extraction pipeline wires together: a tiny fixture's items listed |
| 2 | references are resolved connections, not text matches |
| 3 | identity is computed from structure, and the cascade behaves |
| 4 | the graph persists as one shared store, a warm daemon in front |
| 5 | **the round trip works: a closure becomes real, running code** |
| 6 | syncing two states costs the difference, not the whole |
| 7 | where the pipeline's real-world edges are - written down honestly |

Phase 3's four checks - hashing twice gives the same result; renaming a local variable changes
nothing; renaming an item leaves its own hash; changing one line changes it and everything above it
and nothing beside it - and phase 5 in full are the two to protect above all: between them they are
the thesis, demonstrated rather than argued.

## What it leaves out, on purpose

No layers or predicates, no conflict engine, no whole-project conjuring, no confinement, no runtime,
no network transport, no editor, no code generation of its own. One entry point in, one working
thing out, against one shared store. The second milestone builds around a core already shown to
survive the trip back into working code.

## After it

Hypha developed from its own graph, proven by a fixpoint - built from the graph, it ingests and
re-emits itself to the identical hash - with files and git demoted to a projection. Then layers and
the reach-against-change engine; views and lazy materialisation; the warm base and confinement
derived from the closure; changes classified and swapped live.

The decisions behind all this, each with what was rejected, are in [`data/decisions.json`](data/decisions.json).
