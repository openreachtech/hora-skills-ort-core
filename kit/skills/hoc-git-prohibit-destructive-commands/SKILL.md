---
name: hoc-git-prohibit-destructive-commands
description: "The git commands this library does not run, because what they destroy cannot be given back — a modified file, an untracked note, an object no ref still names — and what is reached for instead. Use before any command that would clear the working tree, discard a change or delete an object, and whenever something else needs the tree clean before it can run. Destroying what the remote holds belongs to the git push convention."
---

# Git: Prohibit Destructive Commands

**What a repository still holds is not the agent's to destroy.** It was put there by whoever is
working, and an agent that did not write a file cannot read what it is worth. The commands below
take it away with nothing left to recover from, so the work is carried through an operation
rather than cleared before one.

## Nothing that discards uncommitted work is run

**No command is run whose effect is that an uncommitted change or an untracked file is gone, and
there is no exception.** The rule is written as the effect rather than as a list of spellings,
so a command not named here is covered by what it does.

Four were measured, and not one of them asked:

| Command | What went |
| :-- | :-- |
| `git reset --hard <target>` | A modified file and a staged one, both |
| `git checkout -- <path>` | The modification at that path |
| `git restore <path>` | The same |
| `git clean -fd` | Untracked files, and untracked directories whole |

- **`git clean` is barred outright rather than merely avoided, and nothing is given up by that.**
  It is the one of the four with no recoverable half: the others take back a change the
  repository still holds an object for, and this one removes what the repository never held. What
  it is reached for is reached another way — `git stash -u` empties the tree of untracked files
  without destroying them — and the operations said to need a clean tree largely do not: a
  rebase, measured, is not blocked by an untracked file at all. A working practice that never
  runs it loses nothing, which is what makes the bar costless to hold.
- **A command that prints what it removed has still not asked.** `git clean -fd` names each path
  as it goes, into a scrollback nobody is reading at that moment. A line after the fact is not a
  confirmation before it.
- **"The tree is clean" is not a reason to run one.** That is a reading taken at an instant while
  the operation is not, and where it is true the command had nothing to do anyway.
- **Needing a clean tree is not a reason either.** Setting the work aside is what that calls for,
  and it is below.

## Why it is absolute

**The judgement it would take is not one to be made here.** Deciding that a file may go means
knowing what it was for and what losing it would cost — a reading of somebody else's work, taken
in passing, with no way to check the answer and no way back from a wrong one. **Where an act
cannot be undone and the judgement behind it cannot be verified, the act is not taken at all.**
That is what makes this absolute rather than a matter of care: care is what is exercised over
decisions that are yours to get wrong.

**The agent did not make every file in the tree.** Notes, a half-written draft, a file copied in
to compare against, a directory kept outside version control on purpose — a person leaves these
where they are working, and nothing in the tree distinguishes them from the agent's own leavings.

Measured: a directory of notes kept untracked was removed whole by an agent clearing the tree.
Nothing announced it to the person who had put it there, and nothing could bring it back.

**And the loss arrives dressed as tidiness.** What is removed looks like debris — stale, untidy,
in the way — so the removal is reported as housekeeping rather than as a loss, and nobody present
has reason to question it. The person who put the thing there learns of it later, from its
absence. **What makes something worth keeping is rarely visible in it**, and an assessment made
on appearance is precisely the judgement that is not available here.

**"No way back" is literal here**, which is what supplies the premise above. A committed change
survives its own deletion, because the object was written and something still names it. **An untracked
file has no object at all** — nothing in the repository ever held it, so it is in no reflog and
no stash, and what removes it from the filesystem is the end of it. That asymmetry is why the
rule reaches an untracked file at least as hard as a tracked one, where every instinct says the
opposite.

## Setting it aside instead

**`git stash -u` is the preserving form of all four.** It takes the modifications and the
untracked files together, leaves the tree clean for whatever needed it, and gives them back.

```bash
git stash -u
<the operation that needed a clean tree>
git stash pop
```

- **`-u` is part of the command rather than an option to it.** Without it the untracked files
  stay in the tree — which is the half a plain stash does not cover, and the half that has no
  other home.
