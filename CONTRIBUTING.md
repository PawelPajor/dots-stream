<!--
SPDX-License-Identifier: CC-BY-4.0
Copyright (c) 2026 Pawel Pajor
-->

# Contributing to DOTS

This document says how changes to this repository are made. Who decides
what goes into the specification is in [GOVERNANCE.md](GOVERNANCE.md).

## 1. Kinds of change

- **Normative**: changes what a stream is or how it is read: the ABNF
  blocks, the requirements or the definitions of the specification. A
  normative change raises `major` or `minor` in both
  [dots-binary.md](dots-binary.md) and
  [dots-content.md](dots-content.md), as Section 1 of dots-binary.md
  requires.
- **Editorial**: changes the wording of the specification without
  changing its meaning. It raises `update` in the document it touches.
- **Other**: the README, the logo, the examples index, the layout of the
  repository. It changes no version.

## 2. Proposing a normative change

1. Open an issue that states the problem and the proposed change: the
   sections and requirement IDs it touches, any flag bit or version it
   needs, and whether existing streams stay valid.
2. Make the change on a branch, one change per branch, for example
   `feat/<short-name>`.
3. In the same branch, raise the version in both documents and add or
   update the [examples](examples/README.md) the change affects.
4. Open a pull request that refers to the issue (`Closes #N`).
5. The maintainer accepts or declines it and gives the reason on the
   pull request. An accepted pull request is squash-merged into `main`.

Editorial and other changes may be proposed as a pull request without an
issue. The maintainer may commit them directly to `main`.

## 3. Examples

The examples in this repository are few and small, each illustrating a
specific point of the specification. They are versioned with it: an
example states the version it conforms to, and a change that alters an
example's meaning updates the example in the same pull request.

## 4. Commit messages

Commit messages follow
[Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/).
The types used here are:

| Type    | Use                                                                       | Version           |
|---------|---------------------------------------------------------------------------|-------------------|
| `feat`  | Normative addition                                                        | `minor`           |
| `fix`   | Normative correction                                                      | `minor`           |
| `docs`  | Editorial change to the specification, or a change to other documentation | `update`, or none |
| `chore` | Layout, tooling, housekeeping                                             | none              |

A change that makes existing streams invalid, or changes their meaning,
is marked with `!` after the type, or a `BREAKING CHANGE:` footer, and
raises `major`. While `major` is 0, it raises `minor` instead: before
1.0.0, a `minor` may be incompatible with the one before it.

The optional scope names the document: `binary` or `content`. For
example:

```
feat(binary): add a flag for looping streams
feat(binary)!: widen dimensions to two octets per axis
fix(binary): correct the ABNF of a run
docs(content): clarify the example in Section 4
chore: move examples into examples/
```

The subject is in the imperative mood, in lowercase, without a final
full stop.

## 5. Licence of contributions

Section 4 of [GOVERNANCE.md](GOVERNANCE.md) applies: by submitting a
contribution you confirm that you wrote it or have the right to submit
it, and you license it under CC BY 4.0.
