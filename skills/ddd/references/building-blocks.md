# Building Blocks

This file covers the tactical side of DDD: working inside a single Bounded Context to model with Entities, Value Objects, Aggregates, Services, Domain Events, Repositories, Factories, Modules, and the Supple Design qualities. Pattern definitions are in `patterns.md` — this file is about *how to apply them and where they go wrong*.

Tactical work belongs inside a Bounded Context that already has a ubiquitous language. If the context or language is unclear, go to `strategic-design.md` first.

## Layered Architecture (or Hexagonal)

The first tactical decision: isolate the domain from infrastructure, UI, and application orchestration.

### The four layers (Evans' classic)

- **User Interface / Presentation**
- **Application Layer**: coordinates use cases, transactions, security. Holds no business rules. Talks to the domain through its interface.
- **Domain Layer**: the model. Entities, Value Objects, Aggregates, Services, Domain Events.
- **Infrastructure**: persistence, messaging, external systems.

### Hexagonal as alternative

Evans explicitly endorses Hexagonal Architecture as serving the same goal (isolation) potentially better. In Hexagonal: the Domain is the centre. Around it, Ports define the contracts. Adapters implement the ports for specific technologies (databases, message brokers, HTTP). The domain knows nothing about the adapters.

The choice between classical Layered and Hexagonal is largely about how you organise dependencies. Both achieve the same DDD goal: the domain model must be free of dependencies on UI, infrastructure, and application-orchestration concerns.

### Anti-patterns

- **Domain depending on infrastructure**: ORM annotations on domain Entities, framework base classes in the domain layer, persistence concerns leaking into business logic. The domain must not import infrastructure.
- **Application logic in the domain**: transaction management, authorisation checks, request routing — these are application concerns, not domain concerns. They live above the domain.
- **Anaemic application service**: application services that contain business logic instead of orchestrating it. The rule of thumb: an application service method should read like "load aggregate, call method on aggregate, save aggregate" — not contain branching business logic.

## Entity vs Value Object

This is the most common tactical decision and the one most often gotten wrong.

### The test

**Does this object have an identity that matters over time, independent of its attributes?**

