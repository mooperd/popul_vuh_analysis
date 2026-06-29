# Popol Vuh Knowledge Graph — Schema & Authoring Contract

This document is the **contract** every graph-building iteration must obey. Each
chapter graph is produced by a *cold* agent that sees only: this file, the raw
text of one chapter, and every graph produced for earlier chapters. The graphs
are both the deliverable and the only memory carried between iterations.

The goal is a **teaching graph** of the Popol Vuh: nodes are teachable units,
edges are the links between them, and every node and edge carries a short
explanation so a learner can read the graph and understand the text.

---

## 1. Files & naming

- Source text lives in `chapters/`. Output graphs live in `graph/`.
- One graph file per chapter. The output filename mirrors the source filename
  with a `.json` extension:
  - `chapters/01_Preamble.txt`   → `graph/01_Preamble.json`
  - `chapters/Part1_Ch01.txt`    → `graph/Part1_Ch01.json`
  - `chapters/Part4_Ch12.txt`    → `graph/Part4_Ch12.json`
- An iteration writes **exactly one** new file and **never edits** an existing
  graph file. Prior graphs are read-only ground truth.

Processing order (strict reading order — "prior graphs" is always well-defined):

```
01_Preamble
Part1_Ch01 … Part1_Ch09
Part2_Ch01 … Part2_Ch14
Part3_Ch01 … Part3_Ch10
Part4_Ch01 … Part4_Ch12
```

---

## 2. File structure

Each file is a single JSON object:

```json
{
  "chapter": "Part1_Ch01",
  "chapter_file": "chapters/Part1_Ch01.txt",
  "title": "Short human title for this chapter",
  "summary": "2-4 sentence plain-language summary of what happens in this chapter and why it matters to the whole work.",
  "nodes": [ /* node objects — see §3 */ ],
  "edges": [ /* edge objects — see §4 */ ]
}
```

| key            | required | notes                                                        |
| -------------- | -------- | ------------------------------------------------------------ |
| `chapter`      | yes      | id of this chapter, matching the filename stem               |
| `chapter_file` | yes      | path to the source text                                      |
| `title`        | yes      | concise descriptive title (not just "Chapter 1")             |
| `summary`      | yes      | plain-language teaching summary                              |
| `nodes`        | yes      | nodes **introduced** in this chapter (may be empty)          |
| `edges`        | yes      | relationships **established** in this chapter (may be empty) |

---

## 3. Nodes — "things in the text"

A node is a teachable unit. A chapter file lists **only the nodes it
introduces** for the first time in the corpus. Nodes defined in an earlier
chapter are simply referenced by `id` in edges — never redefined.

```json
{
  "id": "hunahpu",
  "label": "Hunahpú",
  "type": "hero",
  "chapter": "Part2_Ch01",
  "description": "One of the Hero Twins, son of Hun-Hunahpú and Xquic. With his twin Xbalanqué he avenges his father and defeats the lords of Xibalba; their triumph prefigures the creation of humankind from maize."
}
```

| field         | required | notes                                                            |
| ------------- | -------- | ---------------------------------------------------------------- |
| `id`          | yes      | **corpus-wide unique**, stable, lowercase-kebab (see §5)         |
| `label`       | yes      | display name as it best appears in the text                      |
| `type`        | yes      | one of the controlled types below                                |
| `chapter`     | yes      | chapter where the node is **first introduced** (provenance)      |
| `description` | yes      | 1-3 sentences of teaching content explaining the node            |

### Node `type` vocabulary (controlled)

| type             | use for                                                            |
| ---------------- | ----------------------------------------------------------------- |
| `deity`          | gods and divine creators — Heart of Sky, Tepeu, Gucumatz, Huracán |
| `hero`           | protagonist characters — Hunahpú, Xbalanqué, Hun-Hunahpú, Xquic   |
| `character`      | other named persons/beings who are not divine creators or heroes  |
| `creature`       | animals, monsters, antagonist beings — Seven Macaw, Zipacná       |
| `place`          | locations and realms — Xibalba, the ballcourt, the four corners   |
| `event`          | episodes/actions — the failed creations, the flood, the trials    |
| `object`         | symbolic things — maize, the blowgun, the calabash, the ballgame  |
| `theme`          | abstract recurring ideas — sacrifice-and-rebirth, naming-as-creation |
| `lineage`        | peoples / ancestral groups — the Quiché progenitors, the first men |

If something genuinely fits none of these, choose the closest and note the
ambiguity in the `description`. Do **not** invent new type values.

---

## 4. Edges — the links between things

A chapter file lists the relationships **established in that chapter**. An edge
may connect two new nodes, a new node and a prior node, or two prior nodes (a
relationship the text only now reveals). Edges may freely reference node `id`s
defined in earlier chapters.

