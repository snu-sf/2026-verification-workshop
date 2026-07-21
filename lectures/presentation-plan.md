# From Cells to Memory: Protocols as Resources in CRIS

Concrete plan for a 90-minute lecture in the second-day theory session of the
Program Verification Workshop (Wednesday, 29 July 2026).

The public schedule gives the whole morning block, 10:00--12:00, to
“Separation Logic and CRIS.” This plan assumes that the first approximately 30
minutes are the preceding Cell introduction, and uses the remaining 90 minutes
for memory abstraction and richer resource algebras. If the Cell introduction
finishes earlier, the optional material near the end can expand the session to
the full two hours.

Companion artifacts:

- `presentation-slides.tex`: editable Beamer source.
- `presentation-slides.pdf`: rendered slides (generated from the source).

## 1. Purpose and teaching contract

### One-sentence thesis

A resource is not merely “a piece of memory”: it is a transferable proof-level
capability whose algebra encodes which observations, updates, sharing patterns,
and state transitions a client is allowed to rely on.

### Learning outcomes

By the end, an audience member should be able to:

1. Read the four core memory specifications and explain why `load` accepts a
   fraction while `store` and `free` require full ownership.
2. Explain the authoritative/fragment split of `ghost_map`, including why the
   authoritative map belongs in the CRIS module-local invariant (IST) and
   per-location fragments belong to clients.
3. Derive local reasoning for a store: updating location `l` preserves a framed
   resource for every other location.
4. Distinguish three client protocols: mutable exclusive ownership, fractional
   shared-read ownership, and persistent read-only knowledge.
5. Use a persistent lower-bound snapshot from `mono_nat` to prove that an
   access counter observed before an arbitrary returning function call does not
   exceed the counter observed afterward.
6. Select a plausible resource algebra from a program protocol instead of
   trying to invent the algebraic carrier and operation first.

### Audience model

Assume familiarity with modules, abstract data types, static analysis, maps,
aliasing, and operational semantics. Assume the first-day material covered
interaction trees, CRIS modules, simulation, and contextual refinement. Assume
the immediately preceding talk introduced separation, ownership, and resource
algebras through the repository's Cell example.

Do **not** assume familiarity with Hoare triples, the frame rule, Iris proof
mode, authoritative cameras, step indexing, invariants from concurrent program
logic, or Rust's formal semantics. Introduce only the small amount of notation
needed at the point where it earns its keep.

## 2. Positioning relative to the artifacts

It is important to distinguish three memory examples during the talk.

1. **The current repository example.** `imp_system/mem/MemA.v` already builds a
   custom authoritative map using `authUR`, nested discrete functions,
   `optionUR`, and `dfrac_agreeR`. It exposes points-to resources and the desired
   alloc/load/store/free rules. `MemIAproof.v` relates this logical map to the
   concrete `MemI` state through an IST.
2. **The teaching presentation's `ghost_map` version.** This is a conceptual
   refactoring of the same protocol using Iris's standard `ghost_map` API. It is
   intentionally shown because the packaged rules expose the protocol more
   clearly than the hand-rolled carrier. CRIS already carries a compatibility
   port at `CRIS/theories/iris_system/lib/ghost_map.v`; the current `MemA` simply
   does not use it. The talk must not claim that `MemA` already imports that
   library.
3. **The CRIS paper's hybrid memory example.** Section 7 and Figure 12 use
   `take` to let clients choose between an ownership specification and an
   operational memory model. That is a separate, advanced CRIS point. Keep it
   in the appendix unless there is extra time; do not conflate it with the
   `ghost_map` teaching refactor.

The transition from the earlier Cell talk should be explicit:

> The Cell resource algebra solved agreement and update for one logical slot.
> A memory is the same story at a dynamically growing set of keys. `ghost_map`
> packages exactly that recurring pattern.

The repository Cell and the paper Cell are also different: `ring/CellI.v` is a
direct `get`/`set` cell, whereas the paper's motivating Cell has a callback.
Refer to the repository Cell when making the direct Cell-to-map transition.

## 3. Conventions fixed for the talk

These decisions remove distracting ambiguity from the example.

- A location is a pair `(block, offset)` and the ghost map has conceptual type
  `gmap location val`.
