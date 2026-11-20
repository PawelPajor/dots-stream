<!--
SPDX-License-Identifier: CC-BY-4.0
Copyright (c) 2026 Pawel Pajor
-->

# DOTS examples

Small streams, each illustrating a specific point of the specification
([dots-binary.md](../dots-binary.md),
[dots-content.md](../dots-content.md)). The list aims to be as short as
possible while covering every requirement on a stream.

Each example comes as a pair: a commented listing (`.txt`) and the same
stream as a binary file (`.dots`). Reading a listing is simple: drop the
comments, from `//` to the end of the line, and the spaces, and what is
left is a series of octets in C notation, `0x` and two hex digits,
separated by commas. The `.dots` file holds those octets in binary, in
the same order, and nothing else. The first comment of each listing
states the version of the specification the example is written against;
all examples are written against version 0.1.x, that is, any
`update` of 0.1.

The Python below is one way to produce a `.dots` file from its
listing. It writes the octets of the listing given as its argument to
standard output, and stops at any token outside a comment that is not
a `0x..` octet. It does not check the stream: a reject example comes
out as faithfully as an accept one.

```python
import re, sys

octets = bytearray()
with open(sys.argv[1], encoding="utf-8") as listing:
    for line in listing:
        for token in line.split("//", 1)[0].replace(",", " ").split():
            if not re.fullmatch(r"0x[0-9A-Fa-f]{2}", token):
                sys.exit(f"{sys.argv[1]}: not an octet: {token}")
            octets.append(int(token, 16))
sys.stdout.buffer.write(octets)
```

For example, saved as `txt2dots.py`:

```sh
python3 txt2dots.py accept/one-run.txt > accept/one-run.dots
```

## Accept

Streams a reader shall accept. The first two are complete, end to end;
the others are minimal streams, each showing one point, often a case
that looks wrong but is well-formed.

| Example                                                                                                   | Illustrates         | Shows                                                                                                                      |
|-----------------------------------------------------------------------------------------------------------|---------------------|----------------------------------------------------------------------------------------------------------------------------|
| [full](accept/full.txt) ([.dots](accept/full.dots))                                                       | STR-1, CMP-1, CMP-3 | A complete tiny stream with text: block and text runs, a held frame, zero blocks and erasers, and a `clear`.               |
| [full-no-text](accept/full-no-text.txt) ([.dots](accept/full-no-text.dots))                               | HDR-5               | The full stream with the `contains-text` flag clear and every text layer empty.                                            |
| [minimal](accept/minimal.txt) ([.dots](accept/minimal.dots))                                              | HDR-3, STR-1        | The smallest stream: a header for a 1 by 1 grid, no frames, and `end-of-stream`.                                           |
| [one-run](accept/one-run.txt) ([.dots](accept/one-run.dots))                                              | RUN-1, DEC-1, PIX-1 | One frame with one block run of one block, 0x39 (the example of Sections 4 and 5 of dots-content.md).                      |
| [text-flag-without-text](accept/text-flag-without-text.txt) ([.dots](accept/text-flag-without-text.dots)) | HDR-5               | `contains-text` is set, yet no frame has a text run. HDR-5 forbids only the converse.                                      |
| [empty-run](accept/empty-run.txt) ([.dots](accept/empty-run.dots))                                        | RUN-1, DEC-5        | A run with `count` 0: no data, no cell touched.                                                                            |
| [token-octets-in-data](accept/token-octets-in-data.txt) ([.dots](accept/token-octets-in-data.dots))       | RUN-1, DEC-1        | Block data made of the token values 0xFF, 0xFE, 0xFD, 0x02 and 0x01. They are data, since `count` says where the run ends. |
| [multi-octet-leb128](accept/multi-octet-leb128.txt) ([.dots](accept/multi-octet-leb128.dots))             | INT-1               | A 200 by 4 grid and a run at column 150: both take two LEB128 octets.                                                      |
| [leb128-max](accept/leb128-max.txt) ([.dots](accept/leb128-max.dots))                                     | INT-2               | `frame-rate` is 2^64 - 1, the largest LEB128 value allowed, written in ten octets.                                         |
| [update-nonzero](accept/update-nonzero.txt) ([.dots](accept/update-nonzero.dots))                         | HDR-2               | `dots-version` 0.1.7: a reader decodes it as any 0.1 stream.                                                               |
| [frame-rate-zero](accept/frame-rate-zero.txt) ([.dots](accept/frame-rate-zero.dots))                      | TIM-2               | `frame-rate` 0: each of the two frames is displayed as soon as its `end-of-text` is read.                                  |
| [clear-first-frame](accept/clear-first-frame.txt) ([.dots](accept/clear-first-frame.dots))                | CMP-1               | A `clear` in the first frame. It is redundant, since the first frame starts from the initial state anyway.                 |
| [overlapping-runs](accept/overlapping-runs.txt) ([.dots](accept/overlapping-runs.dots))                   | CMP-1, CMP-2        | Two block runs of one frame cover cell (1,0); the later one wins, leaving 0xFF 0x00 0xFF.                                  |
| [eraser-on-empty](accept/eraser-on-empty.txt) ([.dots](accept/eraser-on-empty.dots))                      | CMP-3               | An eraser on a cell that has no character: it changes nothing.                                                             |
| [utf8-count-vs-cells](accept/utf8-count-vs-cells.txt) ([.dots](accept/utf8-count-vs-cells.dots))          | DEC-2, DEC-4, DEC-5 | A text run at column 13 of 16 with `count` 5: its octets decode to 3 characters, which end exactly in the last column.     |
| [trailing-octets](accept/trailing-octets.txt) ([.dots](accept/trailing-octets.dots))                      | STR-2               | Octets after `end-of-stream`. They are not part of the stream and are ignored.                                             |

