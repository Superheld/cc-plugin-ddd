# Pattern Catalogue

The complete pattern set from Eric Evans' *DDD Reference* (2015). Six groups, around forty patterns. When you need to state what a pattern actually says — read here, do not paraphrase from memory.

Patterns marked with `*` were introduced after the 2004 book.

## Map of the pattern groups

1. **Putting the Model to Work** — foundational practices that make modelling effective at all
2. **Building Blocks of a Model-Driven Design** — tactical patterns inside a Bounded Context
3. **Supple Design** — design qualities that make models a pleasure to work with and change
4. **Context Mapping for Strategic Design** — relationships between Bounded Contexts
5. **Distillation for Strategic Design** — separating core from supporting from generic
6. **Large-scale Structure for Strategic Design** — organising principles spanning the whole system

---

## I. Putting the Model to Work

### Bounded Context

**Problem:** Multiple models inevitably arise on large projects. Code based on distinct models, mashed together, becomes buggy and confused.

**Therefore:** Explicitly define the context within which a model applies. Set boundaries in terms of team organisation, application sub-parts, and physical artefacts like codebases and database schemas. Apply Continuous Integration to keep concepts and terms strictly consistent within these bounds. Standardise a single development process within the context.

### Ubiquitous Language

**Problem:** Within a single Bounded Context, language fractures. Domain experts use one jargon, developers another, code uses a third. Translation blunts communication.

**Therefore:** Use the model as the backbone of a language. Commit the team to exercising that language relentlessly — in diagrams, writing, and especially speech. A change in the language is a change to the model. When difficulties surface, experiment with alternative expressions, which reflect alternative models. Then refactor the code to match.

### Continuous Integration

**Problem:** Once a Bounded Context has more than three or four people, the model tends to fragment.

**Therefore:** Institute a process of merging all code and other implementation artefacts frequently, with automated tests to flag fragmentation quickly. Relentlessly exercise the ubiquitous language to hammer out a shared view of the model.

### Model-Driven Design

**Problem:** Models that don't map cleanly to code lose value. Complex mappings between models and design are unmaintainable.

**Therefore:** Design a portion of the software system to reflect the domain model literally, so mapping is obvious. Revisit the model and modify it to be more naturally implementable. Demand a single model that serves both purposes well.

### Hands-on Modelers

**Problem:** If modellers are separated from implementation, they lose feel for implementation constraints; if implementers don't engage the model, refactorings weaken it.

**Therefore:** Any technical contributor to the model must spend time touching code. Anyone changing code must learn to express the model through it. Every developer must engage with domain experts in dynamic exchange via the ubiquitous language.

### Refactoring Toward Deeper Insight

**Problem:** Sophisticated models don't appear up front. They emerge through iteration.

**Therefore:** Refactor not only for technical reasons but for domain insight. A deep model captures the essential and sloughs off the superficial. Cultivate close collaboration between developers interested in the domain and domain experts.

---

## II. Building Blocks of a Model-Driven Design

### Layered Architecture

**Problem:** When UI, database, and infrastructure code mix with business logic, the domain becomes impossible to reason about.

**Therefore:** Isolate the domain model from infrastructure, UI, and even application logic. Partition into layers. Each layer depends only on layers below. Concentrate domain-model code in one layer. Hexagonal Architecture or similar patterns serve the same goal — the key is isolation.

### Entities (a.k.a. Reference Objects)

**Problem:** Some objects are defined by a thread of identity through time, not by their attributes. Mistaken identity corrupts data.

**Therefore:** When an object is distinguished by identity rather than attributes, make identity primary to its definition. Keep the class focused on lifecycle continuity and identity. Define a means of distinguishing each object regardless of form or history — usually a unique symbol. The model must define what it means to be the same thing.

### Value Objects

**Problem:** Many objects have no conceptual identity. Forcing identity on them adds work and muddles the model.

**Therefore:** When you care only about attributes and logic, classify it as a Value Object. Treat it as immutable. All operations should be Side-Effect-Free Functions. Give it no identity. Avoid the design complexity necessary to maintain Entities.