- The main example calls `alloc(1)`, because the current concrete `free`
  operation deallocates one cell rather than an entire C-style block.
- The access counter counts completed calls to `alloc`, `load`, `store`, and
  `free`. The query operation itself does not increment the counter. This is a
  presentation convention; another convention preserves monotonicity but
  changes what the benchmark measures.
- The counter is an unbounded natural number in the model. A machine-integer
  implementation must rule out overflow, use saturation, or expose modular
  arithmetic; otherwise an exact relation to `mono_nat` is false.
- The benchmark theorem is `before <= after`. If the displayed difference is
  over integers, derive `0 <= (after : Z) - (before : Z)`. Do not present
  nonnegativity of `Nat.sub` as the result, since that is true merely by type.
- “Snapshot” is the pedagogical name. In the installed Iris library the actual
  proposition is `mono_nat_lb_own`, a persistent lower-bound token. There is no
  definition named `mono_nat_snap` in this checkout.
- Unlike `ghost_map` and `mono_list`, CRIS does not currently contain a local
  `iris_system/lib/mono_nat.v` compatibility wrapper. The upstream Iris
  base-logic API supplies the names and laws used in the slides, but an actual
  counted-memory implementation in CRIS should port that small wrapper to
  `CRIS.own` (following the local `mono_list.v` pattern), or define the same
  propositions directly over the upstream algebra-level `mono_natR`.
- The Rust comparison is an analogy: fractional permissions explain why many
  readers can coexist and why writing requires recombination. `ghost_map` does
  not itself model Rust lifetimes, reborrowing, or the borrow checker.
- Persisting a `ghost_map` element freezes the value that is present **at the
  time of persistence**. For a completion flag, first update `false` to `true`,
  then persist the `true` element. If clients need persistent observations
  before the transition, use a monotone/one-shot protocol instead.
- “Arbitrary `f`” means no functional specification is assumed. The property is
  conditional on `f` and the second query returning; `f` may diverge. Physical
  counter state remains protected by CRIS scopes and can only be changed
  through the module interface.

## 4. Narrative arc and timing

| Time | Slides | Segment | Audience question answered |
|---:|---:|---|---|
| 0--7 min | 1--4 | Bridge from Cell; resources as protocols | “Why go beyond the Cell algebra?” |
| 7--17 min | 5--7 | Running memory client and desired rules | “What should callers be allowed to know and do?” |
| 17--33 min | 8--12 | `ghost_map`: authority, elements, validity, IST | “How does one private heap support many local client views?” |
| 33--48 min | 13--17 | Allocate/load/store traces and local reasoning | “Why does an update at `l` preserve facts about `k`?” |
| 48--63 min | 18--24 | Fractions, shared reads, DFrac, freezing | “How can aliases read safely, and how can a fact become permanent?” |
| 63--66 min | 25 | Retrieval checkpoint | “Can I now predict which operations are legal?” |
| 66--78 min | 26--29 | Counter motivation and `mono_nat` | “How can a past observation remain useful after unknown code?” |
| 78--85 min | 30--33 | Combined IST and client proof | “Where does `before <= after` actually come from?” |
| 85--90 min | 34--36 | RA selection recipe, takeaways, afternoon bridge | “How do I use this pattern in the exercises?” |

The timings include two short audience checks and about four minutes of elastic
buffer. Take clarification questions at segment boundaries; defer questions
about camera construction, step indexing, concurrency, or CAS to the appendix.

## 5. Slide-by-slide plan

### Segment A -- from one Cell to protocols (0--7 minutes)

#### Slide 1 -- Title: “From Cells to Memory” (0:00--0:30)

- Subtitle: “Protocols as Resources in CRIS.”
- State the promise verbally: no new program logic machinery first; scale the
  Cell idea and then swap in a different protocol.
- Do not spend time on an agenda here.

#### Slide 2 -- Where we start (0:30--2:30)

- Recap only three facts from the preceding talk:
  `cell(v)` is a client capability, the module retains a companion authority,
  and validity forces agreement.
- Show the chain “one slot -> many keyed slots -> history across time.”
- Explicitly connect to Day 1: refinement gives the executable/open-system
  story; resources add ownership-based local reasoning.

#### Slide 3 -- What the audience will be able to do (2:30--4:00)

- Show three outcome verbs: **read** a memory contract, **predict** what frames,
  and **choose** a resource protocol.