## Reject

Streams a reader shall reject (CNF-2 of dots-binary.md, CON-2 of
dots-content.md). Each differs from a stream a reader shall accept by
exactly one defect, so that it violates exactly one requirement: the
one its name starts with. Its first comment, and the table below, state
the code and offset a reader returns for it (Section 8 of
dots-binary.md).

STR-1 can be violated in countless ways; the examples show the common
ones. RUN-1 can only be shown by a `count` that runs past the end of
the stream: a `count` that is too small makes the remaining data octets
read as tokens, which can form a different, well-formed stream.

Requirements whose subject is a reader, rather than a stream (STR-2,
HDR-2, CMP-1 to CMP-3, TIM-1 and TIM-2), are illustrated by the accept
examples.

| Example                                                                                                            | Violates | Code                  | Offset | Defect                                                                                                                                               |
|--------------------------------------------------------------------------------------------------------------------|----------|-----------------------|--------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| [str-1-bad-fourcc](reject/str-1-bad-fourcc.txt) ([.dots](reject/str-1-bad-fourcc.dots))                            | STR-1    | `MALFORMED`           | 0      | The `fourcc` is "DOTS". ABNF `%x` values are exact octets, so case matters.                                                                          |
| [str-1-leb128-zero-tail](reject/str-1-leb128-zero-tail.txt) ([.dots](reject/str-1-leb128-zero-tail.dots))          | STR-1    | `MALFORMED`           | 9      | `columns` is written 0x90 0x00. A multi-octet `leb128` shall not end with 0x00.                                                                      |
| [str-1-unknown-token](reject/str-1-unknown-token.txt) ([.dots](reject/str-1-unknown-token.dots))                   | STR-1    | `MALFORMED`           | 11     | 0x03 where a run or `end-of-blocks` is expected.                                                                                                     |
| [str-1-clear-misplaced](reject/str-1-clear-misplaced.txt) ([.dots](reject/str-1-clear-misplaced.dots))             | STR-1    | `MALFORMED`           | 17     | A `clear` between the block layer and the text layer. A `clear` may only start a frame.                                                              |
| [str-1-no-end-of-text](reject/str-1-no-end-of-text.txt) ([.dots](reject/str-1-no-end-of-text.dots))                | STR-1    | `MALFORMED`           | 17     | A frame without its text layer: `end-of-blocks` is followed by `end-of-stream`. An empty text layer still needs `end-of-text`.                       |
| [str-1-no-end-of-stream](reject/str-1-no-end-of-stream.txt) ([.dots](reject/str-1-no-end-of-stream.dots))          | STR-1    | `TRUNCATED`           | 18     | The stream stops after a complete frame, without `end-of-stream`.                                                                                    |
| [int-2-leb128-overflow](reject/int-2-leb128-overflow.txt) ([.dots](reject/int-2-leb128-overflow.dots))             | INT-2    | `OVERFLOW`            | 10     | `frame-rate` is 2^64, one more than the largest LEB128 value allowed.                                                                                |
| [hdr-1-version](reject/hdr-1-version.txt) ([.dots](reject/hdr-1-version.dots))                                     | HDR-1    | `UNSUPPORTED_VERSION` | 5      | `dots-version` 0.2.0: minor shall be 1.                                                                                                              |
| [hdr-3-zero-columns](reject/hdr-3-zero-columns.txt) ([.dots](reject/hdr-3-zero-columns.dots))                      | HDR-3    | `INVALID_VALUE`       | 8      | A grid with 0 `columns`. The stream has no frames, so that no run is also outside the grid.                                                          |
| [hdr-4-reserved-flag](reject/hdr-4-reserved-flag.txt) ([.dots](reject/hdr-4-reserved-flag.dots))                   | HDR-4    | `INVALID_VALUE`       | 7      | `flags` is 0x02: bit 1 is reserved and shall be 0.                                                                                                   |
| [hdr-5-text-without-flag](reject/hdr-5-text-without-flag.txt) ([.dots](reject/hdr-5-text-without-flag.dots))       | HDR-5    | `UNEXPECTED_TEXT`     | 12     | A text run while `contains-text` is 0.                                                                                                               |
| [run-1-count-past-end](reject/run-1-count-past-end.txt) ([.dots](reject/run-1-count-past-end.dots))                | RUN-1    | `TRUNCATED`           | 19     | `count` is 4 but one octet of data follows. The reader takes the remaining 4 octets, including `end-of-stream`, as data and then runs out of stream. |
| [run-2-position-outside](reject/run-2-position-outside.txt) ([.dots](reject/run-2-position-outside.dots))          | RUN-2    | `OUTSIDE_GRID`        | 13     | A run at (0,4) in a grid of 4 rows. Its `count` is 0, so that no cell is also outside the grid (DEC-5).                                              |
| [dec-3-lone-continuation](reject/dec-3-lone-continuation.txt) ([.dots](reject/dec-3-lone-continuation.dots))       | DEC-3    | `INVALID_TEXT`        | 16     | Text data 0x80: a continuation octet with no leading octet.                                                                                          |
| [dec-3-truncated-sequence](reject/dec-3-truncated-sequence.txt) ([.dots](reject/dec-3-truncated-sequence.dots))    | DEC-3    | `INVALID_TEXT`        | 16     | Text data 0xE2 0x82: the first two octets of a three-octet sequence. Each run is decoded on its own, so a sequence cannot continue in the next run.  |
| [dec-3-overlong-eraser](reject/dec-3-overlong-eraser.txt) ([.dots](reject/dec-3-overlong-eraser.dots))             | DEC-3    | `INVALID_TEXT`        | 16     | Text data 0xC0 0x80: an overlong form of U+0000. The eraser is written 0x00 only.                                                                    |
| [dec-3-surrogate](reject/dec-3-surrogate.txt) ([.dots](reject/dec-3-surrogate.dots))                               | DEC-3    | `INVALID_TEXT`        | 16     | Text data 0xED 0xA0 0x80: the surrogate U+D800, which is no Unicode scalar value.                                                                    |
| [dec-5-block-run-overflows](reject/dec-5-block-run-overflows.txt) ([.dots](reject/dec-5-block-run-overflows.dots)) | DEC-5    | `OUTSIDE_GRID`        | 16     | A block run of 2 blocks at column 15 of 16. A run does not wrap to the next row.                                                                     |
| [dec-5-text-run-overflows](reject/dec-5-text-run-overflows.txt) ([.dots](reject/dec-5-text-run-overflows.dots))    | DEC-5    | `OUTSIDE_GRID`        | 19     | A text run at column 15 of 16 whose 4 octets decode to 2 characters; the second falls outside the grid.                                              |
