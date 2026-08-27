# Code Quality Contract

This is the bundled default contract for the `review-ready` gate. A project may
override it with its own `docs/agents/code-quality.md` (or equivalent); when it
does, that project contract wins and this file is ignored.

## When it applies

Use this contract when writing or refactoring production code, tests,
workflows, agent instructions, request handlers, CLI commands, migrations,
integration adapters, or other code that changes how the system behaves. Apply
it to the seam you are changing. Leave unrelated cleanup for a separate issue
unless the current change cannot be made safely without it.

## Design standard

A review-ready change should make the changed flow easier to understand through
its public shape, not merely make the diff smaller.

- **Narrative entry points**: a handler, workflow, command, or other
  orchestration entry point should read top-to-bottom as the domain flow.
  Parsing, policy calculation, resource composition, persistence details, and
  provider mechanics belong behind named modules owned by those concepts.
- **Deep modules and strong ownership**: a module should hide cohesive behavior
  behind a small interface. Domain types, errors, constants, validation, and
  lifecycle rules live beside the behavior that owns them. Generic runners and
  utility modules own only genuinely generic concepts.
- **Real seams**: introduce an interface or adapter where behavior actually
  varies or where a policy or effect boundary needs replacement in tests. A
  pass-through wrapper that merely renames another interface does not create
  depth. Dependencies should point from orchestration toward owning modules,
  without cycles.
- **Interface-oriented tests**: test observable behavior and safety invariants
  through the same interface callers use. Test internal helpers only when the
  condition cannot be exercised deterministically through that interface.
  Declarative registration or configuration may be tested as source when the
  declaration itself is the only observable seam.
- **Bounded improvement**: deepen the changed seam and keep the scope local.
  Security checks that must happen at effect time remain explicit at the owning
  effect seam; aesthetic extraction must not hide or weaken them.

## Review-ready gate

Apply this review-ready gate before declaring implementation complete:

1. Re-read every changed file top-to-bottom after tests are green.
2. Trace one representative input through the changed flow and verify the entry
   point still tells that story.
3. Apply these four tests:
   - **Caller-knowledge test**: what must each caller know to use the changed
     module correctly, and can that knowledge move behind the interface?
   - **Deletion test**: if the module vanished, would its complexity disappear
     or spread back across callers?
   - **Ownership test**: does every new concept have one clear home beside the
     behavior that owns it?
   - **Test-surface test**: do tests exercise the interface rather than the
     wiring behind it?
4. Perform one dedicated simplification pass inside the changed scope. Remove
   duplicated policy, speculative configurability, pass-through abstractions,
   stale comments, temporary compatibility, and avoidable interface surface.
5. Run every applicable repository verification command and report failures or
   skipped checks honestly. Explain intentional exceptions to this contract in
   the PR description.

Unresolved concrete violations block declaring the work review-ready.

## Non-goals

- This contract is not a numerical size, file-length, function-length,
  cyclomatic-complexity, module-count, or coverage-quota system.
- A large file is not automatically wrong, and splitting a file is not
  automatically an improvement.
- Extraction is valuable only when it increases module depth, ownership,
  locality, or testability at a real seam.
- The contract does not authorize repo-wide cleanup that is unrelated to the
  current change.
