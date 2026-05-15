# Glossary

Core definitions from Eric Evans' *DDD Reference* (2015, CC BY 4.0). These are the load-bearing terms of DDD. When the user uses one of these loosely, ground it here before continuing.

## Definitions

### domain

A sphere of knowledge, influence, or activity. The subject area to which the user applies a program is the domain of the software.

**In practice:** "the domain" is what the business does — not the software. Banking is a domain. Logistics is a domain. Heating cost billing is a domain. The software *serves* the domain; it does not *define* it.

### model

A system of abstractions that describes selected aspects of a domain and can be used to solve problems related to that domain.

**In practice:** a model is selective. It omits more than it includes. The art is choosing what to include for the problems at hand. A model is not "the truth about the domain" — it is a useful simplification for a specific purpose.

### ubiquitous language

A language structured around the domain model and used by all team members within a bounded context to connect all the activities of the team with the software.

**In practice:** the same words in domain expert conversations, in code, in tests, in documentation, in user stories. If domain experts call something a "Rendezvous", the class is `Rendezvous` — not `Meeting`, not `Appointment`. When the language changes, the model changes; when the model changes, the language changes. They are inseparable.

### context

The setting in which a word or statement appears that determines its meaning. Statements about a model can only be understood in a context.

**In practice:** the same word can mean different things in different parts of a system. A "Customer" in Sales is different from a "Customer" in Billing is different from a "Customer" in Support. Without explicit context, statements about the model are ambiguous.

### bounded context

A description of a boundary (typically a subsystem, or the work of a particular team) within which a particular model is defined and applicable.

**In practice:** a Bounded Context is where one consistent model lives, with one ubiquitous language, with one set of definitions for each term. Outside the boundary, the same word may mean something else, and that is fine — but the translation must be explicit.

## Context relationship terms

These describe how Bounded Contexts relate to each other in a Context Map.

### upstream / downstream

A relationship between two groups in which the upstream group's actions affect project success of the downstream group, but the actions of the downstream do not significantly affect projects upstream.

**In practice:** the upstream team can ignore the downstream team and still succeed. The downstream team is at the mercy of upstream decisions. Most legacy integrations are upstream/downstream relationships. (Analogy: if two cities are on a river, the upstream city's pollution affects the downstream city, not vice versa.)

### mutually dependent

A situation in which two software development projects in separate contexts must both be delivered in order for either to be considered a success.

**In practice:** if either fails, both fail. This is the precondition for a *Partnership* relationship. Note: this includes asymmetric dependencies where one system has no value without the other, even if the technical dependencies run only one way.

### free

A software development context in which the direction, success or failure of development work in other contexts has little effect on delivery.

**In practice:** the team can work without coordinating with others. This is the precondition for *Separate Ways*. Many domains have free contexts that teams artificially couple by sharing databases or models — undo that.

## On using these terms

When the user says any of these words, do not assume they mean what Evans means. Check. The most common drift:

- "Domain" used to mean "module" or "subsystem" — that's not domain, that's a piece of the software.
- "Model" used to mean "data model" or "database schema" — Evans' model is conceptual, not structural.
- "Ubiquitous language" used to mean "we have a glossary somewhere" — no. Ubiquitous means in speech, in code, in tests, in writing, used by everyone, every day.
- "Context" used loosely as English "context" — in DDD, Context is technical. Always ask: bounded by what? Whose model? Which team?
- "Bounded context" used to mean "microservice" — sometimes they align, often they don't. A bounded context is a semantic boundary, not a deployment boundary.

When you find drift, name it. Ground the term. Then continue.
