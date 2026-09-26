---
name: hoc-npm-categorize-deps
description: "Which field of the manifest a package is declared in — what a peer dependency buys that a plain one does not, what the package manager does when the two disagree, and why a second copy is the failure worth designing against. Use when adding a package to a manifest, and whenever a published package has to say what it needs from whoever installs it. Moving a version already declared belongs to the dependency-raising convention."
---

# Categorizing a dependency

**The field a package is declared in is a decision about whose copy runs.** A `dependency` says the
package brings its own; a `peerDependency` says it uses the one the installing project already has.
For a library that hands framework objects back to its caller, those are not two spellings of the
same thing.

## A second copy is the failure to design against

**Where a package and the project installing it both bring their own copy, the package manager
installs both and says nothing.** Measured: a project on one major and a package declaring the
previous one as a plain `dependency` produced two copies — the project's at the root of
`node_modules` and the package's nested beneath it — and the install reported no error and no
warning.

**What that costs is not obvious from either manifest.** The package builds its objects from the
copy it can see, so a factory it exposes hands back an object of the version *it* resolved, while
the caller writes against the version *they* resolved. Reproduced: an object built by the nested
copy accepted a call written for the newer major, raised nothing, and silently failed to behave
as that major would.

**A peer declaration turns the same disagreement into a refused install.** The same pair declared
as a peer failed with a resolution error before anything was written to disk. **The value is the
refusal**, not the sharing: one copy is what a project would have had anyway, and what the peer
buys is that a mismatch cannot pass unnoticed.

## Which field

| The package… | Field |
| :-- | :-- |
| brings code it alone calls | `dependencies` |
| hands the installing project objects built from it, or takes such objects back | `peerDependencies` |
| needs it only to build, test or lint itself | `devDependencies` |

**A package that never imports something at run time may still need it declared.** Where its only
reach is through an object handed to it, the copy in play is the caller's — which is what the peer
field says, and the reason the package can carry no runtime import of it at all.

**The types that describe a package follow the package.** Where the package is a peer, its type
declarations are a peer beside it, so that the two move as one and a consumer cannot end up with
the types of one major and the code of another.

## Two things measured about how peers install

- **A package's own peers are installed when it runs `npm install`.** A manifest declaring a peer
  and nothing else still got the package into `node_modules`, so **listing it under
  `devDependencies` as well buys nothing** and leaves two places to keep in step.
- **A package linked from a path gets no peers installed at all.** Linking one for local
  development reports them as unmet, so the linked package behaves differently from the published
  one — and the difference shows up as missing modules rather than as a version disagreement.

## Out of scope

- **Moving a version already declared.** That belongs to the dependency-raising convention
- **Whether to take the package on in the first place.** That belongs to the convention on
  adopting a third-party tool
- **Approving its install scripts, and answering an advisory against it.** Those belong to the
  conventions beside this one
