---
name: hoc-git-push
description: "Conventions for `git push` itself: the permission each one takes and how narrowly that permission counts, and the force-push that is handed to a person rather than run here. What a branch is and where it merges belongs to the git branch convention; what a pull request says belongs to the pull request convention. Use before every push, and before any command that changes what the remote holds."
---

# Git Push

A push is the moment local work stops being local. Everything before it can be rewritten,
folded, split or abandoned with nobody else affected; everything after it is in someone
else's clone, in a CI record, and in the parents of merge commits yet to be made. This
convention is about that moment, and nothing else.

What a branch is, which branch it merges into and how, belongs to the git branch
convention. What a pull request says belongs to the pull request convention. Neither is
repeated here.

## Every push takes permission granted in this thread

**A push is not made unless it was asked for in the conversation that is running.** Not
implied by the work having been handed over, not carried in from a previous session, not
inherited from permission granted for something else.

- **One grant covers one push.** "Push it" is this push. It is not the next one, and it is
  not an upgrade of this one to `--force`.
- **A force-push is not one of the pushes this section governs.** No grant reaches it,
  because it is not run here at all — see below.
- **Where there is no grant, ask in one line** — naming what is pushed and to which branch
  — and do not proceed on the answer being obvious.
- These are all **not** permission: a complaint, a statement of knowledge ("I know what a
  force push is" says an explanation is unwanted, not that a push is wanted), silence, a
  reply on an adjacent topic, a statement that there is time left, or approval of something
  reversible ("commit that" does not contain a push).
- **That the result would be harmless is not a reason to push.** Whether it was asked for
  is the whole of the test, and recoverability is not a defence offered afterwards.

While waiting for the answer, hand over the local state instead: the branch, the commits
stacked on it, the state of the working tree, and the base the pull request would take.

**Preparing the material needs no permission.** A pull request body, a list of commits, the
command itself written out for a human to run — all of these can be produced freely. What
takes permission is the operation against the remote.

## A trunk is never force-pushed

The trunks are `main` and the four branches that may merge into it — `release/x.x.x`,
`hotfix/xxxx`, `dev` and `env`. **None of them takes `git push --force` or
`--force-with-lease`**, and none takes the local rewrites that would make one necessary —
`rebase`, `commit --amend`, or `reset` onto a commit that has been pushed.

**There is no permission that unlocks this.** Asking does not produce it, and being asked
directly does not either. It is what the names mean, and the git branch convention states it
as a property of the branches rather than a preference about commands.

- **The reason is who else is holding the branch.** A trunk is what every other branch is
  cut from, so its commits are already in clones, in merge commits' parents, and in
  whatever CI recorded against them. Rewriting it does not correct a mistake — it makes
  everyone else's copy disagree with the remote, silently, until they try to push.
- **A mistake already on a trunk is corrected by a new commit**, on a branch that merges in
  like any other.
- **On `main` of a repository that deploys from it, a rewrite goes further than that.** The
  branch is the record of what was released — published, handed to whoever cloned it, or put
  in front of users. Rewriting it rewrites when each of those happened.
- **`dev` is legacy, and the rule still covers it.** It stays in the `main-guard` allowlist
  for backward compatibility and nothing new is opened on it, but a repository still carrying
  one holds it to everything above.

## What a push is for

**A push exists to open a pull request or to update one.** That is the shape of nearly
every legitimate push: a sub-branch goes up so that a base branch can receive it through a
review.

- **A trunk's tip is advanced by a merged pull request, never by a push of a local merge.**
  Merging a sub-branch into `main` / `release/x.x.x` / `dev` / `env` locally and pushing the
  result puts the merge on the remote with no review behind it, and the `main-guard` and
  release workflows — which fire on pull requests — never run. Where the merge has to
  happen, it happens on the host.
  - **A general branch acting as a trunk is the exception**, because it is not one of the
    named branches. A sub-branch cut from `feature/xxxx` merges back into it locally with
    `--no-ff`, and that merge is pushed like any other commit.
- **`hotfix/xxxx` is the one trunk whose own commits are pushed directly.** It is worked on
  in place rather than through sub-branches, so pushing commits onto it is the normal thing,
  not a violation of the rule above. What still goes through a pull request is its merge into
  `main`.
- **A push onto a branch that already has an open pull request moves that pull request.**
  The head advances, the diff changes, and review that was done against the old head no
  longer describes what would merge. Where the pull request carried a verification someone
  else signed off on — a checksum agreed by two people, an approval on a named commit — that
  verification is void, and the pull request is closed rather than quietly extended.
- **Creating a trunk on the remote is a push, and a normal one.** A `release/x.x.x` opened
  locally with its marker commit has to reach the remote before anything can target it.
  Creating the branch and advancing an existing one are different acts; only the second is
  the pull request's.

### A `release/x.x.x` is pushed carrying its marker and nothing else

**The first push of a release branch happens while it holds exactly one commit: the empty
`Release x.x.x` marker.** Not later, with work already stacked on it.

```bash
git switch -c release/1.1.0 origin/main
git commit --allow-empty -m 'Release 1.1.0'
git push -u origin release/1.1.0        # one commit on the branch, and it is empty
```

- **Everything after that arrives through a pull request.** A branch first pushed with commits
  already on it has put those commits on the remote with nothing reviewing them, and no later
  pull request covers them either — the trunk begins its life holding work that never went
  through the gate every other commit on it will go through.
- **It is also what lets the sub-branches exist.** They are cut from the trunk and take it as
  their base, and both need the branch to be on the host. The earliest moment it can be there
  is the moment it is created, which is why the push is not deferred until there is something
  to show.
- **The marker being empty is what makes this cheap.** Pushing a branch that carries no change
  commits nobody to anything and can be deleted without loss, so there is no reason to wait.
- The same holds for any branch opened as a trunk — `env`, or a general branch carrying its
  `Start <verb>ing xxxx` marker.

## Name the remote and the branch

```bash
git push origin <branch>        # every time
git push -u origin <branch>     # the first push of a new branch
```

**A bare `git push` is not written.** What it does depends on `push.default`, on whether
the branch has an upstream, and — since `push.autoSetupRemote` — on whether the repository
quietly creates one. Three configurations give three behaviours for the same five
characters, and the command carries no record of which one was in effect.

- **Never `--all` or `--mirror`.** They push every local branch: trunks whose tips are the
  pull request's to move, half-built sub-branches, and whatever was left lying around from
  a fortnight ago. A push names its branch for the same reason staging names its paths.
- **The branch pushed is the branch being worked on.** Pushing some other branch because it
  happened to be ready is a separate operation and takes its own permission.

## A force-push is the human's to run

**No `git push --force` or `--force-with-lease` is run here, on any branch, in any state.**
Not after asking, either — the question is not withheld out of caution. An answer to it
cannot carry what the act needs.

A trunk is the separate case above: there a force-push is not done at all, by anyone, because
the history is not rewritten. This section is about the branches where a rewrite is
legitimate — a sub-branch already pushed — and about who runs the push that publishes it.

### Why permission does not settle it

A grant is a person typing "yes". What the push does is discard commits the remote holds,
and whether that is safe rests on facts nobody has in front of them at that moment.

- **Asking manufactures consent rather than checking it.** Someone who does not have the
  branch's remote state in mind reads the question as a formality and answers yes. The grant
  then records that the question was put, not that the loss was understood.
- **The check that would settle it does not exist.** Walk it through and it runs out:
  1. `git fetch origin --prune` brings the remote state in.
  2. Compare `origin/<branch>` against the local `<branch>`, and read the author address of
     every commit the remote carries and the local does not.
  3. All of them being the operator's own address looks like clearance.
  4. It is not. That same address is what another session of the same person writes, so the
     commits about to be discarded may be work done elsewhere, minutes ago, still open.
  5. Several checkouts of one repository make that the ordinary case rather than the
     exception.

  There is no sixth step that closes it. The investigation narrows the question and never
  answers it, so a grant obtained after it is worth no more than one obtained before.
- **Two sessions can be holding the same branch name without either having cut it from the
  other.** The name is derived from the work — that is what the branch convention makes it do —
  so two sessions handed related tasks arrive at `fix/type-errors-reported-by-the-client-package`
  independently, each treating the branch as its own. Whichever pushed first is what
  `origin/<branch>` now holds, and to the other the divergence reads as its own branch needing
  to be caught up. Name, base and author address match in both readings, so nothing local tells
  them apart.
  - **The non-fast-forward rejection is the signal that this happened**, and it is the only one
    git gives. `--force` is precisely the removal of it. A rewrite and a collision look the same
    at the moment of the push, and the flag that gets the first one through also gets the second
    one through.
- **The failure is out of proportion to the mistake that causes it.** A force-push that lands
  on someone else's commits is a belt thrown off the engine: the work is gone from the remote,
  nothing announces it, and whoever lost it finds out when their own push is refused. Putting
  it back is a person's job, holding the reflogs of every checkout involved.

### What to do instead

Finish the local work, hand the command over, and stop.

```bash
git push --force-with-lease --force-if-includes origin <branch>
```

- **Do not ask whether to run it.** "Shall I force-push?" is the question this rule exists to
  remove. State that the branch now needs one, show the command, and leave it there.
- **The local rewrite itself is not restricted.** `rebase`, `squash`, `commit --amend` and
  `reset` stay reachable through the reflog, and they are done when asked. What stops here is
  only the push that publishes the rewrite.
- **Hand over what the remote stands to lose**: the branch, and the commits `origin/<branch>`
  holds that the local branch does not. Preparing that costs nothing and needs no permission,
  and it is the material the person actually decides on.
  - **Do not describe those commits as the branch's own earlier state.** They may be another
    session's work under the same name, and which of the two it is, is exactly what the person
    is being asked to judge. Report the subjects and the dates and let them read it.

The flags in the command are there for whoever runs it, and both are required:

- **`--force` on its own is not used.** It overwrites whatever the remote holds without
  looking, including a commit someone pushed to the same branch while the rebase was running.
- **`--force-with-lease` alone is weaker than it reads.** It compares against the
  remote-tracking ref, and any `git fetch` — including one an editor or a background job ran —
  refreshes that ref against commits nobody looked at. The lease then passes over work that
  was never seen. `--force-if-includes` closes it by requiring the remote tip to be reachable
  from the local reflog.
- **A sub-branch already under review still belongs to its reviewers.** Force-pushing it
  invalidates the comments anchored to the old commits. That belongs in the handover too.

## Tags are not pushed by hand

**On a repository carrying `release.yml`, the version tag is created by the merge**, on the
merge commit of the pull request into `main`, together with the GitHub Release. A tag
pushed by hand either duplicates that or contradicts it.

- **Never `--tags`.** It pushes every local tag, including whatever some tool created
  locally and whatever was left over from an experiment.
- **A pushed tag does not update.** Clients do not move a tag they already hold, so
  deleting it and pushing a corrected one leaves every existing clone on the old commit,
  with nothing to indicate a disagreement. A wrong tag is not fixed; it is superseded by
  the next version.
- Where a tag genuinely has to be pushed, it is named on its own, and it takes its own
  permission: `git push origin <tag>`.

## Deleting a remote branch

**The merge deletes the branch.** `delete_branch_on_merge` removes it on the host when the
pull request lands, and there is nothing left to do.

`git push --delete origin <branch>` is therefore for the cases the merge did not cover — a
branch pushed by mistake, or one abandoned without being merged. It destroys the only copy
of anything that is not also local, so it takes permission naming that branch, like any
other irreversible act.

## Before and after

**Before**: state what is about to go up.

```bash
git status --short
git log --oneline <remote-tracking ref>..<branch>
```

A push of eleven commits and a push of one look identical in the command. Reporting the
list is what lets the person granting permission grant it for the right thing.

**After**: confirm what the remote now holds.

```bash
git log --oneline -1 origin/<branch>
```

A push that failed on a hook, or that was rejected as non-fast-forward, reports it — but a
push that succeeded against the wrong branch reports success just as plainly.
