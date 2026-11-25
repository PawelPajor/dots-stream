# DOTS validation suite

Cases, each a small stream illustrating a specific point of the
specification ([dots-binary.md](../dots-binary.md),
[dots-content.md](../dots-content.md)), version 0.1.x, that is, any
`update` of 0.1. The list aims to be as short as possible while
covering every requirement on a stream.

## Cases

`accept/` holds the cases a reader shall accept: minimal streams, each
showing one point, often one that looks wrong but is well-formed.
Complete streams, end to end, are in the
[examples](../examples/README.md).

`reject/` holds the cases a reader shall reject (CNF-2 of
dots-binary.md, CON-2 of dots-content.md). Each differs from a stream a
reader shall accept by exactly one defect, so that it violates exactly
one requirement.

STR-1 can be violated in countless ways; the cases show the common
ones. RUN-1 can only be shown by a `count` that runs past the end of
the stream: a `count` that is too small makes the remaining data octets
read as tokens, which can form a different, well-formed stream.

Requirements whose subject is a reader, rather than a stream (STR-2,
HDR-2, CMP-1 to CMP-3, TIM-1 and TIM-2), are illustrated by the accept
cases.

## Files

Each case is a set of files sharing its name:

- `.txt`: the commented listing of the stream, in the form described
  [with the examples](../examples/README.md#listings). Its first
  comment says, for people, what the case shows.
- `.dots`: the stream itself.
- `.meta`: what a reader shall produce, in [TOML](https://toml.io).
- `.surface`: accept cases only, the surface of each frame.

### `.meta`

```toml
requirements = ["RUN-1", "DEC-1", "PIX-1"]

[header]
dots-version = "0.1.0"
contains-text = false
columns = 16
rows = 4
frame-rate = "10"
```

- `requirements`: the IDs of the requirements the case illustrates or,
  for a reject case, the one it violates.
- `[header]`, accept cases only: the decoded header. `dots-version` is
  `major.minor.update`. `frame-rate` is a decimal string, since its
  largest value, 2^64 - 1, does not fit a TOML integer.
- `code` and `offset`, reject cases only: the error a reader returns
  (Section 8 of dots-binary.md).

### `.surface`

UTF-8 text, one character per cell: the cell's character if it has
one, otherwise the Braille pattern of its block (Section 5 of
dots-content.md). Cells follow each other row by row, each row from
left to right, and frames follow each other in stream order. There are
no line ends or separators: a cell's character may itself be LF or CR,
so each frame is exactly `columns * rows` characters, taken from
`.meta`. A stream without frames has an empty surface file. When
comparing, U+2800 and U+0020 both stand for an empty cell; the files
of the suite use U+2800.
