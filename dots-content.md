<!--
SPDX-License-Identifier: CC-BY-4.0
Copyright (c) 2026 Pawel Pajor
-->

# DOTS content and composition

Version 0.1.0.

## 1. Introduction

This document specifies the content that the runs of a DOTS stream carry,
the encoding of that content, the composition of the block layer and the
text layer into the surface of the grid, and the timing at which a
presenter displays successive frames. It completes the binary format
specified in [DOTS-BINARY].

Section 1 of [DOTS-BINARY] states how the versions of the two documents
relate.

The presentation of a surface is outside this document, apart from the
timing of frames, which Section 7 specifies. Section 5 describes, for
information, one presentation of blocks.

### 1.1. Conventions

- The conventions of Section 1.1 of [DOTS-BINARY] apply, including the
  requirement IDs. The section prefixes used here differ from those of
  [DOTS-BINARY].
- The normative parts of this document are the requirements and the
  definitions in Section 2. All other text is explanatory.

## 2. Terminology

The terms of Section 2 of [DOTS-BINARY] apply. This document adds the
following.

- **Pixel**: an element of a block, either on or off.
- **Block**: the graphical content of a cell: a matrix of pixels, 2
  columns wide and 4 rows high, encoded as one octet.
- **Character**: the textual content of a cell: a Unicode scalar value
  as defined in [UNICODE].
- **Eraser**: the character U+0000.
- **Decoded run**: the position of a run together with the blocks or
  characters obtained from its `data`.
- **State**: for every cell, its block and its character, if any.
- **Initial state**: the state in which every cell has the block 0x00
  and no cell has a character.
- **Surface**: the content of every cell as it is to be presented: its
  character if it has one, otherwise its block.
- **Presenter**: a reader that shows a surface to a person.
- **Displaying a frame**: showing the surface of the state left by the
  frame.

## 3. Decoded runs

Decoding turns the `data` of a run into blocks or characters. All other
sections of this document work on decoded runs.

- **DEC-1** A reader shall decode the `data` of a run in the block layer
  into one block per octet.
- **DEC-2** A reader shall decode the `data` of a run in the text layer
  as UTF-8 [RFC3629] into characters.
- **DEC-3** The `data` of every run in the text layer shall be
  well-formed UTF-8 as defined in Sections 3 and 4 of [RFC3629].
- **DEC-4** A reader shall assign the `i`-th block or character of a
  decoded run, counting from 0, to cell `(x + i, y)`, where `(x, y)` is
  the `position` of the run.
- **DEC-5** Every block or character of a decoded run shall be assigned
  to a cell inside the grid.

Example: the `data` octets `0x24 0x30 0xE2 0x82 0xAC` decode to three
characters: `$` (U+0024), `0` (U+0030) and `€` (U+20AC). The `count` of
the run is 5, and its decoded run has three characters, so it covers
three cells.

## 4. Blocks

- **PIX-1** The pixels of a block are numbered from 0 in row-major
  order: from the top-left pixel, left to right and top to bottom. Bit
  `n` of the octet of a block is set if and only if pixel `n` is on.

Example: the block 0x39 has bits 0, 3, 4 and 5 set. Its pixels, with
`#` for on and `.` for off, are:

```
# .
. #
# #
. .
```

## 5. Braille patterns

A presenter can show the block of a cell as one Unicode Braille Pattern.
Its code point is U+2800 plus the sum of the values added by the set
bits of the block:

| Set bit     | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    |
|-------------|------|------|------|------|------|------|------|------|
| Value added | 0x01 | 0x08 | 0x02 | 0x10 | 0x04 | 0x20 | 0x40 | 0x80 |

Example: the block 0x39 has bits 0, 3, 4 and 5 set, which add 0x01,
0x10, 0x04 and 0x20. The sum is 0x35, so the code point is U+2835,
which is `⠵`.

## 6. Composition

- **CMP-1** A reader shall apply the decoded runs of a frame in the
  order they appear in the stream, starting from the initial state if
  the frame is the first or has a `clear`, otherwise from the state
  left by the preceding frame.
- **CMP-2** A reader shall apply a decoded run of the block layer by
  setting, for each of its blocks, the block of the cell it is assigned
  to, and shall change nothing else.
- **CMP-3** A reader shall apply a decoded run of the text layer by
  setting, for each of its characters, the character of the cell it is
  assigned to, except that the eraser removes the character instead, and
  shall change nothing else.

A `clear` in the first frame is redundant, so a writer may omit it.

Example: frames 1 and 2 address cell `(3, 0)`, frame 3 clears the grid.
The last three columns show that cell after each step.

| Frame | Layer | Decoded content at `(3, 0)` | Block | Character | Surface    |
|-------|-------|-----------------------------|-------|-----------|------------|
| 1     | block | 0xFF                        | 0xFF  |           | block 0xFF |
| 1     | text  | `A` (U+0041)                | 0xFF  | `A`       | `A`        |
| 2     | text  | eraser (U+0000)             | 0xFF  |           | block 0xFF |
| 3     | none  | `clear`                     | 0x00  |           | block 0x00 |

## 7. Presentation timing

- **TIM-1** If `frame-rate` is not 0, a presenter shall attempt to
  display subsequent frames at `frame-rate` Hz.
- **TIM-2** If `frame-rate` is 0, a presenter shall display each frame
  as soon as it has read the `end-of-text` of the frame.

If a presenter has not read a frame by the time TIM-1 calls for its
display, the behaviour of the presenter is outside this document.

## 8. Conformance

The requirements of this document apply in addition to those of
[DOTS-BINARY], where Section 7 defines a stream requirement.

- **CON-1** A writer shall emit only streams that satisfy every stream
  requirement of this document.
- **CON-2** A reader shall reject every stream that does not satisfy
  every stream requirement of this document, and shall satisfy every
  requirement of this document whose subject it is.

## 9. References

### 9.1. Normative references

- [DOTS-BINARY] DOTS binary format, version 0.1.
- [RFC3629] Yergeau, F., UTF-8, a transformation format of ISO 10646,
  STD 63, RFC 3629, November 2003.
- [UNICODE] The Unicode Consortium, The Unicode Standard.

[DOTS-BINARY]: dots-binary.md
[RFC3629]: https://www.rfc-editor.org/info/rfc3629/
[UNICODE]: https://www.unicode.org/versions/latest/
