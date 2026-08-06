# reedsolomon

Reed-Solomon MDS erasure code over GF(2¹⁶) — PAR2-compatible field, SPLIT(16,4) SIMD.

A systematic MDS code: a Cauchy generator matrix guarantees any `dataShards` of the `dataShards + parityShards` shards reconstruct the original data. It doubles as the arithmetic core for [PAR2](https://en.wikipedia.org/wiki/Parchive).

## Highlights

- **CGO_ENABLED=0**; the only dependency is `golang.org/x/sys` (CPU feature detection).
- **MDS** systematic code via a Cauchy generator matrix.
- **PAR2-compatible field**: GF(2¹⁶), primitive polynomial `0x1100B`, generator `2`.
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
```

## Install

```sh
go get github.com/go-erasure/reedsolomon
```

Requires Go 1.26.4 or newer.

## Links

- Source — <https://github.com/go-erasure/reedsolomon>
- API reference — <https://pkg.go.dev/github.com/go-erasure/reedsolomon>
