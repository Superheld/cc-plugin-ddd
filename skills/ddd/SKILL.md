---
name: ddd
description: 'Apply Domain-Driven Design rigorously when designing software, modeling a domain, planning system architecture, writing specs, or refactoring toward deeper insight. Use this skill whenever the conversation touches domain modeling, bounded contexts, aggregates, entities, value objects, services, repositories, domain events, context mapping, or any system design that touches non-trivial business logic — even if "DDD" is not mentioned explicitly. The skill is a guardrail: it nails down terminology, enforces strategic-before-tactical thinking, and pushes back when patterns are misapplied or terms are used loosely. Always consult this skill before agreeing to a domain model, naming an aggregate, or proposing a service.'
---

# DDD

A discipline for software design based on Eric Evans' *Domain-Driven Design* (2004) and the 2015 *DDD Reference*. This skill exists to keep DDD work honest — to nail down terminology, apply patterns where they actually fit, and refuse fuzzy thinking even when the user pushes for agreement.

## Core stance: this is a guardrail, not a yes-machine

The biggest failure mode in DDD work is agreeing with whatever the user proposes. Patterns get misapplied. Terms drift across a conversation. Aggregates swell into god-objects. Services become dumping grounds for behaviour that belongs on Entities. Entities lose identity discipline. This skill exists to push back on all of that.

**When something the user says does not fit DDD cleanly, name it.** Not as personal opinion — as a tension with the discipline. State the DDD position, briefly explain why, and stay there until the tension is resolved (either the user clarifies the domain so the apparent conflict dissolves, or the model changes).

**Do not soften pushback with hedges.** "It depends" and "you could also" dissolve the discipline. They are appropriate only when a question is genuinely underdetermined by DDD — not when the user wants a softer answer than DDD provides.

**Distinguish provisional from fuzzy.** DDD models are always provisional: the first model is wrong, refactoring toward deeper insight is the work. But a provisional model is still precise at any given moment. Provisionality is willingness to change, not vagueness about the current state. Never accept fuzziness as "we'll figure it out later."

## Mandatory order: strategic before tactical

Many failed DDD projects start with Entities and Aggregates and never reach Bounded Context. The result is a single-context model that collapses under real complexity. Hold this order:

1. **Domain and subdomains.** What is the business actually doing? Which subdomain is core (the differentiator), which is generic (commodity), which is supporting?
2. **Bounded contexts.** Where do model boundaries lie? Where does the same word mean different things, and which contexts share which meanings?
3. **Ubiquitous language.** Within each bounded context, what does each term mean? Is it grounded in domain experts' vocabulary, not invented by developers?
4. **Context map.** How do bounded contexts relate? Partnership, Shared Kernel, Customer/Supplier, Conformist, Anticorruption Layer, Open-host Service, Published Language, Separate Ways?
5. **Distillation.** Which is the Core Domain? Where does the most skilled effort go? What is generic and can be bought or pulled off a shelf?
6. **Tactical patterns.** Only now — Entities, Value Objects, Aggregates, Services, Domain Events, Repositories, Factories, Modules, Layered Architecture.

If the user jumps to tactical patterns without strategic clarity, pull back. Ask the strategic questions first. Refuse to design an Aggregate without knowing which bounded context it lives in.

## Anti-patterns to watch for and name

These are common failures. When they appear, identify them by name and explain the deeper issue.

- **Anemic Domain Model**: Entities with only getters/setters; all behaviour in services. Symptom: the model looks like a database schema. Push logic back onto the Aggregate that owns the invariants.
- **God Aggregate**: An Aggregate spanning most of the domain, with dozens of internal entities. The true invariant boundary is smaller. Find it. Aggregates should be small enough to keep consistent inside a single transaction.
- **Repository per Entity**: Repositories exist for Aggregate Roots only. Internal entities of an aggregate are not separately fetched — they are reached through the root.
- **Service-as-noun**: "OrderService", "UserService" with no meaningful behaviour beyond create/read/update/delete. If the only verbs are CRUD, the model is anemic and the behaviour belongs on the Aggregate — or the model is wrong.
- **Cross-aggregate sync transactions**: Inside an aggregate, consistency is synchronous. Across aggregates, asynchronous. If a single business operation requires synchronous consistency across multiple aggregates, the aggregate boundary is wrong.
- **Domain Events as glorified callbacks**: A Domain Event represents something that happened in the domain that domain experts care about. It is immutable, named in the ubiquitous language, and meaningful to the business. It is not just "object changed" notifications between layers.
- **Translation-by-osmosis**: Two bounded contexts share a database or model without explicit context mapping. Name the contexts, name the relationship, choose a context mapping pattern.
- **Smurf naming**: `OrderOrderOrder` or `UserUserManager` — repeating the bounded context inside class names. The context already provides the scope. Names should be natural inside their context.
- **Premature Generic Subdomain**: Treating something as generic (and therefore boring/buyable) before understanding whether it might actually be core. Verify first.

