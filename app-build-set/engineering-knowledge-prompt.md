# Engineering Knowledge Capture System — Master Prompt

This is the source prompt that defines how `EngineeringMemory.md` is maintained. Save this file and include it in future projects so the knowledge capture system is consistent across all work.

---

Your job is not only to build software, but also to learn from every project so future projects require fewer prompts, fewer revisions, and produce higher-quality results.

Throughout development, continuously analyze how the project evolves and maintain an existing markdown file called `EngineeringMemory.md`.

This file is **not** project documentation. It is a reusable engineering knowledge base that captures lessons that should apply to future projects whenever possible.

After every completed feature, major bug fix, refactor, or architectural change, compare the original implementation with the final implementation and determine:

* What worked correctly on the first attempt.
* What required multiple iterations.
* What only required a single correction.
* What required major redesigns.
* What mistakes were repeated.
* What architectural decisions made later work easier.
* What architectural decisions made later work harder.
* Which prompt changes had the biggest impact.
* Which systems integrated well together.
* Which systems caused friction.
* What should always be done in future projects.
* What should never be done again.

Do not simply record what happened. Explain **why** something succeeded or failed so the lesson is reusable.

Maintain the markdown file using sections similar to these:

# Proven Patterns

Record implementations that worked correctly without modification.

Include:

* Pattern
* Why it worked
* Where it applies
* Confidence level

---

# Lessons Learned

For significant features record:

* Initial approach
* Problems encountered
* Changes made
* Final solution
* Engineering lesson

Focus on reasoning rather than logic.

---

# Common Mistakes

Whenever the same issue appears more than once, record:

* Mistake
* Symptoms
* Root cause
* Correct pattern
* Prevention

---

# Design Principles

Extract higher-level engineering principles instead of project-specific details.

Example:

Instead of:

"The sidebar should be 280px."

Record:

"Navigation components should remain independent of business logic."

---

# Architecture Decisions

Whenever an architectural decision proves beneficial, explain:

* Decision
* Alternatives considered
* Why this approach won
* Tradeoffs
* Reusability

---

# Prompt Effectiveness

Track how efficiently features were completed.

For each major feature record:

* Number of prompts required
* Whether requirements were initially clear
* Biggest source of iteration
* Final confidence
* Reusable (Yes/No)

The goal is to identify areas where prompting can be improved.

---

# Performance Notes

Record optimization techniques that consistently improve speed, maintainability, or simplicity.

---

# UX Decisions

Record interface decisions that proved successful along with the reasoning behind them.

---

# Reusable Components

Whenever a component becomes stable and reusable, document:

* Purpose
* Why it is reusable
* Dependencies
* Best practices
* Known limitations

---

# Never Do Again

Whenever an approach causes unnecessary complexity, technical debt, or repeated revisions, record:

* What was done
* Why it failed
* Better alternative
* Prevention strategy

These lessons should receive especially high priority.

---

# Future Improvements

Record ideas discovered during development that should be explored later without interrupting the current work.

---

For every lesson, assign a confidence level:

★★★★★ Proven repeatedly

★★★★☆ Worked well multiple times

★★★☆☆ Worked once but should be validated

★★☆☆☆ Experimental

★☆☆☆☆ Hypothesis only

Always prefer high-confidence patterns when generating future code.

As development progresses, periodically review the entire EngineeringMemory.md file to remove duplicate lessons, merge related concepts, strengthen generalized engineering principles, and improve clarity. Avoid filling the document with project-specific implementation details unless they teach a reusable lesson.

Your objective is to create a continuously improving engineering handbook that allows future projects to start from accumulated experience instead of repeating previous mistakes. The quality of this knowledge base is just as important as the quality of the code itself.
