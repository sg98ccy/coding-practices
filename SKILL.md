---
name: coding-practices
description: Guide implementation, refactoring, and code review across programming languages, especially decisions about separation of concerns, single sources of truth, module interfaces, state ownership, and reliable behavior. Adapt the guidance to the existing project and the change's scope rather than imposing an architecture or toolchain.
---

# Coding Practices

Make code easier to understand and change correctly. Give particular attention to **separation of concerns** and **single source of truth**, then use the other principles where they help the actual task.

These are decision guides, not a compulsory checklist. Work with the project's language, conventions, constraints, and existing specialist guidance. A small change can remain small; a useful design improvement does not justify an unrelated rewrite. No particular framework, directory layout, file length, abstraction count, test percentage, or development tool is prescribed.

## Separate responsibilities that need to change independently

Group code by what it does and why it changes. Where useful, distinguish presentation, application rules, orchestration, and external access such as files, databases, or network services. Keep related behavior together, and expose a boundary that callers can understand without knowing the implementation.

Extract a responsibility when that makes a real task easier: understanding it, changing it independently, testing its rules, or using a different external implementation. Moving code to another file without reducing dependencies is not enough. Small, cohesive responsibilities can share a file; splitting every function or adding layers to a short script can make the design harder to follow.

For example, a substantive pricing rule used by a screen and an export can live independently of UI formatting and file writing. A short script that reads, transforms, and prints a few values may need only clearly named functions in one file.

Source: [Microsoft architectural principles: separation of concerns and encapsulation](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/architectural-principles).

## Give each concept an authoritative owner

Identify the authority for a rule, data model, configuration value, or piece of state within its relevant scope. Prefer having consumers call, import, derive, or generate from that authority rather than maintain independent definitions that must agree.

- Share behavior when it expresses the same concept and should evolve together. Similar-looking code or equal-valued constants can represent different concepts; keep them independent when sharing would couple unrelated changes.
- Derive a value from existing authoritative data when practical. For example, compute a total from its inputs rather than maintain another editable total that needs synchronising.
- Generated artifacts, caches, deployment copies, and installed skill copies can be legitimate. Make their authority and regeneration or refresh relationship clear when it matters; they are not separate places to author the same rule.
- A single source of truth does not imply one giant module, a global mutable store, or one owner for every kind of state. Independent domains and request scopes can have independent owners.
- Reusing a validation schema does not remove the need to enforce validation and authorization at the relevant trust boundary. A browser check cannot substitute for a server check.

Sources: [Microsoft on DRY and coincidental repetition](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/architectural-principles#dont-repeat-yourself-dry), [React on avoiding redundant and duplicated state](https://react.dev/learn/choosing-the-state-structure). The React examples illustrate state ownership; they do not prescribe React for other projects.

## Other useful design lenses

Choose the rows relevant to the work. They are not separate tasks to perform on every change.

| Principle | Guidance and tradeoff |
| --- | --- |
| Simplicity and proportionate abstraction | Solve the current problem with a design the next maintainer can follow. Introduce an abstraction for a concrete benefit, such as shared semantics or an important boundary. Leave speculative flexibility until its requirements are understood. A useful existing abstraction can be reused without inventing another. |
| Mature libraries where suitable | Before implementing substantial functionality, check whether the project's existing dependencies or an established, well-maintained library already meet the need. Prefer a suitable library so the project can benefit from maintainers' accumulated fixes, handling of edge cases, and incremental improvements. Weigh integration and dependency costs; a small or specialized need may still suit simple local code. |
| Clear contracts and limited coupling | Make important inputs, outputs, assumptions, and failure behavior understandable. Keep implementation details behind the boundary that owns them. Parameters or a small interface can make dependencies explicit; not every function needs an interface, dependency-injection framework, or public API. |
| State and dependency ownership | Make it clear who may change a value and which collaborators an operation needs. Keep state local to the useful scope. Prefer derived values where appropriate; when duplication or synchronisation is necessary, identify the authority and update relationship. |
| Types and validation at boundaries | Use the language's types to express useful invariants, and parse or validate external data where it enters the system. Static types and type assertions do not establish the validity of runtime input. Match validation to the data and trust boundary; avoid repeatedly revalidating trusted internal values without a reason. |
| Reliable failure and cleanup | Consider how an operation leaves state when it fails, is cancelled, or finishes after a newer operation. Give resources such as streams, listeners, subscriptions, and temporary state an owner and appropriate cleanup. Use retries, cancellation, rollback, or idempotency where their failure scenario warrants them, rather than adding them everywhere. |
| Meaningful, proportionate verification | Verify behavior and important invariants at the most useful level. For significant logic, regressions, or risky boundaries, choose tests that would catch a plausible failure rather than reproduce the implementation. Minor prose or trivial edits may need only inspection or an existing check. Use configured tools; no new test harness or arbitrary coverage target is required. |
| Readability and useful documentation | Prefer names and control flow that expose intent. Explain consequential reasons, assumptions, units, or constraints that the code cannot make clear. Update affected usage or contract documentation when behavior changes; avoid comments that just restate obvious operations. |

For performance work, identify the relevant cost before adding caching, concurrency, or more complicated structures. Use measurements when they would resolve a meaningful uncertainty; a straightforward efficiency improvement does not require a benchmark project.

Sources: [Google's code-review guidance on design, complexity, useful tests, naming, and documentation](https://google.github.io/eng-practices/review/reviewer/looking-for.html), [Microsoft on encapsulation and explicit dependencies](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/architectural-principles), [OWASP input-validation guidance](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html).

## Apply with judgement

- Read enough surrounding code to understand the current ownership and conventions before proposing a new pattern. Existing framework or domain guidance can supply details this general skill intentionally leaves open.
- Weigh the maintenance benefit against extra indirection, coordination, and migration effort. Temporary duplication or a local implementation can be sensible when the alternatives introduce premature coupling.
- Keep the requested behavior and scope central. Preserve unrelated behavior during a refactor, and distinguish an intentional behavior change from structural cleanup.
- When a consequential tradeoff is involved, explain the relevant choice briefly. Ordinary changes do not need a design document or a report covering every principle.
- State what was actually verified and any material limitation. Successful parsing, compilation, or an isolated test does not by itself establish all runtime, visual, or integration behavior.
