# Family 10 — Error handling

Whether failures are handled deliberately, consistently and at the right level.

Read `../conventions.md` first: severity rubric, hotspot adjustment, ownership rule.

---

### ERR-01 — Swallowed exceptions

- **Aliases:** empty catch, silent failure, catch-and-log-and-continue
- **Definition:** Errors are caught and discarded — empty catch blocks, catch-and-log with no further action, or returning a default that hides the failure.
- **Why it hurts:** Failures surface later as corrupt data or wrong behaviour far from the cause; production incidents become hard to diagnose.
- **Detection heuristics:**
  - Empty `catch` / `except: pass` / `.catch(() => {})`.
  - Catch blocks that only log, then continue as if the operation succeeded.
  - Catch blocks returning `null`, empty collections or default values without the caller being able to tell.
  - Ignored error return values in languages with explicit errors (`_ = err` in Go, unchecked `Result`).
- **False positives:**
  - Explicitly documented best-effort operations (e.g. sending a non-critical metric), ideally with a comment.
  - Expected exceptions used as a check where no alternative API exists (`NumberFormatException` when parsing user input) — if handled meaningfully.
- **Default severity:** High
- **Source:** Bloch, *Effective Java* — don't ignore exceptions; Nygard, *Release It!* (2007; 2nd ed. 2018).

---

### ERR-02 — Exceptions as control flow

- **Aliases:** exception-driven logic
- **Definition:** Exceptions are thrown and caught to implement ordinary, expected outcomes (not found, invalid choice, loop exit) rather than exceptional failures.
- **Why it hurts:** The normal flow is hidden in catch blocks; performance can suffer; real errors get mixed with expected outcomes and caught by the same handlers.
- **Detection heuristics:**
  - Try/catch whose catch branch is a normal business path (`catch (NotFoundException) { createNew(); }`).
  - Custom exceptions thrown and caught within the same method or class.
  - Exceptions used to break out of loops or recursion.
- **False positives:**
  - Languages/idioms where exceptions are the conventional mechanism (Python's EAFP, `StopIteration`).
  - Framework conventions mapping exceptions to HTTP responses at the edge.
- **Default severity:** Medium
- **Source:** Bloch, *Effective Java* — use exceptions only for exceptional conditions; Hunt & Thomas, *The Pragmatic Programmer*.

---

### ERR-03 — Inconsistent error strategy

- **Aliases:** mixed error conventions
- **Definition:** The same kind of failure is reported in different ways across the codebase: `null` here, an exception there, a result type or error code elsewhere, a boolean success flag in a fourth place.
- **Why it hurts:** Callers can't rely on a convention; each call site needs to check how this particular function fails; failures slip through when the wrong check is used.
- **Detection heuristics:**
  - Similar operations (e.g. all repository `find` methods, all external client calls) using different failure signalling.
  - A mix of `Optional`/`Maybe`, `null` and exceptions for "not found" across modules.
  - Result/Either types used in some modules while others throw for the same category of error.
- **False positives:**
  - Different strategies for genuinely different categories (expected business outcomes as results, bugs/infrastructure failures as exceptions) when applied consistently.
  - Third-party APIs dictating their own conventions at the boundary.
- **Default severity:** Medium
- **Source:** Martin, *Clean Code* — error handling; Wlaschin, *Railway Oriented Programming* (2014).

---

### ERR-04 — Low-level errors leaking across layers

- **Aliases:** missing exception translation, infrastructure errors in the domain
- **Definition:** Errors specific to an implementation (SQL exceptions, HTTP client errors, SDK exceptions) propagate up into domain or presentation code, which then catches or inspects them.
- **Why it hurts:** Higher layers become coupled to the infrastructure choice; changing a library changes error handling everywhere; users may see internal details.
- **Detection heuristics:**
  - Domain or application code catching vendor/driver exception types.
  - Controllers inspecting database error codes or messages.
  - Method signatures in domain interfaces declaring infrastructure exceptions.
  - Raw exception messages returned to API clients.
- **False positives:**
  - Infrastructure adapters themselves (they should handle these).
  - Generic top-level handlers that log any exception.
- **Default severity:** Medium
- **Source:** Bloch, *Effective Java* — throw exceptions appropriate to the abstraction (exception translation).

---

### ERR-05 — Catch-all at the wrong level

- **Aliases:** Pokémon exception handling, broad catch
- **Definition:** Code catches the broadest error type (`Exception`, `Throwable`, bare `except`) deep inside the call stack, where it cannot meaningfully recover, instead of at a boundary where it can be reported or retried.
- **Why it hurts:** Programming errors and unexpected failures are hidden or mishandled alongside expected ones; retries or fallbacks fire for bugs.
- **Detection heuristics:**
  - `catch (Exception e)` / `catch (Throwable t)` / bare `except:` in domain or library code rather than at entry points.
  - Broad catches wrapping large blocks with a generic fallback.
  - Catching broad types and then checking `instanceof` inside.
- **False positives:**
  - Top-level boundaries: request handlers, message consumers, job runners, main loops — the right place for a catch-all.
  - Plugin hosts isolating third-party code.
- **Default severity:** Medium
- **Source:** Bloch, *Effective Java*; Nygard, *Release It!* — failure handling at integration points.

---

## Parked

| Name | Definition | Reason parked |
|---|---|---|
| Concurrency hazards | Races, deadlocks, unsafe shared state | Needs runtime information |

## See also (owned elsewhere — do not report here)

- Leaky abstractions (vendor types in interfaces, other than errors) → ABS-06
- Over-specified error assertions in tests → TST-01