- Tell the audience the algebraic implementations will appear only after the
  operational intuition.

#### Slide 4 -- Protocols already present in ordinary programs (4:00--7:00)

- Four concrete examples:
  - key-value ownership: this key currently stores this value;
  - shared immutable access: several clients may read, no client may write;
  - publication/freeze: a final result can be copied forever;
  - monotonic history: a later observation cannot be smaller.
- Ask for one additional protocol from the room (30--45 seconds). Map likely
  answers to “capability + allowed transition,” without attempting to formalize
  them yet.

### Segment B -- specify the memory clients want (7--17 minutes)

#### Slide 5 -- Running program (7:00--9:00)

Use one cell to avoid block-deallocation details:

```text
p = alloc(1);
store(p, 7);
x = load(p);
free(p);
```

Ask: what prevents a second component from storing through `p` between the
store and load? The answer should not be “we inspected all of its code.”

#### Slide 6 -- The four proof rules (9:00--14:00)

Introduce `l |-> v` as an ordinary assertion in prose: “I own permission for
location `l`, and its value is `v`.” Then show:

```text
alloc(n)    returns a fresh region, each cell |-> undef
load(l)     needs l |->{q} v; returns v and the same fraction
store(l,w)  needs full l |-> v; returns full l |-> w
free(l)     needs full l |-> v; consumes it
```

Explain pre/postconditions as resource transfer at a call boundary. Avoid a
general lecture on Hoare logic.

#### Slide 7 -- Why not expose one global heap fact? (14:00--17:00)

- A client assertion `heap = M` is globally fragile: a store anywhere changes
  it.
- A client assertion `l |-> v` is local: it says exactly which entry the client
  can rely on and authorize changing.
- Desired frame example: from `l |-> 7 * k |-> 20`, storing 8 at `l` should
  yield `l |-> 8 * k |-> 20` without reproving anything about `k`.

### Segment C -- `ghost_map` and the CRIS IST (17--33 minutes)

#### Slide 8 -- Memory is a map of Cells (17:00--19:00)

- Draw three map entries and detach each into a client token.
- Transition sentence: `ghost_map` is the standard-library version of this
  repeated Cell pattern.
- Mention, in a small note, that the checked-in `MemA.v` manually constructs an
  equivalent authoritative map; this lecture uses the packaged interface.

#### Slide 9 -- Two views of one logical map (19:00--22:00)

- Module side: `Auth_gamma(M)`, the authoritative view of the whole map.
- Client side: `l ->[gamma]{q} v`, a fragment for one entry.
- The ghost name `gamma` connects the views; it is not a runtime address.
- Authority is not “more concrete memory.” Both pieces are ghost resources.

#### Slide 10 -- The one law to remember (22:00--25:00)

Show only the useful consequence first:

```text
Auth_gamma(M) * (l ->[gamma]{q} v)  entails  M[l] = v
```

Explain validity as a compatibility check on all simultaneously owned claims.
The client cannot forge `l -> w`, because it would be incompatible with the
authority at `v`.

#### Slide 11 -- Where the authority lives in CRIS (25:00--29:00)

Display the simplified invariant:

```text
IST(st_A, st_I) := exists m M,
  st_I["Mem.mem"] = m
  * repr(m) = M
  * ghost_map_auth gamma 1 M
```

- The private implementation state and the logical map agree at every
  interaction point.
- `MemA` has no concrete memory state; clients see only specifications and
  fragments.
- The full authority remains hidden in the module proof. A method temporarily
  combines it with the caller's fragment, performs a frame-preserving update,
  restores the IST, and returns the promised fragment.

#### Slide 12 -- The packaged rules are the protocol (29:00--33:00)

Show a small table of `ghost_map_lookup`, `ghost_map_insert`,
`ghost_map_update`, and `ghost_map_delete`. Highlight that insert/update/delete
need full authority and update/delete also need a full element. Fractions are
enough for lookup.

Do not show `gmap_viewR`, `agreeR`, or camera laws on this slide. Put exact Coq
identifiers in a narrow footer so they become recognizable in the afternoon.

### Segment D -- follow the resources through calls (33--48 minutes)

#### Slide 13 -- Allocate extends the map and creates ownership (33:00--36:00)

