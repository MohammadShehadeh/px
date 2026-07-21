---
name: px-debug
description: House debugging method — localize top-down (page → section → block → component) to the defect site, map references in both directions, trace to the root cause (interrogating where/why/how to prune candidates for wrong-value bugs), and account for the blast radius before fixing. Use when investigating a bug, regression, or unexpected behavior in TypeScript/React/Next.js code.
---

# Debug

Bugs are found by **narrowing location**, fixed by **tracing causation**, and shipped by **checking the blast radius**. Never fix at the first place the bug is visible.

## 1. Localize — walk the tree down

Reproduce first; no fix without a reproduction. Then narrow following the render hierarchy:

**page → section → block → component**

At each level ask "does the bug still reproduce below this point?" and descend. Flat trees (the `px-conventions` skill's `components` rule) keep this walk short — if localizing takes more than a few jumps, that is itself a finding.

The result is the **defect site**: where the bug *reproduces* — not yet where it *lives*.

## 2. Gather — map references in both directions

References are many-to-many: the defect component is used by many consumers, and itself uses many dependencies. Before touching anything, build the local graph:

- **Inbound — who references it**: grep the file's import path. Direct imports with no barrels (`px-conventions`: `structure` rule) mean every consumer is one grep away: `rg "components/date-range-picker"` → the complete consumer list.
- **Outbound — what it references**: its imports — hooks, services, context, constants, lib.

## 3. Trace — follow references to the root cause

Walk outbound from the defect site toward the data: component → hook → service → boundary. The bug usually lives where a **decision** is made (pure logic), not where its result is rendered.

- The greppable log prefix (`'Error in <fn>::'`, `px-conventions`: `errors` rule) tells you which layer produced the bad value.
- Stop at the first place where correct input becomes wrong output. That is the actual bug; everything downstream is symptom.

### Wrong-value bugs: interrogate where / why / how

A crash hands you a stack trace; a **wrong value** ("the table shows the wrong total") does not — so you prune the candidate set by intent, not by reading every line. At each site on the path from step 2's graph, ask:

- **Where** does this value come from? Every transform between boundary and render is a candidate — the DTO mapper, a `??` default, a cast, a `new Date(...)`, a `.find()`, a cache, an optimistic overlay.
- **Why** is it shaped this way *here*? Each of those encodes an **assumption** (this field is always present; this string is a parseable date; this list has the row). Name the assumption. The one that doesn't hold for the failing input is the bug.
- **How** does each consumer use it — as the producer intended? A value correct at its source but read in the wrong unit / shape / timezone by one consumer is a **coupling** bug, not a production bug → carry it to step 4.

Each assumption you *confirm* holds eliminates that site; the one you can't confirm is where to look. That is how the probability space narrows — most branches rule themselves out before you touch code.

## 4. Blast radius — account before you fix

The root cause often sits in shared code, and the higher the abstraction, the more consumers one edit touches. Before editing:

- List every inbound reference of the file you're about to change (step 2's grep, run on the *cause* file).
- For each consumer: does the fix change a contract they rely on, or only the broken behavior?
- Fix once at the shared root — a guard in the shared function beats a patch in every caller.
- If some consumers depend on the "broken" behavior, the coupling is wrong: the shared thing was two things. Split the abstraction; do not add a boolean flag to keep everyone happy — that is wrong coupling being papered over.

## 5. Verify

- The original reproduction path is clean: page → section → block → component.
- Every consumer from step 4's list still behaves — typecheck, tests, and a look at the screens sharing the code.
- Add the regression test where the decision lives (pure logic, `px-conventions`: `testing` rule), not where the symptom rendered.
