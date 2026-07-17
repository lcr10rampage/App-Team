# Session Skills — /app-mind and /remember

These two skills form a learning loop. `/app-mind` loads what the system knows at the start of a session. `/remember` improves what the system knows at the end of one.

---

## /app-mind

**When to run:** At the very start of any app-building session.

### What it does

Reads all `.md` files in the project recursively and ingests the knowledge as a baseline for the entire session. Every build prompt from that point forward uses this knowledge — the agent never starts cold.

### Behavior

1. Find all `.md` files in the project directory and subdirectories
2. Read and extract from each file:
   - Proven patterns (prioritize ★★★★★ confidence)
   - Architecture decisions and why they were made
   - Stack conventions and required setup
   - UI/UX principles
   - Common mistakes and how to avoid them
   - Never-do-again rules
3. Output a brief summary:
   - Files ingested
   - Top patterns and rules (5–10)
   - Stack in one line
   - "Ready to build."
4. For every subsequent prompt in the session: build using this knowledge as the baseline. Never violate a proven pattern without explicitly flagging it.

### Why it matters

Without it: the agent re-learns from scratch each prompt, repeats past mistakes, makes inconsistent decisions.

With it: features get built right the first time because the agent already knows what works.

---

## /remember

**When to run:** At the end of a session, after the work is done.

### What it does

Reviews the full session — every prompt, every response — judges what worked and what didn't, then updates the `.md` files so the next session's `/app-mind` loads better context.

### Judgment criteria (from engineering-knowledge-prompt.md)

For each feature or prompt exchange, determine:

| Result | Action |
|---|---|
| Worked first try | Add to Proven Patterns at ★★★★★ |
| Took multiple attempts | Add to Lessons Learned with root cause |
| Single correction needed | Add to Common Mistakes |
| Major redesign required | Add to Never Do Again |
| Same mistake repeated | Escalate existing Common Mistake entry |
| New UI/UX idea emerged | Add to UX Decisions |
| Architecture insight gained | Add to Architecture Decisions |
| Prompt took too many rounds | Add to Prompt Effectiveness with count |

### Behavior

1. Scan the full session history
2. Apply judgment criteria to each significant exchange
3. Write new entries to the correct `.md` file:
   - `EngineeringMemory.md` — patterns, mistakes, design principles, architecture, UX
   - `animation-retrospective.md` (or equivalent) — animation/feature-specific retrospectives
   - `app-system-guide.md` — update blueprints if a new system was built
4. For each entry, always include: what happened, why it worked or failed, what to do instead, confidence level
5. Remove or downgrade entries this session proved wrong
6. Report: every file changed, every entry added/updated, and the session's single biggest lesson

### Why it matters

Each session should make the next one require fewer prompts. `/remember` is the mechanism that makes that happen. The quality of `EngineeringMemory.md` is as important as the quality of the code.

---

## The loop

```
Start of session:  /app-mind   → loads accumulated knowledge
Build the app:     [prompts]   → agent uses knowledge as baseline
End of session:    /remember   → improves the knowledge base
Next session:      /app-mind   → loads even better knowledge
```

The system gets smarter every session.