Animate conceptually:

```text
Auth(M)
  ~~> Auth(M[l := undef]) * (l |-> undef),   l notin dom(M)
```

The concrete allocator chooses a fresh block; the proof extends the logical
map at the same location and hands the fragment to the caller.

#### Slide 14 -- Load is agreement without change (36:00--39:00)

```text
IST owns Auth(M)       caller supplies l |->{q} v
validity gives M[l]=v  concrete load therefore returns v
IST and fragment are unchanged
```

Point out that read access does not need to know or own the other keys.

#### Slide 15 -- Store changes exactly one entry (39:00--42:00)

Use two visible entries:

```text
Auth({l:7, k:20}) * l |-> 7 * k |-> 20
  ~~>
Auth({l:8, k:20}) * l |-> 8 * k |-> 20
```

Color only the `l` entry as changing. This is the visual statement of local
reasoning.

#### Slide 16 -- One store proof, end to end (42:00--45:30)

Show the five proof obligations, not tactic syntax:

1. Open the IST to obtain concrete heap `m`, logical map `M`, and `Auth(M)`.
2. Combine authority with `l |-> v` to learn that concrete lookup succeeds.
3. Execute the concrete `SGet`/`SPut` store.
4. Apply the joint logical update to obtain `Auth(M[l:=w]) * l |-> w`.
5. Re-establish `repr(m') = M[l:=w]`, close the IST, and return the fragment.

Mention `MemIAproof.v` only as the mechanized instance; do not live-scroll the
full proof.

#### Slide 17 -- Audience check: what frames? (45:30--48:00)

Give 45 seconds to decide:

```text
Alice: l |-> 7       Bob: k |-> 20       l != k
Alice calls store(l, 8). What must Bob reprove?
```

Answer: nothing. Bob's token is a compatible frame before and after the
frame-preserving update. Contrast this with two full tokens for the same key,
which are invalid.

### Segment E -- sharing and freezing with DFrac (48--63 minutes)

#### Slide 18 -- Fractions turn exclusivity into shared reads (48:00--51:00)

Show the split/recombine equation for `q1 + q2 <= 1`:

```text
l |->{q1+q2} v  <->  l |->{q1} v * l |->{q2} v
```

Both fragments agree on the value. Avoid interpreting `q` as a probability or
a physical byte fraction; it is accounting for write permission.

#### Slide 19 -- Two readers, no writer (51:00--54:00)

- Split full ownership into two halves.
- Each half satisfies `load` and is returned.
- Neither half satisfies `store` or `free`.
- Recombining all outstanding fractions recovers full ownership and therefore
  update permission.

#### Slide 20 -- Rust analogy, with a boundary (54:00--56:00)

Use a two-column comparison:

| Permission story | Rust intuition |
|---|---|
| several fractions may read | several shared `&T` borrows |
| full fraction required to write | unique mutable access |
| recombine all shares | all shared borrows have ended |

Then state the boundary aloud: the RA is the logical permission discipline, not
a model of lifetimes or compiler borrow checking.

#### Slide 21 -- Why a fraction cannot be used to store (56:00--58:00)

Make the environmental-frame argument explicit. If Alice with one half could
change `v` to `w`, Bob's still-owned half would assert the old `v`; the combined
resource would become invalid. A legal update must preserve every compatible
frame, so it cannot happen until the shares are recombined.

#### Slide 22 -- DFrac adds “discarded” knowledge (58:00--60:00)

Introduce only two public modes:

- `DfracOwn q`: quantitative, splittable permission;
- `DfracDiscarded` / square notation: persistent, duplicable read-only
  knowledge.

Show `ghost_map_elem_persist` as an irreversible publication step. Use the
wording “discard the write capability, retain duplicable knowledge,” rather
than “drop the resource.”

#### Slide 23 -- Correct one-shot completion pattern (60:00--61:30)

```text
done |-> false
   -- full update --> done |-> true
   -- persist -----> done |->[] true     (duplicable forever)
```

The final token lets any client rely on completion and prevents full ownership
from ever being recovered, so the entry is frozen.

#### Slide 24 -- The persistence pitfall (61:30--63:00)

- Persisting `done |-> false` freezes the flag at `false`; it does not encode a
  future `false -> true` transition.
