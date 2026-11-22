# DOTS examples

Two complete tiny streams, end to end, showing what a DOTS stream looks
like ([dots-binary.md](../dots-binary.md),
[dots-content.md](../dots-content.md)). Streams that illustrate single
requirements, accepted or rejected, are in the
[validation suite](../validation-suite/README.md).

| Example                                                       | Shows                                                                                        |
|---------------------------------------------------------------|----------------------------------------------------------------------------------------------|
| [full](full.txt) ([.dots](full.dots))                         | Blocks and text: block and text runs, a held frame, zero blocks and erasers, and a `clear`.  |
| [full-no-text](full-no-text.txt) ([.dots](full-no-text.dots)) | Blocks only: the full stream with the `contains-text` flag clear and every text layer empty. |

## Listings

Each example comes as a pair: a commented listing (`.txt`) and the same
stream as a binary file (`.dots`). Reading a listing is simple: drop the
comments, from `//` to the end of the line, and the spaces, and what is
left is a series of octets in C notation, `0x` and two hex digits,
separated by commas. The `.dots` file holds those octets in binary, in
the same order, and nothing else.

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
python3 txt2dots.py full.txt > full.dots
```