- **A stash is popped as a merge, not as a replay.** It carries the diff from the commit it was
  taken at, so a pop at some other commit succeeds while that commit still agrees about what the
  stash touched, and conflicts where it does not — leaving the entry in place. Where an operation
  moved the branch away, the pop therefore belongs on whichever ref still holds the commit it was
  taken at, which is the one place nothing can disagree.
  - Measured, popping at a different commit: clean where the edited file stood unchanged at both;
    `UU` where the file's content had moved under the edit; `DU` where the file existed only at
    the commit the stash was taken at. The entry was kept in both failing cases.

## A commit holds what a stash cannot

Where the work has to cross something a stash will not survive — an operation that pops stashes
of its own, a sequence long enough that a conflict on the pop is likely — a commit holds it
instead.

- **It is made on a branch, and never on a trunk.** Which branches are trunks, and why nothing is
  committed directly on them, belong to the git branch convention.
- **Its message is free, and it is folded away before the work is shared.** The git commit
  convention settles the save point made on the premise that it is deleted right away.

## An object, once written, is not deleted

**Staging or stashing is what makes uncommitted work recoverable at all**, because both write it
into the object store. From that moment it exists apart from the file, the index and any branch.
Measured: a staged file deleted from the tree and dropped from the index left its blob intact,
and `git cat-file -p` printed the content back.

**So the object is the thing, and a ref, a reflog entry and a hash somebody printed are three
names for it.** Removing a name costs one way in and the object answers to the others. Removing
the object leaves every name resolving to nothing.

| What it removes | What is left |
| :-- | :-- |
| `git branch -D`, `git reflog expire` | A name. The object stands, reachable by any other |
| `git stash drop`, `git stash clear` | A name — the stash entry. The commit it pointed at stands |
| `git worktree prune` | A name — a worktree's `HEAD` and its reflog, which may be a commit's only one |
| `git gc`, `git prune` | The object, once the expiry has passed over it. Every name for it is then a string that resolves to nothing |
| `git repack -a -d` | The object, at once. No expiry stands between the command and it |

**No command that deletes an unreachable object is run**, and the default grace period is not
what makes one safe. It protects what was lost recently, while the hash still on somebody's
screen is the route for what was lost a while ago — which is the layer the grace period has
stopped covering.

- **A hash printed anywhere is a recovery route.** A tree printed into a terminal keeps every
  hash it showed for as long as that terminal is open, and `git tag <name> <hash>` brings the
  commit back. Measured: it succeeded after the reflog had been expired and every ref to the
  commit deleted, and failed with `Failed to resolve` once the object had been pruned.
- **`git repack -a -d` deletes without the grace period, and `git gc` does not.** Measured on a
  repository whose unreachable commit had already been packed: `git gc` left it, `git repack -d`
  and `git repack -A -d` left it, and `git repack -a -d` removed it outright. `-A` writes an
  unreachable object back out as a loose one, where the expiry still covers it, and it is what
  `git gc` passes; `-a` writes a pack of what is reachable and deletes the old pack whole. The
  two spellings differ by the case of one letter, and one of them is harder than `git gc` is.
- **`git stash clear` takes the last name without printing one.** `git stash drop` names the hash
  of the entry it dropped, so the route above is open at the moment it is needed. `clear` prints
  nothing and removes the stash reflog along with the entries. Measured: the objects survived
  both, and after `clear` nothing short of `git fsck --unreachable` could name them again.
- **This is the rule the rest of this convention stands on.** The asymmetry above holds only
  while the objects are there; against a repository that has been pruned, every case is the
  untracked one.

## Where this stops

**This convention covers whether work survives at all.** Where it is placed once it does — which
ref holds it, in what order the commits land — belongs to the git commit and git branch
conventions. Recovering a trunk that has commits on it by mistake is the branch convention's,
and it reaches for the same mechanisms described here.

**Destroying what the remote holds is the push convention's.** A force-push takes commits off a
branch somebody else may be standing on, and the local work survives it — so what that costs, and
why it is handed to a person rather than run here, belong to `/hoc-git-push`.
