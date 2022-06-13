---
title: String Matching - KMP, Rabin-Karp, Z, and Aho-Corasick
description: Finding a pattern inside a text by sliding it along one character at a time wastes work every time a partial match fails. KMP reuses that work with a failure table, Rabin-Karp turns comparison into arithmetic with a rolling hash, and Aho-Corasick matches thousands of patterns in a single pass.
date: 2022-06-13
draft: false
slug: /algorithms/StringMatching
tags:
  - Algorithms
  - Strings
---

You want every place the word `ana` appears in `banananana`. The obvious method: line `ana` up at position 0, compare left to right, mismatch, shift right by one, try again. It works, and for short strings it's fine.

It's also quietly wasteful. When `ana` matches `ana` at position 1 and then you shift to position 2, you re-compare characters you already looked at — and you already *know* what they are. The text has `n` characters, the pattern has `m`, and the naive method is `O(nm)` because it keeps forgetting what it just learned. This post is three ways to stop forgetting.

## The naive baseline

```python
def naive_search(text, pat):
    n, m = len(text), len(pat)
    for i in range(n - m + 1):
        if text[i:i + m] == pat:
            yield i
```

Worst case is `text = "aaaa...a"`, `pat = "aaa...ab"`: every alignment matches `m - 1` characters before failing, `O(nm)` total. The three classic algorithms below all get to `O(n + m)`, each with a different trick.

## KMP: precompute where to resume

Knuth–Morris–Pratt's insight: when a match fails after `k` matched characters, those `k` characters *are* a known prefix of the pattern. If that prefix has a **border** — a proper suffix that is also a prefix — you can slide the pattern forward so the border lines up, and resume comparing from there instead of from scratch. The text pointer never moves backward.

Precompute, for each position `i` of the pattern, the length of the longest proper prefix of `pat[0..i]` that is also a suffix of it. Call it the **failure function** `fail[i]` (also "prefix function" or "LPS array").

```python
def prefix_function(pat):
    fail = [0] * len(pat)
    k = 0
    for i in range(1, len(pat)):
        while k > 0 and pat[i] != pat[k]:
            k = fail[k - 1]                 # fall back to the next shorter border
        if pat[i] == pat[k]:
            k += 1
        fail[i] = k
    return fail

def kmp_search(text, pat):
    fail = prefix_function(pat)
    k = 0
    for i, ch in enumerate(text):
        while k > 0 and ch != pat[k]:
            k = fail[k - 1]
        if ch == pat[k]:
            k += 1
        if k == len(pat):
            yield i - k + 1
            k = fail[k - 1]                 # keep going for overlapping matches
```

For `pat = "abcab"`, `fail = [0, 0, 0, 1, 2]`: after matching `abca` and failing, `fail[3] = 1` says "you still have a matching `a`, resume from pattern index 1." Both the preprocessing and the scan are amortised `O(m)` and `O(n)` — the `k` variable increases at most once per character and the `while` loop only ever decreases it, so total work is linear. `O(n + m)`, no backtracking on the text.

## Z-algorithm: the same information, laid out differently

The **Z-array** of a string `s` gives, for each position `i`, the length of the longest substring starting at `i` that matches a prefix of `s`. Build it in `O(|s|)` by maintaining the rightmost `[l, r)` interval that's known to match a prefix (the **Z-box**) and reusing it:

```python
def z_array(s):
    n = len(s)
    z = [0] * n
    z[0] = n
    l = r = 0
    for i in range(1, n):
        if i < r:
            z[i] = min(r - i, z[i - l])    # reuse work inside the current Z-box
        while i + z[i] < n and s[z[i]] == s[i + z[i]]:
            z[i] += 1
        if i + z[i] > r:
            l, r = i, i + z[i]
    return z
```

To search, run it on `pat + separator + text` (with a `separator` that appears in neither). Any position `i` in the text portion with `z[i] == len(pat)` is a match. Same linear bound as KMP; many people find the Z-array easier to derive from scratch, and it doubles as a building block for other string problems.

