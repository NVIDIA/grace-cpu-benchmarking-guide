# Rust on NVIDIA Grace

Rust supports Arm64 systems as a tier1 platform.

### Large-System Extensions (LSE)

LSE improves system throughput for CPU-to-CPU communication, locks, and mutexes. LSE can be enabled in Rust, and there have been instances on larger machines where performance is improved by over three times by setting the `RUSTFLAG` environment variable and rebuilding your project.

```bash
export RUSTFLAGS="-Ctarget-cpu=neoverse-v2"
cargo build --release
```
