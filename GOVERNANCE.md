<!--
SPDX-License-Identifier: CC-BY-4.0
Copyright (c) 2026 Pawel Pajor
-->

# DOTS governance

This document says where the DOTS specification lives, who decides what
goes into it, and what may be called DOTS. It is short on purpose.

## 1. The specification

The canonical DOTS specification is the one published at:

https://github.com/PawelPajor/dots-stream

Copies elsewhere are copies. Where a copy and the canonical text differ,
the canonical text is the specification.

## 2. The maintainer

Pawel Pajor is the maintainer. The maintainer decides what goes
into the specification, assigns version numbers and defines flag bits.
Version numbers and flag bits that the canonical specification does not
define are reserved to the maintainer.

## 3. Proposing changes

Proposals are made as issues or pull requests in the canonical
repository. The maintainer accepts or declines each proposal and gives
the reason on the proposal. A normative change raises `major` or `minor`
in both documents of the specification, as Section 1 of
[dots-binary.md](dots-binary.md) requires.
[CONTRIBUTING.md](CONTRIBUTING.md) describes the process.

## 4. Contributions

By submitting a contribution you confirm that you wrote it or have the
right to submit it, and you license it under CC BY 4.0, the license of
the documents.

## 5. Implementing DOTS

Anyone may implement DOTS, in software, firmware or hardware, free of
charge and without asking. The maintainer will not assert any patent
claim against a conforming implementation.

## 6. The name

The license of the documents, CC BY 4.0, does not cover the name. This
section concerns the name only; it does not restrict copying or adapting
the text.

The name DOTS, the fourcc `dots` and the `.dots` suffix are for streams
that conform to the canonical specification. A variant whose normative
content differs from the canonical specification must use its own name
and its own fourcc, and must not present itself as DOTS. Translations
and reformatting that leave the normative content unchanged may keep the
name. Saying that a program reads or writes DOTS streams is fine when it
is true.

## 7. Changes to this document

The maintainer may change this document. Changes are recorded in the
repository history.