- If observers must retain persistent information before the transition, use a
  monotone or one-shot state machine. A Boolean can be encoded as `0 -> 1` with
  the next example's monotone resource.
- Mention that `ghost_map_elem_unpersist` can recover an unspecified proper
  fraction but never the full fraction required to write; publication remains
  irreversible in the relevant sense.

#### Slide 25 -- Retrieval checkpoint (63:00--66:00)

Ask the room to classify each requirement with one of three tokens:

1. “May load and store” -> full owned element.
2. “Two components may load” -> split owned fractions.
3. “Anyone may copy this final fact forever” -> discarded/persistent element.

Then restate the four memory rules in one line. If the room is behind, stop here
for questions and cut Slides 34--35 later.

### Segment F -- a fact about the past with `mono_nat` (66--78 minutes)

#### Slide 26 -- The benchmark across unknown code (66:00--69:00)

```text
before = Mem.get_count();
f();                         // arbitrary, may call Mem
after  = Mem.get_count();
assert(0 <= after - before); // if f returns
```

Ask why ordinary ownership of the **current** value is awkward: after `f`, the
current value has changed, but the proof still needs a stable fact about the
old observation.

#### Slide 27 -- The weak query spec forgets history (69:00--71:00)

A result-only spec, “returns the current counter,” gives two unrelated
existentials when called twice. The module needs to return a durable logical
receipt for the first observation. Name the desired receipt `Snap_gamma(m)`.

#### Slide 28 -- `mono_nat`: current authority plus lower-bound receipts (71:00--74:00)

- `Auth_gamma(n)` means the authoritative current value is `n`.
- `Snap_gamma(m)` means the current value is at least the past value `m`.
- A snapshot is persistent and duplicable. It does not claim the current value
  remains exactly `m`.
- Draw a timeline `0 -> 3 -> 3 -> 7`; snapshots at 0 and 3 remain valid at 7.

#### Slide 29 -- Three rules prove the protocol (74:00--78:00)

Use the exact installed names in a footer:

```text
snapshot: Auth(n)              entails Snap(n)
increase: Auth(n), n <= n'     ~~> Auth(n') * Snap(n')
validity: Auth(n) * Snap(m)    entails m <= n
```

Exact APIs: `mono_nat_lb_own_get`, `mono_nat_own_update`, and
`mono_nat_lb_own_valid`.

Explain why the validity relation is now an order rather than equality. This is
the conceptual contrast with `ghost_map` agreement.

### Segment G -- combine the protocols and prove the client (78--85 minutes)

#### Slide 30 -- Product of protocols inside one IST (78:00--80:00)

Display:

```text
IST := exists m M c n,
  concrete_heap = m * repr(m) = M
  * ghost_map_auth gamma_m 1 M
  * concrete_count = c * c = n
  * mono_nat_auth_own gamma_c 1 n
```

The memory map and counter are independent resources combined with separating
conjunction. Every counted operation updates the relevant map entry (if any)
and moves `n` to `n+1` before reclosing the invariant.

#### Slide 31 -- Query specification with a ghost parameter (80:00--82:00)

The program takes no argument, but the spec is parameterized by a prior lower
bound:

```text
get_count() for any m
  pre:  Snap(m)
  post: exists n, ret = n * Snap(n) * [m <= n]
```

Because `Snap(m)` is persistent, it is not consumed. The initial client gets
`Snap(0)`. The query opens the IST, uses validity to prove `m <= n`, creates a
fresh snapshot of `n`, and returns the concrete `n`.

#### Slide 32 -- The complete client proof (82:00--84:00)

Trace the resources above the code:

1. First query with `Snap(0)` returns `before` and `Snap(before)`.
2. Carry that persistent snapshot across arbitrary `f()`.
3. Instantiate the second query's ghost parameter with `before`.
4. Its postcondition gives `before <= after`.
5. Integer arithmetic gives `0 <= after - before`.

Have the audience supply Step 3 before revealing it.

#### Slide 33 -- What arbitrary `f` can and cannot do (84:00--85:00)

- It may call public memory operations any number of times; those operations
  move the authority only upward.
- It cannot directly write the scoped private counter.
- It may diverge; the second observation and assertion are then unreachable.
- Thus the result is robust to unknown functional behavior, not a termination
  claim about `f`.

