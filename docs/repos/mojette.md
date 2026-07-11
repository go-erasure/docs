# mojette

Mojette transform (discrete Radon, XOR-only) — the RozoFS erasure code, SIMD on all six 64-bit targets.

Data is a grid of blocks; a direction `(P, Q)` defines a family of discrete lines whose projection XOR-accumulates the cells on each line. Reconstruction is the iterative inverse Mojette (corner/back-projection); the Katz criterion decides sufficiency. Being XOR-only, it is an erasure code for whole blocks.

## Highlights

- **CGO_ENABLED=0**; only dependency is `golang.org/x/sys` (CPU feature detection).
- The Mojette / discrete-Radon code used by [RozoFS](https://rozofs.github.io/rozofs/master/), operating XOR-only over GF(2).
- go-asmgen region-XOR kernel on **all six** 64-bit SIMD targets (AVX2/SSE2, NEON, RVV, VSX, z-vector, LSX).
- `Reconstructible` implements the Katz sufficiency criterion.
- 100% test coverage; scalar oracle + differential fuzzer over every kernel.

## Measured on real hardware

XOR fast path, 1 MiB region (scalar to SIMD), on real silicon (not QEMU) — byte-exact
output, no SIGILL/SIGBUS, on every box:

| Arch    | Hardware                      | Scalar   | SIMD      | Speedup |
|---------|-------------------------------|----------|-----------|---------|
| arm64   | Apple M4 Max (NEON)           | 3.8 GB/s | 47.7 GB/s | ~12.6x  |
| ppc64le | POWER8E (VSX `VXOR`)          | 513 MB/s | 7.7 GB/s  | 15.1x   |
| s390x   | IBM z15 (vector `VX`)         | 1.58 GB/s| 15.0 GB/s | 9.5x    |
| riscv64 | SpacemiT X60, RVV1.0 VLEN=256 | 253 MB/s | 1.35 GB/s | 5.3x    |

amd64 (AVX2/SSE2) is verified native on CI; loong64 (LSX) is emitted and QEMU-checked,
with real-hardware validation pending. The scalar loop stays the correctness oracle: a
differential fuzzer checks every kernel byte-for-byte against it on every CI run.

## Install

```sh
go get github.com/go-erasure/mojette
```

Requires Go 1.26.4 or newer.

## Links

- Source — <https://github.com/go-erasure/mojette>
- API reference — <https://pkg.go.dev/github.com/go-erasure/mojette>
