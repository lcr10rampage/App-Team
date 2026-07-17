# Prompt Lessons

> What took multiple prompts to get right vs. what worked immediately — so future agents phrase instructions well the first time. This mirrors the *Prompt Effectiveness* section in `app-build-set/engineering-knowledge-prompt.md`. Manager-approved.

## Entry format
```
### <short label>
- Original instruction:
- Result it produced:
- Problem:
- Improved instruction:
- Final outcome:
- Lesson for future agents:
```

## Seeded lesson (carried from prior work, high confidence)
### Force action, don't narrate
- Original instruction: "Only call a manager when the user's question requires real data."
- Result: the agent described what it would do instead of doing it.
- Problem: models default to narrating intent before acting.
- Improved instruction: explicit rules — "DO IT IMMEDIATELY using tools. Never say 'I will' or 'I can'. Just act."
- Lesson: for agentic behavior, state the action directive explicitly and forbid hedging phrases.

## Entries
(Add team-surfaced prompt lessons below.)
