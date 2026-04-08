# ironwall-edge-bench

Benchmarking harness for Rust/WASM edge compute on Cloudflare Workers.

Measures Groth16 ZK-SNARK verification performance within the hard constraints of a Cloudflare Workers V8 isolate. Built to validate whether cryptographic proof verification can execute within real-time latency budgets at the network edge.

## Benchmarks (Measured)

| Metric | Measured | Target | Status |
|--------|----------|--------|--------|
| Groth16 verification (native) | 3.44ms | < 5ms | ✅ |
| Groth16 verification (WASM) | ~8.6ms | < 10ms | ✅ |
| WASM binary size | 381 KB | < 500 KB | ✅ |
| Workers CPU ceiling | 50ms | 50ms | 17% utilization |
| Round-trip (warm isolate) | ~123ms | — | Measured |
| Round-trip (cold start) | ~298ms | — | Measured |
| Invalid proof rejection | ~76ms | — | Measured |

## Architecture

```
Client (curl/k6) --> Cloudflare Workers --> WASM Groth16 Verifier --> 200 + JWT / 401
                          |
                     V8 Isolate
                     50ms CPU / ~128MB RAM
                     #![no_std] Rust → wasm32-unknown-unknown
```

The harness deploys a `#![no_std]` Rust Groth16 verifier compiled to WASM. It performs BN254 optimal Ate pairing checks on 128-byte compressed proofs. On valid proof: mints a scoped HS256 JWT. On invalid proof: instant 401 (Cryptographic Guillotine).

## Stack

- **Proof system:** Groth16 (BN254 curve) via `ark-bn254` / `ark-groth16`
- **Allocator:** `talc` (bump allocator, no GC pressure)
- **Serialization:** `ark-serialize` for proof deserialization
- **Compilation:** `wasm32-unknown-unknown` target, optimized with `wasm-opt -O3`
- **Anti-replay:** In-memory HashSet + 500ms timestamp window (MVP). Production: Durable Objects Bloom filter.

## Live Endpoint

The verifier is deployed and running at a Cloudflare Workers endpoint. Valid proofs return JWTs. Invalid proofs are rejected in ~76ms.

## About

Built by [Grideos Global Corp.](https://grideos.com) as part of the Ironwall verification platform.