### Domain Events *

**Problem:** Entities track state but rarely explain the causes of state changes. Audit trails don't carry meaning. Distributed systems can't be globally consistent; eventual consistency needs explicit handling.

**Therefore:** Model information about activity in the domain as a series of discrete events. Each event is a domain object. Domain Events are immutable. They include a timestamp and the identities of involved entities. Often a second timestamp records when the event entered the system. Domain Events are distinct from system events — though a system event often carries a domain event into the system. In distributed systems, an entity's state can be inferred from the events known to a node.

### Services

**Problem:** Some domain concepts are not naturally things. Forcing them onto an Entity or Value distorts both.

**Therefore:** When a significant process or transformation is not a natural responsibility of an Entity or Value Object, add it to the model as a standalone interface declared as a Service. Define a service contract (a set of assertions). State them in the ubiquitous language of the bounded context. The service name is part of the ubiquitous language.

### Modules (a.k.a. Packages)

**Problem:** Modules are usually treated as a technical concern, not a modelling concern. Coupling and cohesion are reduced to mechanical metrics.

**Therefore:** Choose modules that tell the story of the system and contain a cohesive set of concepts. Give modules names that become part of the ubiquitous language. Modules are part of the model. If coupling stays high after refactoring, look for an overlooked concept that would unify or separate the elements.

### Aggregates

**Problem:** Guaranteeing consistency across complex object associations is hard. Locking schemes cause contention; distributed objects fragment.

**Therefore:** Cluster Entities and Value Objects into Aggregates and define boundaries around each. Choose one Entity as the root. External objects hold references to the root only (internal references may be passed for use within a single operation). Define invariants for the aggregate and enforce them at the root.

Use the same boundaries to govern transactions and distribution. Within an aggregate, consistency is synchronous. Across aggregates, asynchronous. Keep an aggregate together on one server.

When boundaries don't work for transactions or distribution, reconsider the model — the friction often reveals an important domain insight.

### Repositories

**Problem:** Queries scattered everywhere muddle the model. Database access infrastructure overwhelms client code. Unconstrained queries breach aggregate encapsulation.

**Therefore:** For each Aggregate type that needs global access, create a Repository providing the illusion of an in-memory collection of all instances of the root type. Provide add/remove methods and selection methods using criteria meaningful to domain experts. Return fully instantiated aggregates (or lazy proxies). Provide Repositories only for Aggregate Roots that need direct access.

### Factories

**Problem:** Constructing a complex Aggregate or Value Object reveals too much internal structure. Putting construction logic on the constructed object distorts its design.

**Therefore:** Shift responsibility for creating complex instances to a separate Factory object. The Factory may have no domain responsibility but is still part of the domain design. Provide an interface that hides assembly. Create an entire Aggregate as a piece, enforcing invariants from the start.

---

## III. Supple Design

### Intention-Revealing Interfaces

**Therefore:** Name classes and operations to describe effect and purpose, without reference to implementation. Names conform to the ubiquitous language. Write a test for behaviour before creating it, to force client-developer thinking.

### Side-Effect-Free Functions

**Therefore:** Place as much logic as possible into functions — operations returning results with no observable side effects. Strictly segregate commands (which modify state) into simple operations that return no domain information. All operations of a Value Object should be Side-Effect-Free Functions.

### Assertions

**Therefore:** State post-conditions of operations and invariants of classes and aggregates. If assertions cannot be coded in your language, write automated tests or document them. Assertions define contracts of services and entity modifiers. They define invariants on aggregates.

### Standalone Classes

**Therefore:** When you can, go all the way: eliminate every dependency a class doesn't need. A self-contained class can be studied alone. Every such class significantly eases module understanding.

### Closure of Operations

**Therefore:** Where it fits, define an operation whose return type is the same as the type of its argument(s). If the implementer has state used in the computation, the implementer is effectively an argument too. Such operations are closed under the type. Closure provides a high-level interface with no new dependencies. Most often applied to Value Objects.

