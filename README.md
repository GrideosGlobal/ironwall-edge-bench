# Ironwall Edge Compute Benchmarks

Performance benchmarks for Rust-compiled WebAssembly security modules on Cloudflare Workers.

## What This Measures

Ironwall's Edge Judge runs biological verification logic as Rust/WASM on Cloudflare Workers, targeting sub-21ms classification latency at the edge. This repo contains the benchmarking methodology and harness configuration for measuring:

- **Classification latency**: Time from request receipt to PASS/PHYSICS_FAIL response
- **WASM instantiation overhead**: Cold start vs warm invocation on Workers runtime
- **KV read/write latency**: Session state operations against Cloudflare KV
- **Proof-of-concept throughput**: Concurrent session handling under load

## Architecture

```
Browser -> Cloudflare Edge PoP -> Rust/WASM Worker -> KV State -> Response
           |                      |
           Network latency        Classification latency (target: <21ms)
```

## Benchmark Configuration

See `bench.toml` for harness configuration parameters.

## Results Summary

| Operation | P50 | P95 | P99 |
|-----------|-----|-----|-----|
| Classification (warm) | <1ms | <2ms | <5ms |
| KV Read | ~12ms | ~25ms | ~40ms |
| KV Write | ~15ms | ~30ms | ~50ms |
| Cold Start | ~8ms | ~15ms | ~25ms |
| End-to-End (network + compute) | ~50ms | ~120ms | ~200ms |

*Measured from YYC (Calgary) edge PoP. Results vary by geographic proximity to nearest Cloudflare data center.*

## Tech Stack

- Rust (compiled to wasm32-unknown-unknown)
- Cloudflare Workers runtime
- Cloudflare KV for session state
- worker-build for WASM compilation

## About

Part of the [Ironwall](https://grideos.com) cybersecurity platform by Grideōs Global Corp.

## License

MIT
