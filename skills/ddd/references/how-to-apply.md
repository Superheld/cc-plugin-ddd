# How to Apply DDD

A practical, Evans-centric guide for someone starting a DDD effort. The aim is to keep this as basic as Evans himself does in Part I of the 2004 book: not a recipe, but the small set of practices that make the rest of DDD work. Everything below is from Evans (2004 Part I, 2015 Reference Chapter 3). One single, time-tested practitioner technique — Event Storming — gets a brief mention at the end because it has earned its place over a decade of consistent use; nothing else is suggested.

Source: Evans 2004 chapters 1–3 ("Crunching Knowledge", "Communication and the Use of Language", "Binding Model and Implementation") and the 2015 *DDD Reference* opening section. Paraphrased throughout; consult the original when wording matters.

---

## Before you start: is DDD worth the cost?

DDD is overhead. Knowledge crunching with domain experts, maintaining a Ubiquitous Language, refactoring toward deeper insight — all of this costs real time and attention. The payoff comes when the domain is complex enough that a casual model would not survive contact with the business.

Reach for DDD when:

- The domain has rules, vocabulary, and edge cases that domain experts care about and developers do not yet understand.
- The system is expected to live and evolve for years.
- Getting the model wrong has a real cost (regulatory, financial, operational).

Do not reach for DDD when:

- The domain is essentially CRUD against a known schema. A contact list, a simple form-to-database app. SKILL.md says this directly: DDD on a simple domain produces ceremony without insight.
- The lifespan is short (a one-off tool, a prototype).
- No domain expert is available for sustained collaboration. Without domain experts, knowledge crunching cannot happen, and DDD becomes architecture cosplay.

If you decide DDD is right, accept the practices below as part of the work — not optional embellishments.

---

## 1. Knowledge crunching with domain experts

The first practice and the one everything else depends on. Evans' framing: *"Effective domain modelers are knowledge crunchers."* The work is turning a torrent of chaotic domain information — interviews, documents, examples, edge cases — into a model the team can act on.

What it looks like in practice:

- **Sit with domain experts regularly.** Not "gather requirements" — *model together*. Developers sketch, ask, listen, sketch again. Domain experts correct, elaborate, and recognise their own thinking in the model.
- **Use concrete examples.** Abstract questions ("how does pricing work?") get vague answers. Concrete examples ("this customer ordered five of these on a Friday in March — what is the price?") expose real rules.
- **Refine as you go.** The first sketches will be wrong. That is the point. The model improves because the team keeps learning.
- **No one person owns the model.** Architects who model in isolation produce models that do not survive contact with code; developers who model without experts produce models that do not survive contact with the business.

Continuous practice. Knowledge crunching does not end after a kickoff workshop — it continues for the life of the project, because the domain itself keeps revealing more.

---

## 2. Build the Ubiquitous Language

The shared vocabulary, grounded in the model, used by *everyone* on the team and in *all* artefacts — conversations, code, documentation, tests.

Concretely:

- **Use the same words everywhere.** If domain experts say "policy" and developers say "contract", one of those words is wrong. Pick the domain expert's word — they are closer to the domain truth — and use it in the code, the database, the diagrams, the meetings.
- **Use the language out loud.** Evans calls this *modelling out loud*: try saying the rule you are encoding. If it does not flow as a natural sentence, the model is probably wrong.
- **Treat vocabulary disagreements as model bugs.** When two domain experts use the same word for different things, you have just discovered a Bounded Context boundary. Do not paper over the disagreement; surface it. The resolution will either rename one of the concepts, or split them into two contexts with different vocabularies.
- **No translation layer between domain experts and developers.** Evans is explicit: separate vocabularies for "business analysts" and "developers" of the same model is a sign that no model is actually being shared.

The Ubiquitous Language is bounded by the Bounded Context. Inside one BC, one meaning. Across BCs, the same word may mean different things, and that is fine — the Context Map is where the differences are documented (see `strategic-design.md`).

---

## 3. Bind model and implementation

A model is only real if it is in the code. Evans calls this **Model-Driven Design**: the code is an *expression* of the model, not a translation of it.

What this means in practice:

- **Class names, method names, module names come from the Ubiquitous Language.** If a domain concept is called "settlement", there is a `Settlement` class. If domain experts speak of "reconciling" two accounts, there is a `reconcile` method.
- **The code says what it does in domain terms.** When the model changes, the code changes. When the code changes, the model changes. They are not independent.
- **No analyst/developer wall.** Evans is direct: separating "people who model" from "people who code" hollows out the model. Developers must touch the model; modellers must touch the code. The hands-on modellers principle.

If a domain concept is hard to express in code, that is information about the model, not about the language or the developer. The model probably needs work.

---

## 4. The first model is wrong

DDD is a practice of *refactoring toward deeper insight*. Evans returns to this idea repeatedly because it is uncomfortable: every model is provisional. The team's understanding will deepen, and when it does, the model must change to match.

What this looks like:

- **Expect to rename things.** Words that seemed right in week one will turn out to be wrong by week six because the domain expert will say something offhand that reveals a deeper concept. Rename, in code and in conversation.
- **Expect to redraw Aggregate boundaries.** What looked like one invariant cluster will turn out to be two; what looked like two will sometimes turn out to be one.
- **Expect to split or merge Bounded Contexts.** The boundaries you draw early will move as you learn where the language actually changes.

SKILL.md is clear about the distinction: *provisional* is not the same as *fuzzy*. The model is precise at any given moment. It is *willing to change*, not *vague about the present*. Refactoring toward deeper insight requires the current model to be sharp enough that the next insight is recognisable as a difference.

---

## 5. Sequence: strategic before tactical

This is the single most violated rule in DDD practice, and SKILL.md gives it the most space for that reason. The order:

1. **Identify the domain and its subdomains.** Where is the business? Which subdomains are Core, Generic, Supporting? See `glossary.md`.
2. **Find the Bounded Context boundaries.** Where does the same word mean different things? Where does one model end and another begin? See `glossary.md` and `strategic-design.md`.
3. **Build the Ubiquitous Language inside each Bounded Context.** One vocabulary per context.
4. **Draw the Context Map.** Name every relationship between contexts with a Context Mapping pattern (Partnership, Shared Kernel, Customer/Supplier, Conformist, Anticorruption Layer, Open-host Service, Published Language, Separate Ways). See `strategic-design.md`.
5. **Distil the Core.** Where does the most skilled effort go? What is generic and can be bought? See `strategic-design.md`.
6. **Only now: tactical work.** Inside the Core (and inside the supporting contexts), apply Entities, Value Objects, Aggregates, and the rest. See `building-blocks.md`.

Jumping to step 6 without steps 1–5 produces a beautifully patterned single-context model that collapses the first time the system has to integrate with anything else. Do the strategic work first.

---

## 6. One established practitioner technique: Event Storming

Outside Evans' own writing, one workshop technique has earned its place in DDD practice through a decade of consistent use: **Event Storming**, developed by Alberto Brandolini and documented in his book *Introducing EventStorming* (2013, refined through 2017+).

The format: a long roll of paper on the wall, orange sticky notes for Domain Events (past-tense things that happened), and a structured progression — *Big Picture* event storming (all the events in the business), *Process Modelling* event storming (how a particular process flows), *Software Design* event storming (where Aggregates and policies emerge). Cross-functional participants — domain experts, developers, designers, ops — work together at the wall.

Why it matters here: Event Storming is the most direct, time-tested way to do the *Knowledge Crunching* practice from section 1, at the strategic level. It surfaces the events the business cares about, exposes vocabulary disagreements, and routinely reveals Bounded Context boundaries that were not visible from conversation alone.

This is a brief pointer, not a tutorial. If you want to use Event Storming, read Brandolini's book; the format is simple but the facilitation is not. Other workshop formats and canvases exist in the DDD community, but they are not included here because they are newer, less consistently described, or still finding their place — and the user instruction for this file is "only what has proven itself".

---

## Cross-references

For terms (subdomain, Bounded Context, Ubiquitous Language, etc.), see `glossary.md`. For the strategic patterns (Context Mapping, Distillation, Large-scale Structure), see `strategic-design.md`. For the tactical patterns (Entities, Value Objects, Aggregates, etc.), see `building-blocks.md`.