### Declarative Design

**Therefore:** Where possible, write parts of the program as executable specifications. A precise description of properties controls the software. Many of the benefits emerge once you have Intention-Revealing Interfaces, Side-Effect-Free Functions, and Assertions combinable into meaningful statements.

### Drawing on Established Formalisms

**Therefore:** Reuse refined conceptual systems from your domain or others — accounting, well-developed math, established frameworks. Specialised math is clean, combinable by clear rules, and easy for people to grasp. Look for it and dig it out.

### Conceptual Contours

**Therefore:** Decompose design elements (operations, interfaces, classes, aggregates) into cohesive units, considering important divisions in the domain. Observe the axes of change and stability through successive refactorings; look for underlying conceptual contours. Align the model with the consistent aspects of the domain.

---

## IV. Context Mapping for Strategic Design

### Context Map

**Problem:** Without a global view, contexts blur into each other. Relationships impose constraints that surface through non-technical channels and confuse design.

**Therefore:** Identify each model in play on the project and define its Bounded Context, including the implicit models of non-OO subsystems. Name each context; the names join the ubiquitous language. Describe points of contact — translations, sharing, isolation, levels of influence. Map the terrain first; plan transformations later.

### Partnership *

**Therefore:** Where development failure in either of two contexts would mean delivery failure for both, forge a Partnership between the teams. Institute coordinated planning and joint management of integration. Teams cooperate on the evolution of their interfaces. Interdependent features ship for the same release. A clear process governs integration — for instance, a shared test suite run on the server system's CI.

### Shared Kernel

**Therefore:** Designate with an explicit boundary some subset of the domain model that two teams agree to share — model, code, possibly database. Keep the kernel small. Changes require consultation between teams. Run a Continuous Integration process for the kernel. Integrate functioning systems frequently, somewhat less often than within-team CI.

### Customer/Supplier Development

**Therefore:** Establish a clear customer/supplier relationship between the upstream and downstream teams. Downstream priorities factor into upstream planning. Negotiate and budget tasks for downstream needs. Jointly develop automated acceptance tests; add them to the upstream's CI to let the upstream change freely without breaking the downstream.

### Conformist

**Therefore:** When the upstream has no motivation to serve the downstream, eliminate translation complexity by slavishly adhering to the upstream model. Conformity cramps the downstream's design but simplifies integration enormously and shares the ubiquitous language.

### Anticorruption Layer

**Therefore:** As a downstream client, create an isolating layer that provides the upstream's functionality in terms of your own domain model. The layer talks to the other system through its existing interface, requiring little or no change to the other system. It translates in one or both directions as needed.

### Open-host Service

**Therefore:** When a subsystem must integrate with many others, define a protocol giving access to it as a set of services. Open the protocol to all who need to integrate. Enhance it for general integration needs. Use one-off translators only for idiosyncratic clients. The host is upstream; clients are downstream and may be Conformist or Anticorruption Layer.

### Published Language

**Therefore:** Use a well-documented shared language that expresses the necessary domain information, translating as needed into and out of it. Often combined with Open-host Service. Many industries have established published languages as data interchange standards.

### Separate Ways

**Therefore:** If two sets of functionality have no significant relationship, declare a Bounded Context to have no connection to the others. Developers find simple, specialised solutions within the small scope. Integration is always expensive; sometimes the benefit is too small to justify it.

### Big Ball of Mud *

**Therefore:** When you encounter a system region where models are mixed and boundaries are inconsistent, do not try to apply sophisticated modelling to it. Draw a boundary around the entire mess and designate it a Big Ball of Mud. Be alert to its tendency to sprawl into other contexts.

---

## V. Distillation for Strategic Design

### Core Domain

**Therefore:** Boil the model down. Define a Core Domain and provide a way to easily distinguish it from supporting model and code. Bring the most valuable and specialised concepts into sharp relief. Make the core small. Apply top talent to it and recruit accordingly. Justify investment in other parts by how they support the core.

