# ddd — Domain-Driven Design skill for Claude Code

A skill that teaches Claude to apply Domain-Driven Design rigorously — and to push back when patterns are misapplied or terminology drifts. Based on Eric Evans' *Domain-Driven Design* (2004) and the 2015 *DDD Reference*.

## What it does

The skill loads automatically whenever a conversation touches domain modeling, bounded contexts, aggregates, entities, value objects, services, repositories, domain events, context mapping, or any non-trivial business-logic design — even if "DDD" is not mentioned explicitly.

It is a **guardrail**, not a yes-machine:

- Enforces *strategic before tactical* — subdomains, bounded contexts, ubiquitous language, context map, distillation come before Entities/Aggregates.
- Nails down terminology so terms don't drift across a conversation.
- Pushes back when a proposed model conflicts with DDD discipline, instead of agreeing reflexively.

## Install

Via the [`claude-bauchladen`](https://github.com/Superheld/claude-bauchladen) marketplace:

```bash
/plugin marketplace add Superheld/claude-bauchladen
/plugin install ddd@claude-bauchladen
```

## Layout

```
skills/ddd/
├── SKILL.md                    # entry point, loaded by Claude on trigger
└── references/
    ├── building-blocks.md      # Entities, Value Objects, Aggregates, Services, ...
    ├── glossary.md             # terminology
    ├── patterns.md             # tactical and strategic patterns
    └── strategic-design.md     # bounded contexts, context mapping, distillation
```

## License

MIT