### Segment H -- generalize and hand off (85--90 minutes)

#### Slide 34 -- Choose an RA by asking what must remain compatible (85:00--87:00)

| Program protocol | Compatibility relation / library shape |
|---|---|
| stable agreement on a value | `agree` |
| unique capability | `excl` |
| central truth plus client views | `auth` |
| independently owned keyed entries | `ghost_map` |
| increasing number/integer | `mono_nat` / `mono_Z` |
| append-only history with prefix snapshots | `mono_list` |

Say explicitly that the carrier/operation/validity definitions are an
implementation of the protocol, not the place to begin explaining it.

#### Slide 35 -- A four-question design recipe (87:00--88:30)

1. What concrete state is private?
2. What capability or durable fact should a client receive?
3. Which simultaneous client claims must be compatible?
4. Which updates must preserve all compatible frames?

Then choose a standard RA and place its authority in the IST. Compose multiple
protocols with a product/separating conjunction.

#### Slide 36 -- Three equations to take to the afternoon (88:30--90:00)

End on exactly these:

```text
map authority + element     => exact lookup
full element                => update; fraction => read
counter authority + snapshot => past <= current
```

Point to the rule/notation appendix in the deck and say the afternoon exercises
are applications of these resource movements. Leave the final slide visible
for questions.

## 6. Board/animation choreography

The talk is easiest to follow if the same visual grammar is used throughout.

- Draw authoritative resources inside a blue “Mem IST” box.
- Draw caller fragments as amber cards outside the box.
- Draw persistent facts with a small pin or square marker.
- Use green only for a legal frame-preserving transition and red only for an
  incompatible resource combination.
- On every operation trace, move the caller token into the module boundary,
  perform the authority/fragment rule, reclose the boundary, and move the
  resulting token back. This makes pre/postconditions look like a call
  protocol, not disconnected logical formulas.
- Keep runtime state and ghost state on separate horizontal rows. The IST is
  the vertical bridge between them.

Recommended incremental reveals:

1. On Slide 10, show authority and element first; reveal `M[l] = v` only after
   asking what compatibility should imply.
2. On Slide 15, leave `k |-> 20` stationary while only the `l` card and map
   entry change.
3. On Slide 19, visibly tear a full card into two halves; join them before the
   store on Slide 21.
4. On Slide 32, pause before instantiating the second query with `before`.

The supplied Beamer deck uses static versions of these visuals for robustness;
the presenter can reveal rows progressively or reproduce the token movements
on a board/tablet.

## 7. Interaction plan and diagnostic questions

### Check 1: local reasoning (Slide 17)

Prompt: “A store updates `l`; Bob owns `k |-> 20` for `k != l`. Which fact about
`k` must be re-established by hand?”

Expected answer: none. If the audience says “prove the values do not alias,”
acknowledge that `l != k` is the one pure fact; after that, framing is automatic.

### Check 2: permission accounting (Slide 25)

Prompt: classify full, fractional, and persistent tokens by whether each
supports load, store, free, splitting, and arbitrary duplication.

Expected table:

| Token | Load | Store/free | Split | Duplicate freely |
|---|:---:|:---:|:---:|:---:|
| full owned | yes | yes | yes | no |
| proper owned fraction | yes | no | yes | no |
| discarded/persistent | yes as knowledge | no | n/a | yes |

Clarify that “load” for the persistent token means a specification that accepts
a general DFrac/persistent read-only element. The checked-in `MemA.load` is
currently parameterized by an owned rational `q`; the `ghost_map` teaching API
is what directly supports the persistent form.

### Check 3: snapshot use (Slide 32)

Prompt: “What should the ghost parameter of the second query be?”

Expected answer: `before`, using the snapshot returned by the first query.

If the audience proposes comparing the two returned integers without a token,
ask what connects the two independently quantified results across arbitrary
`f`; this reveals the purpose of the snapshot.

## 8. Likely questions and concise answers

**Is the authoritative map the concrete heap?**

No. It is ghost state. The IST asserts that it agrees with the concrete private
heap at simulation interaction points.

**Can a client update an element token by itself?**

No. The standard update combines the module's full authority with the caller's
full element. The token authorizes the method call; the method proof performs
the joint update.

**Why is a fraction sufficient to read?**