## Rabin-Karp: compare numbers, not characters

Turn every length-`m` window of the text into a number via a hash, and turn the pattern into a number the same way. A window matches *only if* its hash equals the pattern's hash — so you do one integer comparison per position instead of up to `m` character comparisons.

The trick that makes it fast is a **rolling hash**: computing the next window's hash from the current one in O(1). Treat the window as a base-`b` number mod a large prime `q`:

$$H(\text{s}[i..i{+}m]) = \left(\sum_{j=0}^{m-1} s[i+j]\cdot b^{\,m-1-j}\right) \bmod q$$

Sliding one character right: subtract the leading character's contribution, multiply by `b`, add the new trailing character.

```python
def rabin_karp(text, pat, b=256, q=(1 << 61) - 1):
    n, m = len(text), len(pat)
    if m > n:
        return
    high = pow(b, m - 1, q)
    ph = th = 0
    for i in range(m):                       # hash the pattern and first window
        ph = (ph * b + ord(pat[i])) % q
        th = (th * b + ord(text[i])) % q
    for i in range(n - m + 1):
        if ph == th and text[i:i + m] == pat:   # verify on hash hit
            yield i
        if i < n - m:
            th = ((th - ord(text[i]) * high) * b + ord(text[i + m])) % q
```

Expected `O(n + m)`. A hash collision forces a full character check, so a bad `q` or an adversary crafting collisions degrades it to `O(nm)` — use a 61-bit prime and a random base. Where Rabin–Karp shines is **multi-pattern search of equal length** (hash all patterns into a set, one rolling hash over the text) and **2D pattern matching** (roll a hash in both dimensions).

## Aho-Corasick: many patterns at once

Searching a text for a *set* of patterns — a keyword blocklist, a set of virus signatures, a dictionary — you could run KMP once per pattern, `O(n · k)` for `k` patterns. Aho–Corasick does it in **`O(n + total pattern length + matches)`**, one pass.

Build a **trie** of all patterns. Then add KMP-style **failure links**: from each trie node, a link to the node representing the longest proper suffix of the current string that is also a prefix of some pattern. A BFS over the trie fills them in. Scanning the text, you walk the trie on each character, following failure links on a mismatch — exactly KMP, but over a tree of patterns instead of one string. **Output links** chain together every pattern that ends at the current node (or at a node reachable by failure links), so you report all matches, including ones nested inside others.

This is the structure behind `grep -F` with many `-e` flags, [content-filtering](/citadel/pentesting/web-exploitation) proxies, and intrusion-detection signature engines.

## Choosing

- **One pattern, worst-case guarantee, no hashing risk** → KMP or Z. KMP if you want streaming (never look back at the text); Z if you find it cleaner or need the array for something else.
- **One or many equal-length patterns, average-case is fine, 2D** → Rabin–Karp.
- **Many patterns, different lengths, one pass** → Aho–Corasick.
- **Repeated queries against a fixed text** → build a **suffix array** (all suffixes sorted, binary-search any pattern in `O(m log n)`) or a **suffix automaton** once, then answer each query fast. That's the right structure when the text is a database and patterns keep arriving.

The standard-library `str.find` in most languages is a tuned hybrid (often a variant of Boyer–Moore–Horspool, which skips *ahead* on mismatches using a bad-character table and is sublinear on typical text). Reach for the algorithms here when you need the guarantee, the multi-pattern pass, or the building block.

## The one idea to keep

Naive matching is slow because it throws away what a failed comparison just told it. KMP writes that knowledge into a failure table so the text pointer never rewinds; Rabin–Karp encodes each window as a rolling number so a comparison is one subtraction and a multiply; Aho–Corasick generalises the failure table to a whole trie so thousands of patterns cost the same single pass as one. Same `O(n + m)`, three different things to precompute.
