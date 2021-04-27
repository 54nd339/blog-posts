---
title: Encode and Decode Strings - Framing a List for the Wire
description: Serialising a list of arbitrary strings into one string and back — a delimiter fails because any byte can appear in the payload, so each string carries its own length prefix.
date: 2021-04-27
draft: false
slug: /dsa/encode-and-decode-strings
tags:
  - Competitive Programming
  - Arrays & Hashing
  - Strings
---

The naive instinct is to join with a comma or newline. That breaks the moment a payload contains that character. The fix is the one every wire format uses: tell the reader how many bytes to read before the bytes themselves.

## Description

Design `encode`, which turns a list of strings into a single string, and `decode`, which turns that string back into the original list. The strings may contain any characters, including any delimiter you might pick.

**Example**

```
Input:  ["neet", "code", "love", "you"]
encode -> "4#neet4#code4#love3#you"
decode -> ["neet", "code", "love", "you"]

Input:  ["we", "say", ":", "yes"]
encode -> "2#we3#say1#:3#yes"
```

**Constraints**

- $0 \le \text{strs.length} \le 100$
- $0 \le \text{strs}[i]\text{.length} \le 200$
- `strs[i]` contains any possible characters out of 256 valid ASCII values.

## Prerequisites

- String slicing and integer parsing.
- The concept of *framing*: making a byte stream self-describing so a reader knows where each field ends.

## Approach 1: Encoding & Decoding (lengths block first)

### Intuition

Write all the lengths at the front, separated by commas and terminated by a `#`, then all the payloads concatenated. The decoder reads the header to learn every length, then slices the payload.

### Algorithm

1. **Encode:** collect `len(s)` for each string; join them with commas; append `#`; append every string concatenated.
2. **Decode:** scan up to the first `#`, splitting the comma-separated lengths. Then, starting just past the `#`, repeatedly slice off the next `length` characters.

```python
class Codec:
    def encode(self, strs: list[str]) -> str:
        if not strs:
            return "#"
        header = ",".join(str(len(s)) for s in strs)
        return header + "#" + "".join(strs)

    def decode(self, s: str) -> list[str]:
        hash_idx = s.index("#")
        if hash_idx == 0:
            return []
        lengths = [int(x) for x in s[:hash_idx].split(",")]
        out, i = [], hash_idx + 1
        for length in lengths:
            out.append(s[i:i + length])
            i += length
        return out
```

### Complexity

- **Time:** $O(m + n)$ per call, where `m` is the total character count and `n` the number of strings.
- **Space:** $O(m + n)$ for the output.

## Approach 2: Encoding & Decoding (length prefix per string, optimal)

### Intuition

Interleave the framing instead of front-loading it: before each string, write its length, a `#`, then the string. The decoder reads digits until a `#`, converts to `k`, then takes exactly the next `k` characters — whatever they are.

### Algorithm

1. **Encode:** for each string, append `str(len(s))`, then `"#"`, then the string.
2. **Decode:** start at index `0`. Scan forward to the next `#` to read the length; move one past the `#`; slice that many characters as one decoded string; set the index to just after that slice; repeat until the string is consumed.

```python
class Codec:
    def encode(self, strs: list[str]) -> str:
        return "".join(f"{len(s)}#{s}" for s in strs)

    def decode(self, s: str) -> list[str]:
        out, i = [], 0
        while i < len(s):
            j = i
            while s[j] != "#":
                j += 1
            length = int(s[i:j])
            start = j + 1
            out.append(s[start:start + length])
            i = start + length
        return out
```

### Complexity

- **Time:** $O(m + n)$ per call — every character is written once and read once.
- **Space:** $O(m + n)$ for the output.

## Common Pitfalls

- **Reaching for a "rare" delimiter.** Any sentinel character can appear in a payload of arbitrary bytes; a length prefix has no such assumption.
- **Fixed-width length fields.** Padding the length to, say, 4 digits caps string length at 9999. `digits + "#"` has no limit and is unambiguous because the digits region never overlaps the payload.
- **Off-by-one after a slice (Approach 2).** The next length prefix begins at `start + length`, not `start + length + 1` — there is no separator after a payload.
- **Empty list.** Encode it to a distinguishable marker (here `"#"` / an empty header) so decode returns `[]`, not `[""]`.

## The keystone

Delimiter framing needs the delimiter to be impossible in the payload; length-prefix framing just tells the reader how far to read. That is why real formats carry lengths — HTTP's `Content-Length`, protobuf varints, Redis's `$6\r\nfoobar`. The same "write the size, then the bytes" idea returns in [Serialize and Deserialize Binary Tree](/citadel/dsa/serialize-and-deserialize-binary-tree).
