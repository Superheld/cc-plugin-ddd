# Strategic Design

This file covers the strategic side of DDD: identifying bounded contexts, mapping their relationships, distilling the core domain, and applying large-scale structure. Pattern definitions are in `patterns.md` — this file is about *how to apply them*.

Strategic work always comes before tactical work. If a conversation jumps to Aggregates and Entities before bounded contexts are clear, pull back to here.

## Step 1: Find the bounded contexts

A Bounded Context is where one consistent model lives, with one ubiquitous language. The boundary is *semantic*, not technical.

### Signals that a context boundary exists

The same word means different things to different people. The same Entity carries different attributes that different teams care about. Two teams describe the "same" workflow incompatibly. A model that makes one stakeholder happy makes another confused. Translation is constantly happening between groups but no one names it.

**When you hear:**
- "We have a Customer table, but Sales and Support keep arguing about which fields belong on it" → two contexts, one model trying to serve both
- "The Order in our system has 47 fields because every team needs something different" → the Order Entity is spread across multiple contexts that haven't been separated
- "Marketing calls them 'leads', Sales calls them 'prospects', Onboarding calls them 'new customers', and they're all the same person" → either one entity moving through multiple contexts, or three contexts each with their own Entity. Usually the latter.

### Signals that a single context is correctly sized

One team can hold the entire model in their heads. The ubiquitous language is consistent — no one needs translation when speaking to a teammate. Domain experts and developers use the same words in the same way. Refactoring stays within reach.

### How to draw the boundary