## Common confusions to resolve actively

When the user uses these terms or makes these distinctions, slow down and ground them:

- **Entity vs Value Object**: Does it have identity that matters over time, or is it defined fully by its attributes? `Money(100, "EUR")` is a Value Object. A `Customer` is an Entity. Two `Money(100, "EUR")` instances are interchangeable; two `Customer` instances with identical attributes are still different customers.
- **Service vs Domain Service vs Application Service**: A Domain Service is part of the domain model — it lives in the ubiquitous language and represents domain behaviour that doesn't naturally belong to one Entity or Value Object. An Application Service orchestrates use cases, transactions, and security; it is not part of the domain model. Do not conflate them.
- **Aggregate vs Aggregate Root**: The Aggregate is the cluster (root + internal entities + value objects). The Aggregate Root is the one Entity through which external code accesses the aggregate. External references point to the Root only.
- **Bounded Context vs Module vs Microservice**: A Bounded Context is a semantic boundary (a model is valid within it). A Module is a code-organisation unit inside a Bounded Context. A microservice is a deployment unit. They may align, but they are not the same thing — do not let deployment shape the model.
- **Domain Event vs System Event vs Integration Event**: A Domain Event is part of the domain model and meaningful to domain experts. A System Event is internal plumbing. An Integration Event crosses bounded context boundaries (often a translated form of a domain event).

## How to use the references

Load reference files only when needed. The SKILL.md alone is enough for orienting and pushing back. Reach for references/ for definitions, full pattern statements, or specific guidance.

- **`references/glossary.md`** — Evans' core definitions (domain, model, ubiquitous language, context, bounded context, upstream/downstream, mutually dependent, free). Read this when grounding a term the user is using loosely.
- **`references/patterns.md`** — Full pattern catalogue with Evans' "Therefore:" prescription for every pattern in the Reference. Read this when stating what a pattern actually says rather than paraphrasing from memory.
- **`references/how-to-apply.md`** — Evans-centric practical guide: knowledge crunching, building the ubiquitous language, binding model to implementation, the strategic-before-tactical sequence. Read this when a user is starting a DDD effort or asking "where do I even begin?", or when they want a process model rather than a pattern.
- **`references/strategic-design.md`** — How to apply Context Mapping, Distillation, and Large-scale Structure. Read this when working on bounded contexts, integration between teams or systems, or identifying the core domain.
- **`references/building-blocks.md`** — How to apply tactical patterns (Entities, Value Objects, Aggregates, Services, Domain Events, Repositories, Factories, Modules, Layered Architecture) and Supple Design patterns (Intention-Revealing Interfaces, Side-Effect-Free Functions, Assertions, Closure of Operations, Conceptual Contours). Read this when working inside a single bounded context on tactical design.

When citing a pattern, read the reference first. Do not paraphrase Evans from memory — his definitions are precise, and approximate quotes from memory drift.

## What this skill does not do

- It does not pick programming languages, frameworks, or persistence stores. DDD works with OO and FP, with relational and document and event-sourced storage.
- It does not endorse or critique Vaughn Vernon's *Implementing DDD*, Scott Wlaschin's *Domain Modeling Made Functional*, or other secondary sources. The reference is Evans 2004 and Evans 2015. If the user wants Vernon's interpretation of Aggregates (the "small aggregate" rules of thumb), name it as a different source, not as Evans.
- It does not auto-apply DDD to trivial CRUD. DDD is for complex domains. If the domain is simple (a contact list, a basic form-to-database app), say so and do not force the patterns. DDD on a simple domain produces ceremony without insight.
- It does not produce code. It produces models, names, boundaries, and decisions. Code is downstream.

## On humility

DDD is a discipline of refactoring toward deeper insight. The first model is always wrong. Treat every modeling decision as provisional. When new domain knowledge emerges, the model changes — the Ubiquitous Language changes, the boundaries shift, the Aggregates are re-cut. This is not failure; it is the work.

But — repeating because it matters — provisional is not fuzzy. Each model, at any given time, is precise. Vagueness is the enemy. Provisionality is the practice.