- "Two of these with the same attributes — are they the same thing, or different?" → if same: Value Object. If different: Entity.
- "If I change one attribute, is it still the same thing?" → if yes (the identity persists): Entity. If no (it's a new value): Value Object.

### Examples

- `Money(100, "EUR")`: two instances are interchangeable → Value Object.
- `Address("Hauptstraße 1, 12345 Berlin")`: two people can have the same address → Value Object.
- `Customer(name="Alice", email="alice@example.com")`: a second person named Alice with that email is *not* the same customer → Entity.
- `Order(items, total, status)`: even two orders with identical contents are different orders → Entity.

### Anti-patterns

- **ID-everything**: assigning UUIDs to objects that don't need identity. A `Money` does not need an ID. Adding one means you have to manage uniqueness, equality, lifecycle for something that doesn't conceptually have those things.
- **Value Object with identity creeping in**: when a "value object" needs to be referenced from multiple places, audited, or has a lifecycle — it is becoming an Entity. Recognise the shift and refactor.
- **Mutable Value Objects**: Value Objects should be immutable. Mutability defeats their reason for existing. Operations on them return new instances.
- **Anaemic Value Objects**: a `Money` class with two getters and no operations. Value Objects should carry behaviour (currency conversion, arithmetic, formatting). They are not data containers.

## Aggregates

The hardest tactical decision. Aggregates define consistency boundaries and transaction boundaries.

### Finding aggregate boundaries

Ask: **what invariants must hold synchronously?** Whatever must be consistent within a single transaction goes inside one aggregate. Everything else is across aggregate boundaries and is eventually consistent.

Example: in an Order aggregate, the rule "total = sum of line items" must hold at every moment. So Order and OrderLines are one aggregate. The rule "customer's total lifetime spend = sum of order totals" need not hold instantly — Customer and Order are separate aggregates.

### Heuristics

- **Aggregates should be small.** (This is Vernon's heuristic, not Evans' — but it has held up well in practice.)
- **Reference other aggregates by ID, not by object reference.** Holding object references across aggregate boundaries leads to god aggregates.
- **One aggregate per transaction.** If a use case must modify two aggregates atomically, the boundary is wrong — or you need an asynchronous orchestration (Saga, process manager, or just sequential operations with compensation).
- **The aggregate root protects invariants.** External code calls methods on the root; internal entities are reached only through the root. Internal entities can be passed out for a single operation, but external code does not hold references to them.

### Anti-patterns

- **God Aggregate**: an aggregate spanning most of the domain. Symptoms: deeply nested object graph, many internal entities, transactions that lock large parts of the system. The true invariant boundary is smaller.
- **Cross-aggregate synchronous consistency**: a business operation that requires Aggregate A and Aggregate B to update atomically. Either the boundary is wrong (they should be one aggregate) or the operation is wrong (it should be eventual consistency with explicit handling of in-between states).
- **Aggregate as DTO**: treating the aggregate as a data structure rather than an enforcer of invariants. If the root's methods are setters and the invariants live in services, it's not an aggregate; it's a database record with extra steps.
- **Repository returning internal entities**: a repository should return the root. Returning internal entities lets external code modify them without the root's knowledge, defeating the aggregate's purpose.

## Services (Domain Services)

A Domain Service is for behaviour that is genuinely in the domain but doesn't naturally belong to one Entity or Value Object.

### When to use a Domain Service

- The behaviour involves multiple aggregates and no single one owns it
- The behaviour is a domain transformation that isn't naturally a method on an Entity (e.g. "transfer money between accounts" is awkward as a method on either Account)
- Domain experts speak of the behaviour as a thing-it-does, not a thing-it-is

### When *not* to use a Domain Service

- The behaviour belongs to a single Aggregate — put it on the Aggregate Root.
- The behaviour is orchestration (load A, call A.do(), save A) — that is an Application Service, not a Domain Service.
- The "service" only has CRUD methods — the domain logic belongs on Entities and Value Objects; the service is anaemic.

### Anti-patterns

- **Service-as-noun**: `OrderManager`, `UserHandler`, `PaymentProcessor`. The name says "I do things to nouns" — usually a sign the behaviour belongs on the noun itself.
- **Domain Service / Application Service conflation**: putting transaction management or authorisation inside a Domain Service. Those are application concerns.
- **Service explosion**: every business operation gets its own service. The domain becomes a procedural script with data on the side.

## Domain Events

A Domain Event represents something that happened in the domain that domain experts care about.

### Properties of a good Domain Event

- **Named in the past tense**, in the ubiquitous language: `OrderPlaced`, `PaymentReceived`, `AddressChanged`. Not `OrderUpdate` or `OrderEvent`.
- **Immutable** — it's a record of what happened
- **Carries the identities of involved entities and a timestamp**
- **Meaningful to a domain expert**, not just to the technical team

### When to use Domain Events

- Decoupling aggregates: Aggregate A raises an event; Aggregate B (in a separate transaction) reacts.
- Capturing the *reason* for state changes, not just the after-state.
- Distributed systems: events propagate state across nodes; an entity's state can be reconstructed from the events known to a node.
- Audit and history that has business meaning, not just technical logs.

### Anti-patterns

- **Domain Event as setter notification**: "the order's total field changed" — not a domain event. "An item was added to the order" might be.
- **Mutable events**: events are facts about the past. Past facts don't change. (If new information arrives, raise a new event — don't mutate the old one.)
- **System Events disguised as Domain Events**: "OrderCreatedInDatabase", "CacheInvalidated" — these are system concerns, not domain concerns.
- **Eventing for its own sake**: emitting events nothing listens to and nothing needs. Events have a cost (handling, ordering, observability). Justify each one.

## Repositories

A Repository provides query access to aggregates, expressed in the ubiquitous language.

### Rules

- **One Repository per Aggregate Root**, not per Entity. Internal entities are not separately retrieved.
- **Methods speak the ubiquitous language**. Not `findById`, `findByName`, `findByX` for every field — but `outstandingInvoicesForCustomer(...)`, `ordersAwaitingFulfilment(...)`. Method names express domain queries.
- **Returns fully constructed aggregates** (or lazy proxies that look like aggregates). Not partial DTOs.