Start with team structure (Conway's Law is descriptive, not prescriptive — but ignoring it is expensive). Then look at the linguistic boundaries: where does the meaning of key terms shift? Then look at the rates of change: parts of the model that change together belong together; parts that change for different reasons belong in different contexts.

A bounded context is too large when the language inside it fractures. It is too small when valuable concepts can't be expressed without crossing the boundary on every operation.

## Step 2: Establish the ubiquitous language

Within each Bounded Context, one language. Used by domain experts, developers, in code, in tests, in writing, in speech.

### How to build it

The language emerges through conversation with domain experts — it is not invented by developers. Listen for the terms experts use unprompted. Note where developers translate ("you mean the order, right?") and resist the translation: use the expert's word in code.

Watch for awkward terms. If a name feels forced or ambiguous, that is usually evidence the model is wrong, not the name. Try alternative phrasings out loud. The one that makes scenarios easy to say is usually closest to a good model.

### Anti-patterns

- **Pseudo-ubiquitous language**: a glossary document no one consults. Real ubiquitous language is in speech and code, not glossaries.
- **Developer dialect**: words used by developers that domain experts don't recognise (`Manager`, `Helper`, `Processor`, `Handler`). If a domain expert wouldn't use the word, the class probably shouldn't have it as its name.
- **Tech-leaked language**: `OrderDTO`, `OrderEntity`, `OrderModel`. Technical suffixes drag implementation into the language.
- **Translation drift**: developers and domain experts agree on a term, then weeks later, the developers have quietly redefined it. Catch this in conversation: "When you say 'reservation', do you still mean what we agreed last month?"

## Step 3: Draw the Context Map

For every pair of Bounded Contexts that interacts, decide the relationship. Use the patterns from `patterns.md` (section IV).

### Choosing the right relationship

The choice depends on three questions:

**Q1: Who needs whom for success?**
- Both teams need each other → **Partnership** (rare, expensive, but powerful)
- Upstream succeeds independently; downstream depends on upstream → upstream/downstream
- Neither cares → **Separate Ways**

**Q2: Can the teams coordinate?**
- High trust, frequent contact, shared release cadence → **Shared Kernel** (for a small piece) or **Partnership** (for whole interface)
- Good will but separate cadences → **Customer/Supplier**
- Limited or no coordination possible → **Conformist** or **Anticorruption Layer**

**Q3: How many integrators are there?**
- Few, ad-hoc → bilateral translation (Anticorruption Layer on each consumer)
- Many → **Open-host Service**, often with **Published Language**

### Quick-reference decision tree

When the situation maps to the simple case, this collapses Q1-Q3 into a single walk:

```
Do the two contexts need to talk at all?
├── No  → Separate Ways
└── Yes →
    Is the dependency mutual?
    ├── Yes →
    │   Is the shared piece small and stable?
    │   ├── Yes → Shared Kernel
    │   └── No  → Partnership
    └── No (one is clearly upstream, the other downstream) →
        Can the downstream team negotiate with the upstream?
        ├── Yes → Customer/Supplier Development Teams
        └── No  →
            Does the downstream have many peers consuming the same upstream?
            ├── Yes → ask the upstream for an Open-host Service / Published Language
            └── No  →
                Is the upstream's model acceptable as the downstream's?
                ├── Yes → Conformist
                └── No  → Anticorruption Layer
```

This is a starting heuristic, not a rule. Real systems combine patterns — e.g. an Open-host Service speaking a Published Language, with one downstream as Conformist (because the language fits its model) and another with an Anticorruption Layer (because it does not). When the tree gives an answer that feels wrong, return to the Q1-Q3 questions above and check which assumption broke.

### Anti-patterns in context mapping

- **Pretending no map exists**: every integration that crosses a context is governed by *some* relationship, named or unnamed. Unnamed relationships drift toward Big Ball of Mud.
- **Shared Kernel by accident**: two contexts that "happen to use" the same library or the same database table have a Shared Kernel without the discipline. Either formalise it or undo it.
- **Conformist by reflex**: when the team adopts an upstream model because changing is hard, not because conformity serves the domain. Often it would be cheaper long-term to build an ACL.
- **ACL as data mapper**: an ACL is supposed to translate between domain models, not just rename columns. If your "ACL" is a field-mapping table, you don't have an ACL; you have a translation layer for an undifferentiated model.

### The Big Ball of Mud question

When you find a region with no real model, no language discipline, and tangled dependencies — name it as a Big Ball of Mud. Do not try to apply DDD inside. Draw a boundary around it, and use ACL when integrating with it. Be alert to its tendency to sprawl.

## Step 4: Distil the Core Domain

Not all subdomains are equal. Distinguishing them is one of the highest-leverage moves in DDD.

### The three kinds of subdomain

- **Core Domain**: the differentiator. The reason this software exists rather than buying a generic alternative. The competitive advantage. The hard problem the business actually faces.
- **Generic Subdomain**: necessary but not differentiating. Authentication, billing, notification, audit trails. Often well-served by buying or using off-the-shelf components.
- **Supporting Subdomain**: specific to this business but not the differentiator. Custom but not strategic.

### How to find the Core

Ask: if this software were bought from a vendor instead of built, what would still need to be built in-house? That is the Core.

Ask: what part, if removed, would make this product worthless to its customers? That is the Core.

Ask: where do the deep domain conversations happen — what do domain experts argue about for hours? That is usually near the Core.

### Anti-patterns

- **Core inflation**: treating most of the system as core because "it's all important". Importance is not the criterion. Differentiation is.
- **Core hidden in generic**: building authentication or billing as if it were a strategic project. It rarely is. Buy or use libraries unless authentication itself is the product.
- **Tech-team in the Core**: assigning the team's strongest developers to generic subdomains because the work is technically interesting. Top talent goes to the Core. The Core is where deep modelling and supple design pay off most.
- **Domain Vision Statement as marketing copy**: the vision statement is a tool for orienting the team, not a website tagline. It captures what makes the domain model valuable. Specific, short, revised as insight grows.

### Distillation patterns to consider

Once the Core is identified:

- **Segregated Core**: physically separate Core code from supporting code in the codebase
- **Highlighted Core**: a short document (or a tag in the model) that makes the Core visible without restructuring code
- **Generic Subdomains**: extract and isolate; consider replacing with off-the-shelf
- **Cohesive Mechanisms**: extract complex computational machinery into a separate framework so the Core can focus on the "what" not the "how"
- **Abstract Core**: when the Core has many subdomain implementations, find the abstract concepts that all share

## Step 5: Apply Large-scale Structure (only if needed)

Large-scale Structure is a system-wide organising principle (Responsibility Layers, System Metaphor, Knowledge Level, Pluggable Component Framework). It is the strategic pattern most often misapplied.

### When to apply

The system has grown large enough that no team can grasp the whole. Patterns of organisation recur across the codebase but are not made explicit. The team is starting to lose track of where responsibilities belong.

### When *not* to apply

The system is small enough that everyone holds it in their heads. The team is still discovering the domain. The structure would be a guess, not an emergent pattern.

**Evans is emphatic on this:** an ill-fitting large-scale structure is worse than no structure at all. Aim for the *minimal* structure that solves the problems that have actually emerged. Let it evolve. Be ready to change or drop it.

### Anti-patterns

- **Architecture-first**: deciding on a Large-scale Structure before the domain is understood. The structure will not fit the eventual model.
- **Comprehensive structure**: trying to assign every element of the system to a layer/metaphor/component. Less is more. Cover only what genuinely benefits.
- **Frozen structure**: keeping a Large-scale Structure once it no longer fits. Evolving Order means changing the structure type when needed.

## A workflow for strategic design conversations

When the user wants to "design a system" or "model a domain", run this checklist mentally. If a step is unclear, that is where to focus the conversation:

1. What is the **domain**? (Plain English, no software words.)
2. What are the **subdomains**? Which is **core**, which **generic**, which **supporting**?
3. What are the **bounded contexts**? How are they bounded — by team, by linguistic shift, by rate of change?
4. Inside each context, what is the **ubiquitous language**? Where did it come from — domain experts or developers?
5. What is the **context map** — relationships between contexts?
6. For the Core: what is the **Domain Vision Statement**? What deep modelling investment does the core warrant?
7. Only now: tactical design (see `building-blocks.md`).

If the user wants to skip ahead to tactical, do not skip with them. Pull back. The work of steps 1-6 saves enormous tactical rework later.
