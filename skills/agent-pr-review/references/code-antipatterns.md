# AI coding anti-patterns — catalog

Source: Yehor Sedinin's `declutter` skill (`anti-patterns.md`), shared in the
[August 18, 2026 team thread](https://simulmedia.slack.com/archives/C022S15NCJ0/p1787057765549379).
The catalog below is his file verbatim. Extend it with new categories or
examples as a team; keep additions in the same before → after shape and mark
them as team additions so the original author is not credited with them.

## How to use this catalog in a PR review

The `declutter` skill finds and *applies* these patterns. A PR review only
*reports* them, and only when they matter:

- Inspect added/changed code by default; follow unchanged code only for context
  or a concrete regression. Review is read-only.
- A catalog match is not a finding. A narrating comment, redundant `else` or
  one-use wrapper is a cleanup preference, not a PR blocker. Report a match only
  when it hides or causes a real defect — for example a swallow-and-continue
  `except` that turns a failed computation into an empty report (category 3),
  or a removed guard at a trust boundary (category 2, "Keep").
- Before proposing that a guard is over-defensive, trace the callers and runtime
  inputs. Type annotations alone do not establish runtime guarantees.
- Removing a catch can change exception type, message, side effects or fallback
  behavior; do not present that as behavior-preserving without evidence.
- If the user asks for a cleanup pass, run `declutter` (or follow its
  report-then-apply workflow) as a separate step from bug review.

---

Each entry: what it is, why it's slop, before → after. Examples are Python and TypeScript; apply the idea in any language.

---

## 1. Narrating comments

Comments that restate what the code already says. They add noise and rot when the code changes.

```python
# Before
count = count + 1  # increment count
users = []  # initialize empty list of users
def get_user(id):  # function to get a user by id
    ...
```
```python
# After
count += 1
users = []
def get_user(id):
    ...
```

Also remove: section banners (`# ---- helpers ----`) that just label one function, docstrings that only repeat the signature, and `# TODO` left by the model for work it already did.

**Keep:** comments explaining a non-obvious *why*, a workaround for a known bug/quirk, a link to a spec, or a surprising business rule.

---

## 2. Over-defensive guards

Null/None/type checks for conditions that cannot occur given the code's own contract.

```python
# Before
def total(items: list[int]) -> int:
    if items is None:          # items is typed list and always passed
        return 0
    if not isinstance(items, list):
        raise TypeError("expected list")
    return sum(items)
```
```python
# After
def total(items: list[int]) -> int:
    return sum(items)
```

```typescript
// Before
function fullName(u: User): string {
  if (!u) return "";              // u is non-optional in the type
  return `${u.first} ${u.last}`;
}
```
```typescript
// After
function fullName(u: User): string {
  return `${u.first} ${u.last}`;
}
```

**Keep:** validation at trust boundaries — user input, network/API responses, file/env parsing, public library entry points. There the check is real.

---

## 3. Redundant / impossible error handling

```python
# Before — catches something that can't fail, then does nothing useful
try:
    value = config["key"]
except KeyError:
    raise KeyError("key not found")   # re-raises the same error

# Before — swallow-and-continue that hides bugs
try:
    result = compute()
except Exception:
    result = None
```
```python
# After
value = config["key"]

result = compute()
```

Also remove: `try/except` wrapping a single infallible statement, broad `except Exception: pass`, and catch-log-rethrow that adds no context.

**Keep:** error handling that recovers, retries, adds context, or cleans up resources.

---

## 4. Redundant guards & dead branches

```python
# Before
if user is not None:
    if user.is_active:        # nested, collapsible
        send(user)

# Before — else after return
if x > 0:
    return "pos"
else:
    return "non-pos"
```
```python
# After
if user is not None and user.is_active:
    send(user)

if x > 0:
    return "pos"
return "non-pos"
```

Also: `len(items) == 0` → `not items`; `if flag == True` → `if flag`; default values reassigned right before use; the same condition checked twice in one flow.

---

## 5. Single-use abstractions & over-engineering

Indirection introduced for a single caller or a hypothetical future.

```python
# Before — factory + interface for one concrete type
class GreeterFactory:
    @staticmethod
    def create() -> "Greeter":
        return Greeter()

# Before — config object with one field used once
DEFAULTS = {"timeout": 30}
def fetch(url): requests.get(url, timeout=DEFAULTS["timeout"])
```
```python
# After
def fetch(url): requests.get(url, timeout=30)
```

Remove: wrapper functions that only call one other function, base classes with one subclass, parameters that are always passed the same value, "for future extensibility" hooks nothing uses.

**Keep:** abstraction with two or more real callers, or a seam the codebase already depends on.

---

## 6. Verbose constructs

```python
# Before
result = []
for x in items:
    result.append(x * 2)

# Before
def is_even(n):
    if n % 2 == 0:
        return True
    else:
        return False
```
```python
# After
result = [x * 2 for x in items]

def is_even(n):
    return n % 2 == 0
```

Apply only when the shorter form is at least as readable. Don't compress a clear loop into an unreadable nested comprehension.

---

## Keep — not slop (do NOT remove)

- Input validation at trust boundaries (user input, I/O, network, env, public APIs).
- Comments/docstrings that explain *why*, document a public contract, cite a spec, or warn about a footgun.
- Error handling that recovers, retries, adds context, or releases resources.
- Edge-case handling for inputs that genuinely occur.
- Anything whose removal changes behavior, even subtly. When in doubt, keep it and flag it.