Any nonzero compatible fragment must agree with the authoritative value. A
read changes neither side, so it preserves every frame.

**Why can a fraction not write if this is a sequential setting?**

Because another component may own the remaining fraction even without
concurrency. Updating would invalidate that component's framed old-value fact.

**Is a persistent element an eternal exact fact?**

Yes for that frozen `ghost_map` entry. Discarding makes it impossible to
recover full write ownership. By contrast, a `mono_nat` snapshot is eternally a
lower bound, not an eternally current exact value.

**Why not represent the counter with a normal points-to token?**

A normal mutable points-to fact changes with the current value. The client
needs to retain arbitrarily many compatible facts about past observations;
`mono_nat` makes those facts persistent lower bounds.

**Does the snapshot stop `f` from accessing memory?**

No. It intentionally tolerates any number of accesses. It only rules out a
decrease of the private counter.

**What if the counter wraps?**

Then unbounded monotonicity is not the correct abstraction. Add a no-overflow
condition, use a wider/unbounded model, saturate, or specify modular behavior.

**Where is concurrency used?**

Nowhere in the main example. Framing and fractional permissions already matter
for modular sequential components and unknown callbacks. Similar resources are
also useful in concurrent Iris proofs, but that is not required here.

## 9. Source map for the presenter

### CRIS workshop and paper

- Workshop schedule: second day, 10:00--12:00, “Separation Logic and CRIS,”
  explicitly listing Cell motivation, Memory abstraction, and other RAs.
- CRIS paper PDF pp. 2--5: refinement/SL bridge and Cell motivation.
- CRIS paper PDF pp. 6--8: resources, authority/fragment intuition, and the
  Cell module-local invariant.
- CRIS paper PDF pp. 10--11: scoped private state protects abstractions from
  unverified contexts.
- CRIS paper PDF p. 15: the simulation is parameterized by the user-supplied
  module-local invariant.
- CRIS paper PDF p. 21, Figure 12 and Section 7: optional hybrid-memory appendix.

### Repository Cell bridge

- `ring/CellI.v:12`: private `cv`; `ring/CellI.v:17` and `ring/CellI.v:22`:
  direct get/set implementation.
- `ring/CellA.v:27`: pending token; `ring/CellA.v:31`: raw cell resource;
  `ring/CellA.v:35`: client fragment; `ring/CellA.v:39`: authority.
- `ring/CellA.v:64`: authority/fragment agreement;
  `ring/CellA.v:77`: joint update.
- `ring/CellIAproof.v:11`: Cell IST relating private state and resources.

### Current memory artifact

- `imp_system/mem/MemI.v:3`: concrete memory representation.
- `imp_system/mem/MemI.v:83`: alloc; `:96`: free; `:104`: load; `:111`: store.
- `imp_system/mem/MemA.v:7`: hand-rolled DFrac agreement map;
  `:10`: authoritative wrapper.
- `imp_system/mem/MemA.v:90`: singleton points-to;
  `:96`: fractional instance; `:111`: persistent discarded instance;
  `:115`: agreement; `:129`: fraction validity.
- `imp_system/mem/MemA.v:220`: alloc spec; `:225`: free spec;
  `:230`: load spec; `:235`: store spec.
- `imp_system/mem/MemIAproof.v:9`: relation between logical and concrete
  memory; `:224`: IST containing full authority.
- `imp_system/mem/MemIAproof.v:116`: lookup lemma; `:140`: update lemma;
  `:164`: free lemma; `:245`: alloc simulation; `:309`: load simulation;
  `:325`: store simulation.

Do not use `Mem.wf` from `MemI.v:9` on slides; the refinement proof uses the
sensible `mem_wf` from `MemIAproof.v:7`.

### Installed Iris `ghost_map`

File: `_opam/lib/coq/user-contrib/iris/base_logic/lib/ghost_map.v`.

- Lines 1--3: intended mutable/fractional/persistent-read-only interface.
- Lines 28--49: authority, element, and notation definitions.
- Lines 62--69: persistent/fractional instances.
- Lines 81--100: fragment validity and value agreement.
- Lines 136--150: persist and unpersist rules.
- Lines 176--187: allocation.
- Lines 221--235: authority/element lookup.
- Lines 250--279: insert, delete, and update.
- Lines 293--344: bulk insert/update variants.

