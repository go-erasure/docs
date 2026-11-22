# reedsolomon

Reed-Solomon MDS erasure code over GF(2¹⁶) and the real PAR 2.0 code, SPLIT(16,4) SIMD.

Two codes over the same field, as two types so they cannot be mixed. `Encoder` is a systematic MDS code: a Cauchy generator matrix guarantees any `dataShards` of the `dataShards + parityShards` shards reconstruct the original data (its parity is not PAR2 recovery data). `PAR2` is the [PAR 2.0](https://en.wikipedia.org/wiki/Parchive) code: it produces and consumes real PAR2 recovery slices, byte-identical to par2cmdline 2.0.0 in both directions.

## Highlights

- **CGO_ENABLED=0**; the only dependency is `golang.org/x/sys` (CPU feature detection).
- **MDS** systematic code via a Cauchy generator matrix.
- **PAR 2.0 code** (`PAR2`, v0.6.0): the specification's matrix over little-endian words in GF(2¹⁶) (primitive polynomial `0x1100B`, generator `2`), checked against par2cmdline output, at least as fast as `Encoder` on s390x, ppc64le, amd64 and arm64.
- **SIMD region multiply** (GF-Complete SPLIT(16,4)): a byte-permute fast path on **all six** 64-bit targets — amd64 (SSSE3 `PSHUFB`), arm64 (NEON `TBL`), s390x (z13 `VPERM`), ppc64le (POWER8 VSX `VPERM`), riscv64 (RVV `vrgather.vv`) and loong64 (LSX `vshuf.b`) — each proven bit-identical to the scalar loop by a differential oracle.
- **100% test coverage**, verified across nine GOOS/GOARCH targets.

## Measured on real hardware

Region multiply (`galMulAdd`, 1 MiB region), scalar → SIMD, on real silicon (not
QEMU) — every kernel proven byte-identical to the scalar oracle:

| Arch    | Hardware                      | Scalar    | SIMD      | Speedup |
|---------|-------------------------------|-----------|-----------|---------|
| arm64   | Apple M4 Max (NEON)           | 1.45 GB/s | 19.5 GB/s | ~13.4x  |
| ppc64le | POWER8E (VSX `VPERM`)         | 202 MB/s  | 3.41 GB/s | 16.9x   |
| loong64 | Loongson 3C5000L (LSX)        | 255 MB/s  | 4.26 GB/s | 16.7x   |
| s390x   | IBM z15 (vector `VPERM`)      | 679 MB/s  | 9.49 GB/s | 14.0x   |
| riscv64 | SpacemiT X60, RVV1.0 VLEN=256 | 18 MB/s   | 1.11 GB/s | 61.1x   |

amd64 (SSSE3) is verified native on CI.

## Example

```go
enc, _ := reedsolomon.New(4, 2)
// Shards are byte slices read as big-endian uint16 words, same even length.
enc.Encode(shards)                 // writes parity shards
enc.Reconstruct(shards, present)   // rebuilds missing shards

// PAR 2.0 recovery data (little-endian words, exponents as in the .par2 file):
p, _ := reedsolomon.NewPAR2(len(data))
p.Encode(data, exponents, recovery)                // same bytes as par2cmdline
p.Reconstruct(data, present, exponents, recovery)  // rebuilds lost input slices
```

## Install

```sh
go get github.com/go-erasure/reedsolomon
```

Requires Go 1.26.4 or newer.

## Links

- Source — <https://github.com/go-erasure/reedsolomon>
- API reference — <https://pkg.go.dev/github.com/go-erasure/reedsolomon>
