# Every bug fix ships with the test that would have caught it

> Canonical. Extracted from `alcovio/AGENTS.md` on 2026-08-18 and generalised.
> Do not fork this into a project file.

A fix without a test is a fix that comes back. This is not a nice-to-have step at the
end: **the test is part of the fix, and the work is not done without it.**

The quality bar is not "a test exists". It is these six, all learned the hard way in the
week of 2026-08-05.

## 1. The test must FAIL against the old code

If it passes before the fix, it is testing something else and proves nothing. **Check
this, do not assume it.**

## 2. Test the real thing, not a copy of it

A test that re-implements the condition it is checking passes forever while the real
code rots. When the logic is buried inside a big function, EXTRACT it and import it —
the way `seat_height_impossible()` was pulled out of `ingest_cj.py` so
`test_extension_rules.py` could call the actual function.

## 3. Count what the test really covers, and compare it with reality

The title rules were documented as enforced "with NO allowlist" while the check read
twelve literal strings out of 1,006 pages.

Before claiming a rule is enforced, **state how many real pages, rows or products the
check touches**. If you cannot state the number, it is not enforcement. Prefer running
the SAME builder the production path runs, over a snapshot of real data.

## 4. Fixtures hold INPUTS, never finished output

A snapshot of rendered output gets regenerated alongside the change that broke it, so it
records the drift instead of catching it.

## 5. Pin the specific wrong value so it cannot return

Beside the positive assertion, assert the old bad value is gone:

    expect(body).not.toContain("depth is 18 to 24 inches")

That is what stops a well-meaning edit reintroducing it.

## 6. Name the incident in a comment

One or two sentences: what shipped, what it cost, what the rule is now. A bare assertion
gets deleted by the next person who finds it inconvenient; an assertion that explains
itself gets fixed properly.

## Overturning a test deliberately

**When a test encodes a decision you are deliberately overturning, REPLACE the
assertion, never delete it.** Deleting it removes the guard; replacing it moves the
guard to the new rule and leaves a record that the change was intentional. Say so in the
commit and flag it for Ian, because that is him being overruled.

## Cross-language rules

A rule expressed in more than one language needs a test per language **plus one that
asserts the declarations still agree**. Two implementations that "obviously match" is
exactly how the source fingerprint broke.