```json
{
  "source": "hunahpu",
  "target": "xibalba",
  "relation": "defeats",
  "description": "Hunahpú and Xbalanqué overcome the lords of Xibalba by enduring and outwitting their trials.",
  "directed": true,
  "weight": 5,
  "evidence": "they were not defeated by all the tests of Xibalba"
}
```

| field         | required | notes                                                                |
| ------------- | -------- | -------------------------------------------------------------------- |
| `source`      | yes      | node `id`                                                            |
| `target`      | yes      | node `id`                                                            |
| `relation`    | yes      | one of the controlled relations below                                |
| `description` | yes      | 1-2 sentences explaining *why* the link exists (the teaching payload) |
| `directed`    | no       | default `true`. Set `false` for symmetric relations (see below)      |
| `weight`      | no       | integer 1-5, relative importance/centrality of the link             |
| `evidence`    | no       | short quote or paraphrase from the chapter grounding the claim       |

### Edge `relation` vocabulary (controlled)

**Structural**
| relation          | meaning                                            |
| ----------------- | -------------------------------------------------- |
| `creates`         | source brings target into being                    |
| `destroys`        | source ends/annihilates target                     |
| `transforms into` | source becomes target                              |
| `is parent of`    | kinship, parent → child                            |
| `is sibling of`   | kinship, symmetric (set `directed: false`)         |
| `is instance of`  | source is a specific example of target (often a theme) |
| `located in`      | source is situated within place target             |
| `part of`         | source is a component of target                    |

**Narrative / causal**
| relation        | meaning                                          |
| --------------- | ------------------------------------------------ |
| `leads to`      | source causes/precedes target                    |
| `defeats`       | source overcomes target                          |
| `deceives`      | source tricks target                             |
| `avenges`       | source takes vengeance for target                |
| `sacrifices`    | source offers up / kills target (incl. self)     |
| `resurrects as` | source returns to life as target                 |
| `seeks`         | source pursues/desires target                    |
| `opposes`       | source is antagonist to target                   |

**Thematic / cross-chapter**
| relation         | meaning                                                  |
| ---------------- | -------------------------------------------------------- |
| `prefigures`     | source foreshadows a later target                        |
| `pays off in`    | source is the later fulfillment of an earlier target     |
| `parallels`      | source mirrors target (set `directed: false` if mutual)  |
| `inverts`        | source is a structural reversal of target                |
| `exemplifies`    | source is a concrete enactment of theme target           |

Do **not** invent new relation values. If no relation fits, choose the closest
and make the nuance explicit in `description`.

### Directionality & "flow" (Sankey)

- Default edges are **directed** (`directed: true`).
- Use `directed: false` only for genuinely **symmetric** relations
  (`is sibling of`, mutual `parallels`). Symmetry — not magnitude — is what
  bidirectional means here.
- For **flow/quantity** relationships (e.g. the substance of creation moving
  mud → wood → maize, or vengeance flowing across generations), model them as
  **directed edges carrying `weight`**, not as a single symmetric edge. The same
  data then renders as a force graph or aggregates into a Sankey diagram.

---

## 5. Continuity rules (how 46 files become one graph)

These rules are what make the cold-iteration design work:

1. **Global, stable ids.** `id` is unique across the entire corpus, not per
   chapter. Lowercase kebab-case, derived from the canonical name
   (`heart-of-sky`, `gucumatz`, `seven-macaw`, `sacrifice-and-rebirth`).
2. **Reuse, don't duplicate.** Before minting a node, search every prior graph.
   If the entity already exists, **reuse its id** — never create a second node
   for the same thing under a different id or spelling.
3. **Introduce once, reference forever.** A node appears in `nodes` only in the
   chapter that first introduces it. Later chapters reference it by id in edges.
4. **Provenance is preserved.** A node's `chapter` is where it was first
   introduced and never changes. An edge lives in the chapter where its
   relationship is established.
5. **Prior graphs are read-only.** An iteration only appends its own file.
6. **Surface long-range arcs.** When a chapter fulfills, mirrors, or reverses
   something earlier, add an explicit cross-chapter edge (`prefigures`,
   `pays off in`, `parallels`, `inverts`) against the earlier node id. This is
   what turns 46 chapter graphs into one teachable whole.

---

## 6. Authoring checklist (per iteration)

1. Read this schema. Read **all** prior `graph/*.json`; build a mental registry
   of existing ids, labels, and types.
2. Read the current chapter text.
3. Identify nodes. For each, decide: already in the corpus (reuse id) or new
   (mint id per §5)? Add only the **new** ones to `nodes`.
4. Identify relationships established in this chapter → `edges`. Include
   cross-chapter edges to prior nodes where the text earns them.
5. Add `weight`/`evidence`/`directed` where they add teaching value.
6. Validate: every edge `source`/`target` resolves to a node defined either in
   this file or in a prior graph; no duplicate ids; all `type`/`relation`
   values are from the controlled vocabularies.
7. Write exactly one file: `graph/<ThisChapter>.json`.