### Anti-patterns

- **Repository per Entity**: defeats the aggregate boundary.
- **Generic Repository**: `Repository<T>` with `find(criteria)` — convenient but pushes query construction to client code, leaks persistence details, and the method names no longer speak the ubiquitous language.
- **Repository returning DTOs**: query optimisations creeping in. There are legitimate reasons for separate read models (CQRS), but if you're returning DTOs from your repository, it isn't doing the aggregate-loading job — it's a query service.
- **Domain logic inside the repository**: business rules in `findX` methods. The repository fetches; it does not decide.

## Factories

A Factory encapsulates complex construction of an Aggregate or Value Object.

### When to use a Factory

- Construction requires assembly across multiple objects
- Construction needs to enforce invariants that the constructor would violate (chicken-and-egg cases)
- Construction reveals too much internal structure to the client
- An aggregate must be created "as a piece" with all invariants satisfied from the start

### Anti-patterns

- **Factory for everything**: simple objects don't need factories. A two-field Value Object is constructed with `new`.
- **Factory that becomes a Service**: a "factory" with methods that do business operations beyond construction. Split it.
- **Factory ignoring invariants**: producing partly-constructed aggregates "to be filled in later". Aggregates are created whole and valid.

## Modules

Modules are part of the model, not just code organisation.

### Naming

Module names belong to the ubiquitous language. `com.company.billing` is fine; `com.company.utils` is a red flag — "utils" is not in any domain's vocabulary.

### Boundaries

Modules group concepts that change together and serve a coherent purpose. If a module has heterogeneous content (some classes change for reason A, others for reason B), consider splitting.

### Anti-patterns

- **Technical modules**: `models`, `services`, `controllers` — these are layers, not modules. They cut across domain concepts rather than grouping them.
- **Utility modules**: `utils`, `helpers`, `common` — usually grow into a dumping ground. Push the helpers into the modules that actually use them.
- **One-class modules**: a module per class is no organisation at all. Modules should hold cohesive sets.

## Supple Design qualities

These patterns make a model pleasant to work with and easy to change.

### Intention-Revealing Interfaces

Names express *what* and *why*, never *how*. `OrderConfirmation.send()` not `OrderConfirmation.executeEmailSendingProcedure()`. Names live in the ubiquitous language.

### Side-Effect-Free Functions

As much logic as possible into functions returning new values rather than mutating state. Commands (state-modifying operations) return nothing useful and are kept simple. Value Objects' operations are always side-effect-free.

### Assertions

State the post-conditions and invariants. If your language has no assertion mechanism, write tests or document them. Assertions are *contracts*: they tell clients what they can rely on.

### Standalone Classes

When you can, eliminate dependencies entirely. A standalone class is studied alone and reduces the cognitive load of the module enormously.

### Closure of Operations

Operations that take and return the same type are deeply composable. `Money + Money → Money`. `Set ∪ Set → Set`. They introduce no new dependencies. Look for these in Value Objects especially.

### Conceptual Contours

Look at where the design naturally bends under change. Decompose along those bend-lines, not along arbitrary "consistent granularity" rules. Some things should be coarse-grained; some fine-grained. Follow the domain.

## A workflow for tactical design conversations

When working inside a known Bounded Context, walk this:

1. **What aggregate are we touching?** What invariants must hold synchronously?
2. **What is the Aggregate Root?** What methods does it expose? What invariants does it protect?
3. **Inside the aggregate: Entities or Value Objects?** Run the identity test on each.
4. **Where does this behaviour belong?** First try: on an Entity or Value Object in the aggregate. Second try: Domain Service in the same context. Third (and last): Application Service for orchestration.
5. **Are there Domain Events?** What facts about the past does the rest of the system need to know?
6. **Is a Repository needed?** Only for the Aggregate Root, with methods in the ubiquitous language.
7. **Is the model getting hard to use?** Apply Supple Design patterns: rename for intent, push toward immutability, add assertions, simplify dependencies.

If at any step the design fights back — the Aggregate feels wrong, the invariants don't fit, the Service multiplies — that is signal. The model needs refactoring toward deeper insight. Go back to the strategic level if needed.