### Generic Subdomains

**Therefore:** Identify cohesive subdomains that are not the motivation for your project. Factor them out and place them in separate modules. Leave no trace of your specialties in them. Give them lower priority than the core. Consider off-the-shelf solutions or published models for them.

### Domain Vision Statement

**Therefore:** Write a short description (about one page) of the core domain and the value it brings — the value proposition. Ignore aspects that don't distinguish this domain model from others. Show how the model serves and balances diverse interests. Keep it narrow. Write it early and revise it as insight grows.

### Highlighted Core

**Therefore:** Either write a very brief document (three to seven sparse pages) describing the core domain and primary interactions among core elements; or flag core elements within the model repository so developers can effortlessly tell what is in or out of the core. Or both.

### Cohesive Mechanisms

**Therefore:** When computation grows so complex that the "what" is swamped by the "how", partition the mechanism into a separate lightweight framework with an Intention-Revealing Interface. The domain model expresses the problem; the framework handles the solution.

### Segregated Core

**Therefore:** Refactor the model to separate core concepts from supporting players. Strengthen the cohesion of the core while reducing its coupling. Factor generic or supporting elements into other objects in other packages, even if this means refactoring the model in ways that separate highly coupled elements.

### Abstract Core

**Therefore:** Identify the most fundamental differentiating concepts in the model. Factor them into distinct classes, abstract classes, or interfaces. Design this abstract model so it expresses most of the interaction between significant components. Place the abstract overall model in its own module; leave specialised implementations in their own modules by subdomain.

---

## VI. Large-scale Structure for Strategic Design

### Evolving Order

**Therefore:** Let the conceptual large-scale structure evolve with the application, possibly changing to a different type of structure entirely. Don't over-constrain detailed design decisions that need detailed knowledge. Apply large-scale structure only when one greatly clarifies the system without forcing unnatural constraints. An ill-fitting structure is worse than none. Aim for a minimal set that solves problems that have emerged. Less is more.

### System Metaphor

**Therefore:** When a concrete analogy captures the team's imagination and leads thinking in a useful direction, adopt it as a large-scale structure. Organise the design around it; absorb it into the ubiquitous language. Continually re-examine the metaphor for overextension or inaptness; drop it if it gets in the way.

### Responsibility Layers

**Therefore:** Look at conceptual dependencies in your model and the varying rates and sources of change in different parts of your domain. If natural strata appear, cast them as broad abstract responsibilities. Refactor so the responsibilities of each object, aggregate, and module fit neatly within one layer.

### Knowledge Level

**Therefore:** Create a distinct set of objects that describe and constrain the structure and behaviour of the basic model. Keep these concerns separate as two "levels": one concrete, the other reflecting rules and knowledge that a user or super-user can customise. (See Fowler, *Analysis Patterns*, 1997.)

### Pluggable Component Framework

**Therefore:** Distil an abstract core of interfaces and interactions; create a framework allowing diverse implementations of those interfaces to be freely substituted. Allow any application to use those components, so long as it operates strictly through the interfaces of the abstract core. Only attempt this in a very mature, deep, distilled model — usually after several applications have been built in the same domain.

---

## On using this catalogue

Read a pattern entry before stating what the pattern says. Evans' formulations are precise. Common drift to watch for:

- **Aggregates**: people often quote the "small aggregate" rule from Vaughn Vernon as if it were Evans. It is not. Evans says aggregates should be a consistency boundary; Vernon's "design small aggregates" is a heuristic from later experience. Use Vernon's heuristic — but attribute it correctly.
- **Domain Events**: people sometimes treat any pub/sub message as a Domain Event. Evans is specific: it is something domain experts care about, immutable, with timestamp and entity identities.
- **Services**: people often conflate Evans' Service (domain operation) with Application Service (use case orchestration). Evans' Reference is talking about the former.
- **Bounded Context**: people sometimes equate this with microservice. They can align, but a Bounded Context is a semantic boundary, not a deployment boundary.
