# go-erasure

Pure-Go erasure coding — Reed-Solomon over GF(2^16) and the RozoFS Mojette transform, with go-asmgen SIMD.

`go-erasure` provides two independent, dependency-free erasure-coding libraries in pure Go: a Reed-Solomon MDS codec over GF(2¹⁶) (the PAR2-compatible field) and the XOR-only Mojette discrete-Radon transform used by [RozoFS](https://rozofs.github.io/rozofs/master/). The region hot loops dispatch to [go-asmgen](https://github.com/go-asmgen/asmgen) SIMD kernels, validated byte-for-byte against a scalar oracle on real hardware.

## Repositories

<div class="repo-grid" markdown>
<a class="repo-card" href="repos/reedsolomon.md"><code>reedsolomon</code><br><small>Reed-Solomon MDS erasure code over GF(2¹⁶) — PAR2-compatible field, SPLIT(16,4) SIMD.</small></a>
<a class="repo-card" href="repos/mojette.md"><code>mojette</code><br><small>Mojette transform (discrete Radon, XOR-only) — the RozoFS erasure code, SIMD on all six 64-bit targets.</small></a>
</div>
