<!--
SPDX-License-Identifier: CC-BY-4.0
Copyright (c) 2026 Pawel Pajor
-->

# DOTS binary format

Version 0.1.0.

## 1. Introduction

This document specifies the binary format of DOTS streams. The
conventional filename suffix for a stream stored on a storage medium is
`.dots`.

The meaning of the octets of a layer, the effect of a `clear`, the
composition of the layers and the timing of the presentation of frames
are outside this document and are covered by [DOTS-CONTENT].

This document and [DOTS-CONTENT] have the same `major` and `minor`
version, which are the ones a stream carries in its `dots-version`. A
normative change to either document changes `major` or `minor` in both.
The `update` numbers of the two documents are independent.

### 1.1. Conventions

- The key words `shall` and `shall not` are normative.
- Grammar is written in ABNF [RFC5234], as updated by [RFC7405],
  including its core rule `OCTET`. In ABNF, octet values are written
  `%x..`; in prose they are written `0x..`.
- The normative parts of this document are the ABNF blocks, the
  requirements and the definitions in Section 2. All other text is
  explanatory.
- Every requirement has an ID made of a section prefix and a number, for
  example `HDR-2`.

## 2. Terminology

- **Stream**: a sequence of octets, received by a reader in that order,
  each octet exactly once.
- **Writer**: a program that produces a stream.
- **Reader**: a program that consumes a stream.
- **Grid**: the rectangle of cells, `columns` wide and `rows` high,
  where `columns` and `rows` are the `x` and `y` of the header
  `dimensions`. A cell is addressed as `(x, y)`: `x` is its column and
  `y` its row, each counted from 0, columns increasing to the right and
  rows downward.
- **Cell**: an element of the grid.
- **Run**: octets anchored to a grid position. The octets are an encoded
  form of cell content: from this document alone, nothing can be assumed
  about which cells the decoded content occupies.
- **Frame**, **block layer**, **text layer**: the parts of a stream
  matched by the rules `frame`, `block-layer` and `text-layer` of
  Section 3.
- **Bit `n` of `flags`**: bit `n` of the value of `flags` as defined by
  INT-1, bit 0 being the least significant. **`contains-text`**: bit 0
  of `flags`.

## 3. Stream

- **STR-1** A stream shall match the rule `stream` of the grammar formed
  by the ABNF blocks of this document.

```
stream      = header *frame end-of-stream
frame       = [clear] block-layer text-layer
block-layer = *run end-of-blocks
text-layer  = *run end-of-text
```

- **STR-2** Octets after `end-of-stream` are not part of the stream and
  shall be ignored by a reader.

## 4. Basic structures

### 4.1. LEB128 integer

```
leb128 = %x00-7F / 1*%x80-FF %x01-7F
```

- **INT-1** The value of a `leb128` is the following, where `b0`, `b1`,
  `b2`, ... are its octets in stream order:

```
value = b0[6:0] | (b1[6:0] << 7) | (b2[6:0] << 14) | ...
```

- **INT-2** The value of a `leb128` shall be at most `2^64 - 1`.

### 4.2. Token

```
clear         = %x01
start-of-run  = %x02
end-of-blocks = %xFE
end-of-text   = %xFD
end-of-stream = %xFF
```

### 4.3. Version

```
version = major minor update
major   = leb128
minor   = leb128
update  = leb128
```

### 4.4. 2D coordinates

```
coordinates = x y
x           = leb128
y           = leb128
```

## 5. Header

```
header       = fourcc dots-version flags dimensions frame-rate
fourcc       = %x64.6F.74.73
dots-version = version
flags        = leb128
dimensions   = coordinates
frame-rate   = leb128
```

- **HDR-1** The `major` and `minor` fields of `dots-version` shall be
  `0` and `1`.
- **HDR-2** A reader shall decode a stream independently of the `update`
  field of `dots-version`, and shall not reject a well-formed stream
  because of it.
- **HDR-3** In `dimensions`, `x` and `y` shall each be at least 1.

### 5.1. Flags

- **HDR-4** Bits 1 and above of `flags` shall be 0.
- **HDR-5** If `contains-text` is 0, the `text-layer` of every frame
  shall contain no runs.

## 6. Run

```
run      = start-of-run position count data
position = coordinates
count    = leb128
data     = *OCTET
```

- **RUN-1** `data` shall consist of exactly `count` octets.
- **RUN-2** `position` shall lie inside the grid: `x < columns` and
  `y < rows`.

## 7. Conformance

A requirement whose subject is a stream, or a part of a stream, is a
stream requirement. A stream is well-formed if it satisfies every stream
requirement of this document.

- **CNF-1** A writer shall emit only well-formed streams.
- **CNF-2** A reader shall reject every stream that is not well-formed,
  and shall satisfy every requirement of this document whose subject is
  a reader.

[DOTS-CONTENT] adds requirements for streams and readers, and states
the conformance of the format as a whole.

## 8. References

### 8.1. Normative references

- [RFC5234] Crocker, D., Ed., and P. Overell, Augmented BNF for Syntax
  Specifications: ABNF, STD 68, RFC 5234, January 2008.
- [RFC7405] Kyzivat, P., Case-Sensitive String Support in ABNF,
  RFC 7405, December 2014.

### 8.2. Informative references

- [DOTS-CONTENT] DOTS content and composition, version 0.1.

[DOTS-CONTENT]: dots-content.md
[RFC5234]: https://www.rfc-editor.org/info/rfc5234/
[RFC7405]: https://www.rfc-editor.org/info/rfc7405/
