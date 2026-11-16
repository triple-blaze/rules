# Git protocol — who runs git

> Canonical. Extracted from `alcovio/AGENTS.md` on 2026-08-18 and generalised.
> Do not fork this into a project file. If a project needs a variation, state the
> variation in that project's own file and say which clause it overrides.

## FIRST decide which kind of session you are

This rule was originally written for the Cowork device bridge and was then applied
everywhere, including sessions with a perfectly good shell. **Handing Ian a block of
terminal commands he did not ask for is a FAILURE, not compliance.** He is the business
owner, and "run this in your terminal" is the answer he was trying to avoid. Check the
precondition before invoking the handover.

### A) Sessions that CAN run git

Claude Code, or anything else with a working shell. Verify with `git status` plus a
write to `.git/`.

Do the work yourself: stage, commit, verify. Do NOT print a paste-ready block. Do NOT
ask Ian to run anything he could ask you to do instead. This covers everything, not just
git — regenerating a file, running a script, checking CI. **If you catch yourself
writing "run this", stop and run it.**

### B) Sessions that CANNOT run git

A Cowork or Claude session reaching a repo over the device bridge. The bridge leaves
`.git/*.lock` files behind and cannot delete them (`rm` fails with "Operation not
permitted").

Only here does the division of labour apply:

1. **Claude** edits the working tree, then ends the session by printing a
   ready-to-paste terminal block containing: the lock-file cleanup, `git add` of every
   file it touched (**named explicitly, never `git add -A`**), a written commit message,
   `git push`, and the verification commands.
2. **Ian** runs that block.
3. **Claude** then verifies against origin as in (A).

A type-B session that edits a repo and does NOT hand over a commit block is incomplete.
A type-A session that hands over one instead of just committing has misread this rule.

### Commit-block formatting rules

- **No inline `#` comments and no blank lines inside the block.** Ian pastes it as one
  chunk; comments and blank lines break the paste. Explain the steps in prose above or
  below the block, never inside it.
- Ian's shell is **zsh**, where a glob matching nothing aborts the WHOLE command. Never
  write `rm -f .git/*.lock .git/objects/*/tmp_obj_*` — this silently skipped the cleanup
  on 2026-07-29. Use instead:

      rm -f .git/index.lock
      find .git/objects -name 'tmp_obj_*' -delete 2>/dev/null

## The push gate

`git push` is the one exception to "just do it" in a type-A session. Pushing to the
default branch may deploy to a live site, so **confirm before pushing** — once, in a
sentence, not with a wall of commands. That is a review gate on what reaches the public
and it stands whatever the session type.

Projects that deploy on push say so in their own file. Absent that statement, still
confirm.

## Committed is not persisted

There are TWO ways an edit silently reverts, and both have happened repeatedly:

1. **Uncommitted** working-tree edits are discarded by the next `git checkout`,
   `git reset`, or `pull --rebase`. Branch-hopping and rebasing repos lose them without
   warning.
2. **Committed but not pushed** edits are discarded whenever the environment resets to
   or re-clones `origin` — the local commit never existed there. The 2026-07-25 and
   2026-07-27 tracker fixes were both committed yet sat unpushed for days, which is why
   the file kept reappearing at its older *pushed* state.

**The rule: `git add` + `git commit` + `git push` in the SAME session, then verify:**

    git rev-parse HEAD origin/main
    git show origin/main:<path> | grep <marker>

The two hashes must match. A "source of truth" that lives only in the working tree, or
only in an unpushed local commit, WILL revert.

**Committed ≠ persisted. Pushed-and-confirmed-on-origin = persisted.**

## Pull requests in Ian's sessions

This section applies to sessions Ian Ratner runs. It does not apply to other people's sessions on a shared repository.

Open every pull request as a draft while you are still working on it. When the work is done and GitHub shows no conflicts with main, mark it ready for review, which starts the required checks. Wait until every required check on the ready pull request has finished and passed, and only then tell Ian it is ready to merge, with its full link. If a pull request you opened gets a conflict with main, merge main into its branch, resolve the conflict, push, wait for the checks again, and tell Ian in 1 line what you changed. Where resolving it means choosing between 2 versions of wording or behavior, stop and send a Question for Ian to the project management chat. Never change another session's or another person's branch.

Source: Ian Ratner, November 16, 2026.

Give the full link every time you mention a pull request, whether you name it by number or describe it, in any part of a reply or a note.

Before you ask Ian to merge, review or do anything else with a pull request, check its state on GitHub in the same turn: whether it is open, a draft, merged or closed, whether its required checks have passed, and whether it conflicts with main. Ask Ian only about a pull request that is open, ready for review, free of conflicts, and passing every required check. A report from another session or an earlier turn is not a check, because a pull request can merge or change between that report and your reply. Where you cannot reach GitHub in that turn, say so in 1 line and leave the pull request out of the actions you list for Ian.

Source: Ian Ratner, November 16, 2026.