The DFrac explanation comes from
`_opam/lib/coq/user-contrib/iris/algebra/dfrac.v:1--20`; constructors and public
notation are at lines 26--43, and the discard update is at lines 207--230.

### Installed Iris `mono_nat`

Base-logic wrapper:
`_opam/lib/coq/user-contrib/iris/base_logic/lib/mono_nat.v`.

- Lines 1--9: authoritative current value and persistent lower-bound overview.
- Lines 23--39: exact proposition names.
- Lines 49--54: lower-bound persistence.
- Lines 78--84: `mono_nat_lb_own_valid` (`m <= n`).
- Lines 90--92: `mono_nat_lb_own_get` (take a snapshot).
- Lines 103--127: allocation and monotone update.

Algebra-level implementation:
`_opam/lib/coq/user-contrib/iris/algebra/lib/mono_nat.v:5--18`, an authoritative
resource over `max_nat`; the order-validity and update rules are at lines
87--109.

### CRIS compatibility ports

- `CRIS/theories/iris_system/lib/ghost_map.v` ports the main `ghost_map` API to
  CRIS's `GRA`, strong allocation/update modality, and syntactic resources. The
  authority/element definitions are at lines 30--51; fraction/persistence at
  lines 63--72 and 139--142; lookup/insert/delete/update at lines 233--291; and
  syntactic reduction support starts at line 360.
- The local port comments out upstream `ghost_map_elem_unpersist`, so do not use
  that lemma in a CRIS exercise without porting it. It is mentioned only as an
  upstream Iris nuance in this plan.
- `CRIS/theories/iris_system/lib/mono_list.v` is the closest template for a
  CRIS `mono_nat` port: it wraps algebra-level resources with `CRIS.own`, adapts
  allocation to `o=>`, and provides syntactic-resource reductions.
- No local CRIS `mono_nat.v` exists at the time of writing. The counter portion
  of the deck is therefore a specification design with exact upstream laws,
  not a claim that the counted module is already implemented in this checkout.

## 10. Optional expansion for a two-hour allocation

If more than 90 minutes is available, use this order rather than slowing every
main slide:

1. **Five minutes:** open `MemA.v:7--10` and map the hand-rolled construction to
   `ghost_map`; explain why standard libraries matter for proof engineering.
2. **Seven minutes:** show the key lines of the mechanized store proof in
   `MemIAproof.v:325--346`, aligned with the five conceptual proof steps.
3. **Five minutes:** let pairs design a protocol for an append-only event log;
   then reveal `mono_list` (current list authority, persistent prefix
   snapshots).
4. **Three minutes:** show the CRIS paper's Figure 12 and explain that its hybrid
   operational/ownership choice is orthogonal to today's ghost-state choice.

This adds 20 minutes and fills the full 10:00--12:00 block after a 30-minute
Cell introduction.

## 11. Compression plan if time slips

- Five minutes behind: skip Slide 20 (Rust table) and deliver its caveat in one
  sentence on Slide 19; combine Slides 34--35 verbally.
- Ten minutes behind: additionally skip the detailed store pipeline on Slide
  16 and the persistent-flag pitfall's `unpersist` nuance on Slide 24.
- Never cut Slides 11 (IST placement), 15 (local store), 22--24 (correct DFrac
  semantics), 29 (three mono rules), or 32 (client proof). They carry the
  conceptual argument.

## 12. Preparation checklist

- Replace the placeholder presenter name in `presentation-slides.tex`.
- Decide whether the spoken language is Korean or English; keep mathematical
  names and source identifiers in English either way.
- Rehearse the 90-minute core once with the two audience pauses included.
- Print or expose the final three-rule slide during the afternoon exercises.
- If showing source, pre-open only `MemA.v:220--239`,
  `MemIAproof.v:224--228`, and the exact Iris lemma ranges listed above.
- If turning the counter section into a live Rocq exercise, port the upstream
  `mono_nat` wrapper into CRIS before the workshop; the current checkout has the
  algebra but not the CRIS base-logic adapter.
- State the counter policy and no-overflow assumption before the benchmark.
- Test the projector at 16:9 and verify amber/blue contrast in grayscale.
- Keep CAS, concurrency, camera construction, and the paper's hybrid memory
  example in backup slides unless the audience asks.
