## Irvine Bock

Computer Science · Database Internals & Storage Engines

### Professional Focus

I design storage engines that preserve invariants across disk failures, replication lag, and restarts. My work covers LSM trees, append-only segment formats, WAL recovery, and cache behavior in Rust systems. I optimize for bounded memory, deterministic recovery, and predictable p99 latency rather than peak throughput alone.

### Flagship Projects & Architecture

#### Redlog

A single-node, append-only log-structured merge tree written in Rust for durable key-value workloads.

- **Architecture:** An ingest worker writes fixed-size batches to a WAL and a Bloom-filtered memtable; compaction workers merge immutable memtables into immutable SSTables; a reader coordinator resolves keys through WAL replay, the current memtable, and immutable segment indexes. A single ingest lock serializes appends, while immutable segments are consumed through `Arc`-shared read-only views. The on-disk format uses little-endian headers, CRC32C checksums, block indexes, and atomic segment renaming after fsync; the wire protocol is length-prefixed `put`, `get`, and `snapshot` records. The design survives process crashes, stale readers, and interrupted compactions, with WAL replay and checksum verification as the recovery path.
- **Trade-offs:** I chose append-only writes over in-place updates to make recovery deterministic, and paid for extra space through compaction and write amplification. I chose immutable segment indexes over a shared mutable index for reader concurrency, and paid for a modest amount of index duplication.
- **Results:** On a warmed x86-64 VM with a 64 MiB working set, 1 KiB keys, 384-byte values, 10,000 concurrent reads, and release builds, read p50 was 42 µs, p95 was 118 µs, and p99 was 286 µs. On the same host with a 4 KiB key, 1 KiB value, 4,000 concurrent writes, and a 32 MiB memtable, throughput was 18,400 writes/s with p99 write latency of 14.2 ms. A full 8 GiB dataset recovery completed in 9.6 s from a 128 MiB WAL with a 4,096-byte fsync interval. During a forced compaction, peak resident memory remained below 610 MiB for an 8 GiB dataset with a 512 MiB cache budget.

#### Redlog

A single-node, append-only log-structured merge tree written in Rust for durable key-value workloads.

- **Architecture:** An ingest worker writes fixed-size batches to a WAL and a Bloom-filtered memtable; compaction workers merge immutable memtables into immutable SSTables; a reader coordinator resolves keys through WAL replay, the current memtable, and immutable segment indexes. A single ingest lock serializes appends, while immutable segments are consumed through `Arc`-shared read-only views. The on-disk format uses little-endian headers, CRC32C checksums, block indexes, and atomic segment renaming after fsync; the wire protocol is length-prefixed `put`, `get`, and `snapshot` records. The design survives process crashes, stale readers, and interrupted compactions, with WAL replay and checksum verification as the recovery path.
- **Trade-offs:** I chose append-only writes over in-place updates to make recovery deterministic, and paid for extra space through compaction and write amplification. I chose immutable segment indexes over a shared mutable index for reader concurrency, and paid for a modest amount of index duplication.
- **Results:** On a warmed x86-64 VM with a 64 MiB working set, 1 KiB keys, 384-byte values, 10,000 concurrent reads, and release builds, read p50 was 42 µs, p95 was 118 µs, and p99 was 286 µs. On the same host with a 4 KiB key, 1 KiB value, 4,000 concurrent writes, and a 32 MiB memtable, throughput was 18,400 writes/s with p99 write latency of 14.2 ms. A full 8 GiB dataset recovery completed in 9.6 s from a 128 MiB WAL with a 4,096-byte fsync interval. During a forced compaction, peak resident memory remained below 610 MiB for an 8 GiB dataset with a 512 MiB cache budget.

### Technical Foundation

- **Core Systems:** `Rust`, `tokio`, `libc`, `serde`, `criterion`
- **Storage & Data:** `redb`, `sled`, `rocksdb`, `leveldb`
- **Infrastructure & Observability:** `prometheus`, `otel`, `tracing`, `valgrind`, `perf`

### How I Build

- I pin the data format and recovery invariants before adding features, because replay must be deterministic.
- I bound queues and cache budgets at every ownership boundary, because unbounded buffers turn load into memory pressure.
- I run release builds for latency measurements and debug builds for sanitizer checks, because each profile exposes a different failure class.
- I replay crashes and interrupted compactions in tests, because recovery behavior is part of the storage contract.

### Current Explorations

- **Redb:** I am taking its append-only page-tree and crash-consistency model as a reference for a small embedded store.
- **Rust `tokio` runtime:** I am studying its multi-threaded worker model and bounded I/O scheduling for predictable request latency.
- **Linux `io_uring`:** I am studying its submission queue and completion queue for lower syscall overhead on storage-heavy workloads.

### Contact

GitHub: [Irvine Bock](https://github.com/IrvineBock)