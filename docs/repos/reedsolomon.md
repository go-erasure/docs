# reedsolomon

Reed-Solomon MDS erasure code over GF(2¹⁶) — PAR2-compatible field, SPLIT(16,4) SIMD.

A systematic MDS code: a Cauchy generator matrix guarantees any `dataShards` of the `dataShards + parityShards` shards reconstruct the original data. It doubles as the arithmetic core for [PAR2](https://en.wikipedia.org/wiki/Parchive).

## Highlights

- **CGO_ENABLED=0**, standard library only, no third-party dependencies.
- **MDS** systematic code via a Cauchy generator matrix.
- **PAR2-compatible field**: GF(2¹⁶), primitive polynomial `0x1100B`, generator `2`.
- **SIMD region multiply** (GF-Complete SPLIT(16,4)): `PSHUFB`/`TBL` fast path on amd64 (SSSE3) and arm64 (NEON), ~13.5x scalar, proven bit-identical.
- The other four 64-bit targets build on the verified scalar fallback today.
- **100% test coverage**, verified across nine GOOS/GOARCH targets.

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
