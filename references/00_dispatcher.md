# Movie Poster Context Auditor — Technical Operational Dispatcher

**Framework Author**: William Fitzpatrick (*Writer Science*)  
**Source Lecture**: [Watch this if you want to IMPROVE the way you write](https://www.youtube.com/watch?v=LAmzkEyt60E)  
**Parent Collection**: [Master Collection](../../kirby-fitzpatrick-writers-collection/SKILL.md) | [Global Help](../../../kirby-help/SKILL.md)  

---

## 1. Cognitive Foundation: The Movie Poster Invariant

### 1.1 The originating idea

A movie poster has exactly one job: put the star front and center, large enough to read from across the street. Fitzpatrick's observation — the one this dispatcher operationalizes — is that **a sentence is a movie poster**. Every sentence you write has a star (the entity the reader is supposed to be looking at) and a position where that star lands. In English, the star's default position is the **grammatical subject**: the leading slot, the first two to four words the eye hits.

From this follows the invariant that governs the whole skill:

> **Movie Poster Invariant** — across a run of consecutive sentences that share a topic, the primary subject stays in the leading position. When the star moves, the reader's camera moves with it — and if you didn't mean to move the camera, you have just cost the reader a re-orientation.

This is not a style preference. It is a statement about how a reader builds a mental model from linear input.

### 1.2 Why the leading position is load-bearing

Readers process text incrementally, one clause at a time, with no rewind button and a working-memory budget of a few items. Every sentence gets parsed into two functional slots:

```
        THE TWO SLOTS OF EVERY SENTENCE

  |<--- LEADING POSITION --->|<-------- PREDICATE -------->|
         the anchor                  the new information

   " The parser   reads config.toml  before the server binds. "
     ^^^^^^^^^^    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
      STAR          payload the reader files under that star
```

- **Leading position = the anchor.** It answers "what are we talking about *now*?" The reader uses it to decide where to attach the incoming payload — which node of the mental model gets updated.
- **Predicate = the payload.** It answers "what is new about it?" This is where the information lives; the anchor is where it gets *filed*.

The invariant is a claim about the **filing path**. If sentence 2 files its payload under the same node as sentence 1, the reader's model grows smoothly: one anchor, accumulating detail. If sentence 2 files under a *different* node, the reader must (a) notice the switch, (b) decide whether it was intended, (c) re-anchor, and (d) hold the previous node in the background in case it comes back. That is four operations of overhead on a budget of maybe four items.

### 1.3 Attention jumping: the failure mode this skill prevents

**Attention jumping** is the accumulator defect: each individual sentence is grammatical and even clear in isolation, but the reader's focus is yanked from entity to entity with no signal. The reader cannot tell whether they are following a narrative or missing a transition.

Consider a paragraph describing one actor — the parser — doing one thing.

```
                       BEFORE  (attention jumping)
  ─────────────────────────────────────────────────────────────────────

  S1  The parser reads config.toml at startup.
      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
      [ STAR: the parser ]

  S2  The raw table is validated by a schema guard.
      ^^^^^^^^^^^^^^^^
      [ STAR: the raw table ]        -> camera yanked to the DATA

  S3  A schema guard rejects any unknown key.
      ^^^^^^^^^^^^^^^
      [ STAR: a schema guard ]       -> camera yanked to a NEW ACTOR

  S4  Warnings from the guard are surfaced to the caller.
      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
      [ STAR: warnings ]             -> camera yanked to an ARTIFACT

  S5  Startup fails if the caller ignores them.
      ^^^^^^^
      [ STAR: startup ]              -> camera yanked to a CONSEQUENCE

  READER: 5 sentences, 5 posters, 5 camera positions, 0 anchors.
```

Every sentence is defensible. Together they are a slideshow of unrelated posters. The reader reconstructs the same target in their head four separate times, and each reconstruction is a chance to misread.

```
                        AFTER  (invariant held)
  ─────────────────────────────────────────────────────────────────────

  S1  The parser reads config.toml at startup.
      ^^^^^^^^^^^^
      [ STAR: the parser ]

  S2  It validates the raw table against a schema guard.
      ^^
      [ STAR: the parser  (anaphor holds the poster) ]

  S3  It drops unknown keys and records a warning.
      ^^
      [ STAR: the parser ]

  S4  It surfaces those warnings to the caller.
      ^^
      [ STAR: the parser ]

  S5  It fails startup if the caller ignores them.
      ^^
      [ STAR: the parser ]

  READER: 5 sentences, 1 poster, 5 payloads filed under one node.
```

Nothing about the *facts* changed. `the parser` is now the grammatical subject of all five sentences, so the reader's anchor never moves and the payload accumulates. Note what the repair required: promoting the actor out of a `by`-phrase (S2), out of a nominalization (S3: *a schema guard rejects* → *it drops ... and records*), and out of a backgrounded role (S5: *startup fails* → *it fails startup*).

> **The poster pass, stated as a sentence:** read only the leading positions down the page. If that left-edge column reads like a coherent story, the invariant holds. If it reads like a list of nouns that have never met, you have attention jumping.

### 1.4 Why this matters disproportionately in software engineering

Technical prose is read like a debugger, not like a novel: by practitioners who jump to a section, read three sentences, form a hypothesis, and act. That reading mode makes anchor continuity more valuable, not less, because there is no surrounding narrative to absorb a jump.

Concretely, four costs:

1. **Misattribution under time pressure.** In a code review, a reader who loses the anchor attributes the constraint to the wrong component. That is how a reviewer requests the wrong fix.
2. **Reconstruction cost in incident documentation.** During postmortem review, readers are already rebuilding a timeline from memory. Every unnecessary star change forces them to rebuild the same actor again, and the timeline fragments into "things that happened" instead of "what the system did."
3. **Tutorial dropout.** In sequential tutorials, attention jumping at step boundaries makes the reader believe they missed an instruction. They re-read, lose position, abandon the section.
4. **Retrieval and machine-readability.** Consistent subject + anaphor chains produce clean, resolvable coreference — which is exactly what makes a paragraph chunk-retrievable and quotable by both humans and automated tooling. Star-hopping paragraphs fragment into low-value, context-free claims.

### 1.5 The counter-rule: do not over-apply it

The invariant is *continuity*, not *repetition*. Three identical leading positions in a row is a poster; nine is a metronome, and the reader stops seeing it. Hold the star with **legitimate coreference** — pronouns, definite noun phrases, a tighter synonym — rather than swapping to a new entity and rather than chanting the same noun.

- Legitimate continuity: `the parser` → `it` → `it` → `the parser` (after a long interruption) → `it`.
- Illegitimate monotony: `the parser` → `the parser` → `the parser` → `the parser`.
- Illegitimate jump: `the parser` → `the config table` → `a schema guard` → `warnings` → `startup`.

Two more guardrails:

- **Apply the invariant within a paragraph or a single logical unit** — the consecutive sentences that genuinely share a topic. Paragraph boundaries are where the camera is *allowed* to move.
- **A deliberate cut is a feature.** When the topic truly changes, change the star once, cleanly, and signal it (paragraph break, heading, or a linking transition). The skill is not "never move the camera"; it is "never move the camera *by accident*." An accidental cut in the middle of a run reads as a jump; the same cut at a paragraph boundary reads as a new scene.

---

## 2. Core Transformation Protocols

### Protocol 1 — Name the star before you write the paragraph

Before drafting, write one line: `STAR: <the entity this paragraph is about>`. If you cannot name exactly one, the paragraph is two paragraphs. This single line resolves most jumps before they are written, because a sentence whose lead is not the star announces itself as a problem at draft time instead of review time.

### Protocol 2 — Run the poster pass (mechanical, 30 seconds)

1. Underline the first three to four words of every sentence in the block.
2. Read only the underlined column, top to bottom.
3. Score it: **coherent story** (invariant holds) or **noun salad** (attention jumping).

```
  the parser        |  the parser
  the config file   |  it
  a schema guard    |  it
  warnings          |  it
  startup           |  it
  ^^^^^^^^^^^^^^^      ^^^^^^^^^^
     NOUN SALAD         ONE STORY
```

The poster pass needs no understanding of the content. It is a purely positional check, which is why it survives skim-reading and works on prose you did not write.

### Protocol 3 — Draw the chain (continuity test)

Between each pair of consecutive sentences, draw an arrow **only if** the leading position is the star itself or a legitimate reference to it (pronoun, definite NP, tight hyponym).

```
  S1 --*--> S2 --*--> S3 --X            S4 (star changed)
                   ^
                   break: S3's lead is "a schema guard",
                   an actor never introduced as the star
```

Any missing arrow is either (a) an accidental jump — repair it — or (b) a legitimate scene change — move it to a paragraph boundary. The chain makes this distinction visible in one drawing.

### Protocol 4 — Convert the jump: repair moves in ranked order

Apply the cheapest move that works, in this order:

| Rank | Move | What it fixes |
|---|---|---|
| 1 | **Re-subject** — rewrite so the intended star is the grammatical subject | Any jump, when the star is present but not in lead position |
| 2 | **De-nominalize** — turn the abstract noun back into the verb and restore its actor | `validation of X by Y` → `Y validates X` |
| 3 | **De-passivize** — promote the `by`-phrase agent into the leading position | `X was read by the parser` → `the parser read X` |
| 4 | **De-expletive** — remove `it is` / `there are` and place the real star in lead | `There are three keys the parser ignores` |
| 5 | **Resolve the demonstrative** — replace orphaned `this` / `that` with the actual noun | `This causes a failure` → `This schema mismatch fails startup` |
| 6 | **Split** — if two stars are genuinely in contention, split into two paragraphs | Compound topics masquerading as one |

Moves 1–3 fix *most* engineering prose, because the dominant mechanism of attention jumping in technical writing is **the actor being demoted out of the subject slot by passive voice and nominalization** — not by carelessness but by the impression that passives sound more formal.

### Protocol 5 — Change the star deliberately, once, at a boundary

When the topic changes, do all of the following:

1. Break the paragraph (or open a heading).
2. Name the new star in the leading position of the first sentence — do not ease into it.
3. Optionally bridge with one linking sentence that keeps the *old* star in lead and points forward.

```
  ... It surfaces those warnings to the caller.        <- old star: the parser
                                                        <- PARAGRAPH BREAK
  The caller decides whether a warning is fatal.       <- new star named in lead
```

### Protocol 6 — Re-anchor after any interruption

After a code block, a table, a diagram, an aside, or a footnote-length parenthetical, the reader's anchor is stale. Re-state the star by name (not pronoun) in the leading position of the first sentence following the interruption. This is the single highest-yield fix in tutorials and step-by-step procedures, where the star is systematically washed out by alternating prose and code.

### Protocol 7 — Never re-star mid-list

In a bulleted or numbered list, the star is held constant across items — usually by eliding the subject entirely or by keeping it in an explanatory clause after the item's distinct lead. A list whose items each promote a different entity to the lead is a legitimate list of *different things*; if it is meant to describe *one thing*, it is a paragraph in disguise.

```
  BAD (star shifts per item)            GOOD (one star, parallel payloads)
  - The parser reads config.toml.       - Reads config.toml before the bind.
  - Validation happens via a guard.     - Validates the raw table against a guard.
  - Warnings go to the caller.          - Drops unknown keys, records a warning.
  - A failure aborts startup.           - Fails startup if the caller ignores it.
```

### 2.8 Transformation table: anti-patterns and clean replacements

| # | Anti-pattern | Broken poster | Clean replacement |
|---|---|---|---|
| 1 | **Passive smuggling** — actor demoted to a `by`-phrase | `The raw table is validated by a schema guard.` | `The guard validates the raw table.` (or keep the star: `It validates the raw table against a guard.`) |
| 2 | **Nominalization drift** — verb turned into a noun, actor disappears | `Validation of the table occurs prior to startup.` | `The parser validates the table before startup.` |
| 3 | **Expletive lead** — `it` / `there` occupies the star's seat | `There are three keys the parser ignores.` | `The parser ignores three keys.` |
| 4 | **Orphaned demonstrative** — `this` with no visible antecedent | `This causes startup to fail.` | `This schema mismatch fails startup.` |
| 5 | **Entity hopping** — new artifact promoted to lead for no reason | `The config file is then read, warnings are collected, and startup proceeds.` | `The parser then reads the config file, collects warnings, and proceeds with startup.` |
| 6 | **Consequence-first** — effect becomes the star instead of the actor | `A hard failure results if the caller ignores warnings.` | `The parser fails hard if the caller ignores the warnings.` |
| 7 | **Agent in a prepositional phrase** — actor buried mid-sentence | `Before the bind, config.toml is read by the parser.` | `Before the bind, the parser reads config.toml.` |
| 8 | **Star swap inside a procedure** — each step re-stars | `Step 3: The schema is applied. Step 4: Errors are raised.` | `Step 3: Validate the schema. Step 4: Raise on any mismatch.` (subject elided, held constant) |

---

## 3. Engineering Application Scenarios

### 3.1 Code Review Comments

**Star:** the code under review (the function, the type, the line, the invariant it is supposed to hold). **Never** the reviewer's chain of reasoning.

Review comments are read in the worst possible context: a diff hunk with no narrative, one comment at a time, often by someone who did not write the code. A comment that re-stars mid-paragraph reads as three unrelated complaints against three unrelated lines.

```
  BEFORE — star drifts from code to reviewer's thinking to a side case
  ────────────────────────────────────────────────────────────────────
  "I think we should avoid resolving the path here, because the loader
   is called on every request. Also, relative paths break in containers.
   There's a helper we already have for this that handles that case, and
   callers in the CLI rely on the old behavior."

  Left edge: I | the loader | relative paths | there | callers   <- NOUN SALAD

  AFTER — star is the function; each sentence files a payload under it
  ─────────────────────────────────────────────────────────────────────
  "resolvePath() runs on every request, so it should not hit the
   filesystem. It also mishandles relative paths in containers. We
   already have resolveCachedPath() for exactly this case. Renaming
   this call to that one preserves the CLI's existing behavior."

  Left edge: resolvePath() | it | we | renaming   <- one topic, one chain
```

**Audit rubric for a review thread:** every comment names its subject in the lead; the subject is the code, not the reviewer; if a comment has two independent concerns, it is two comments (Protocol 4, move 6); after a quoted diff hunk, the first sentence re-names the star rather than opening with `This` (Protocol 6).

### 3.2 Pull Request Descriptions

**Star:** the change's protagonist — the component that now behaves differently. One star for the whole description. The diff is the *evidence*, not the star.

PR descriptions are skimmed in a queue by reviewers deciding whether to engage now or later, so the left-edge column is effectively the whole description for a large fraction of readers.

```
  BEFORE
  ────────────────────────────────────────────────────────────────────
  ## Summary
  Some refactoring was done in the ingest path. Cache invalidation had
  been broken for a while, which showed up as stale reads. A new
  invalidation hook fixes it. Performance is also improved because
  batching was added at the same time. There are tests for the hook.

  Left edge: Some refactoring | cache invalidation | a new hook |
             performance | there            <- five posters, one PR

  AFTER
  ─────────────────────────────────────────────────────────────────────
  ## Summary
  The ingest worker now invalidates the read cache on every successful
  write, which fixes stale reads on the query path. It batches its
  invalidations, so the change also removes the per-write round trip.
  It is covered by tests in ingest_worker_test.go.

  Left edge: the ingest worker | it | it   <- one star, three payloads
```

**Rule for the "Testing" and "Risk" sections:** they get their *own* star (`the new hook`, `the batching change`) and their own paragraph. Do not append them to the description paragraph as trailing sentences — that is the classic tail-jump, where the last sentence of the summary silently switches to the author talking about process rather than about the system.

### 3.3 Architecture RFCs and ADRs

**Star:** the system, service, or component whose behavior the decision changes. It must survive the entire **Decision** section unchanged; the **Context** section may hold a different star (the current state), and the boundary between them is exactly one deliberate cut (Protocol 5).

This is where the invariant pays off most, because RFC readers are building an architecture model and then holding it while evaluating consequences. Every accidental re-star forces a partial rebuild of that model, and a reviewer who rebuilds the model differently than you did will reject the proposal for reasons that sound unrelated.

```
  BEFORE — Decision section re-stars four times
  ────────────────────────────────────────────────────────────────────
  ## Decision
  We will move the billing service behind an async queue. Latency
  drops as a result. The queue introduces a retry policy that must be
  configured. Exactly-once delivery is not guaranteed by the broker.
  Downstream consumers will need idempotency keys. Idempotency key
  storage will need a new table.

  Left edge: we | latency | the queue | exactly-once | downstream |
             idempotency storage        <- camera moves 6 times

  AFTER — one star (the billing service), consequences filed under it
  ────────────────────────────────────────────────────────────────────
  ## Decision
  The billing service will publish to an async queue instead of calling
  consumers directly. It will therefore see its p99 latency drop, since
  it no longer waits on consumer round trips. It must be configured
  with a retry policy, and it must attach idempotency keys to every
  event, because the broker does not guarantee exactly-once delivery.
  It requires a new table to store those keys.

  Left edge: the billing service | it | it | it | it   <- one chain

  ## Consequences
  (new paragraph, deliberate cut)
  Consumers must now tolerate duplicate events ...
```

**Also apply the invariant to the ASR (alternatives) table:** describe each rejected option with that option as the star for the full cell. A cell that drifts from option → cost → team opinion is unreadable in table form, where there is no paragraph break available to signal the change.

---

## 4. Verification Checklist

- [ ] **Star is named.** For each audited block, I can state the single primary subject in one line, and every sentence in the block contributes payload to it — or I have split the block at the point where the topic genuinely changed (Protocol 1, Protocol 4 move 6).
- [ ] **Poster pass is clean.** I read only the leading position of every consecutive sentence; the resulting column reads as a coherent story about one entity, with no unexplained entity appearing and disappearing (Protocol 2, Protocol 3).
- [ ] **No accidental cuts.** Every star change happens at a paragraph break, heading, or explicit transition — never mid-run — and the new star is named in the leading position of the first sentence after the boundary, not eased in (Protocol 5).
- [ ] **Interruptions are re-anchored.** Every code block, table, list, or long aside is followed by a sentence whose lead names the star explicitly rather than opening with `This`, `That`, `It`, or `There` (Protocol 6, Protocol 4 moves 3–5).
- [ ] **Continuity, not monotony.** The star is held via legitimate coreference (pronoun or definite NP) rather than by chanting the same noun in every lead, and no lead column is a metronome of eight identical noun phrases (Section 1.5).