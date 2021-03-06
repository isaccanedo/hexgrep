# Fixed Pattern Scanner

The goal is to extract every 40 character hex string from a source and print
them for later examination to see if they are valid git commit shas.

Something _kinda_ like this, except filtering out hex strings longer than 40
characters.

```
$ grep -o -E '/[0-9a-f]{40}/' < input > maybe-commits.txt
```

The obstacle is this needs to be run over a lot of data and fairly frequently,
so it is probably worth optimizing. Or at least considering optimizing.

## Implementation

Naive implementations were done (in order) in Rust, Go, and JavaScript (node).
After implementing the same naive implementation in C I started down the path
that brought me here. In all cases, there are no dependencies other than the
relevant compiler/runtime; just standard i/o.

By applying some of the Boyer-Moore approach to a pattern instead of a string,
we can gain a couple features. We don't need to pre-process the needle since
each byte of the needle is effectively identical, but we can still make use of
the skipping because we do have a known fixed length.

In theory, what we end up with is a special case of what a good regular
expression engine would generate, but without the overhead of supporting other
types of patterns.

## Times

These times are best of 3 runs of piping sample file that is representative of
my intended workload (tar file generated by `docker save`).

The times themselves don't matter much; I'm more interested in how the different
implementations compare to each other.

### 329MB Sample

| Time | real | user | system |
|------|------|------|--------|
| grep | 0m8.105s | 0m7.194s | 0m0.893s |
| ripgrep | 0m0.821s | 0m0.735s | 0m0.071s |
| simple (Go) | 0m2.242s | 0m1.797s | 0m0.774s |
| simple (Rust) | 0m0.922s | 0m0.675s | 0m0.244s |
| simple (Node) | 0m3.713s | 0m3.113s | 0m0.732s |
| custom (C) | **0m0.123s** | **0m0.055s** | **0m0.066s** |

### 986MB Sample (Sample 1 x3)

| Time | real | user | system |
|------|------|------|--------|
| grep | 0m26.256s | 0m22.659s | 0m3.324s |
| ripgrep | 0m2.447s | 0m2.202s | 0m0.218s |
| simple (Go) | 0m6.793s | 0m5.453s | 0m2.364s |
| simple (Rust) | 0m2.811s | 0m2.054s | 0m0.722s |
| simple (Node) | 0m11.054s | 0m9.142s | 0m2.231s |
| custom (C) | **0m0.325s** | **0m0.135s** | **0m0.173s** |

By comparing the times you can see that each implementation is more or less
*O(n)* (or *O(nm)*;  since they are all using the same needle size it
essentially becomes a constant). The difference is in the constant factor. On
this sample, the custom search only actually looks at 1/9 of the bytes being
processed thanks to aggressive skipping.


