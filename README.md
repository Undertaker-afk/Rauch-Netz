You are an expert Rust systems and cryptography programmer with deep experience building privacy-focused P2P networks (Tor/Arti, I2P-style, GNUnet, libp2p-based overlays). Your code must be production-grade, memory-safe, auditable, and prioritize anonymity, forward secrecy, and resistance to de-anonymization attacks.

Build the initial foundation for **Smoke Net** — a new decentralized anonymity overlay network written in pure Rust. It combines onion/garlic routing ideas from Tor and I2P with unique anti-de-anonymization features.

### Core Requirements (all must be implemented or properly stubbed with clear extension points):

1. **Project Structure**
   - Cargo workspace with these crates:
     - `smokenet-core` (shared types, crypto, config, error handling)
     - `smokenet-netdb` (distributed network database)
     - `smokenet-tunnel` (multi-hop tunnel / circuit creation and management)
     - `smokenet-anomaly` (anomaly detection and instant vanishing logic)
     - `smokenet-cli` (binary with basic commands: start node, send test message, status, etc.)
   - Use edition 2021, Rust 2024 edition where possible.

2. **P2P Stack**
   - Use the latest stable `libp2p` (feature flags: tokio, tcp, quic, noise, yamux or mplex, identify, ping, kad, gossipsub).
   - Build a custom `NetworkBehaviour` that combines Kademlia (for netDB), Gossipsub (for control messages and tunnel signaling), Identify, and Ping.
   - Strong preference for QUIC + Noise where possible, with TCP fallback.
   - Full async/await with Tokio runtime.

3. **netDB (Network Database)**
   - Kademlia-based DHT (or hybrid with floodfill-style elements inspired by I2P) to store:
     - RouterInfo: peer identity (PeerId + public keys, supported transports, capabilities, version).
     - LeaseSets / Service descriptors for hidden services.
   - Entries must be signed and time-limited with expiration.

4. **.smoke Addresses**
   - Cryptographic destination addresses in the form `xxxxxxxxxxxx.smk` or similar (base32 or z-base32 of a public key hash + version/checksum).
   - Resolution: lookup in netDB → get LeaseSet → establish inbound tunnel.
   - No central DNS or any centralized naming authority.

5. **Tunneling & Hopping**
   - Dynamic multi-hop tunnels (onion-style layered encryption, default 3–6 hops, configurable).
   - Support tunnel pooling and **tunnel hopping** (graceful rebuild/extend/rotate tunnels on the fly without breaking streams).
   - Garlic routing capability (bundle multiple messages in one tunnel for better metadata protection) — at least stubbed.
   - Streams should support TCP-like proxying (SOCKS5-like interface planned).

6. **Instant Vanishing on Anomaly Detection (core unique feature)**
   - Implement a pluggable anomaly detector in `smokenet-anomaly`.
   - Basic anomalies to detect (expandable): unusual traffic volume/spikes, timing correlation attempts, flood/Sybil patterns, high error rates on circuits, suspicious peer behavior.
   - On trigger:
     - Immediately zeroize ALL session keys, tunnel keys, and sensitive material using the `zeroize` crate (and `Zeroizing<T>` wrapper where appropriate). Use `ZeroizeOnDrop` for structs holding secrets.
     - Tear down all active tunnels instantly.
     - Purge related netDB entries and local caches.
     - Optionally broadcast a signed "poison pill" or self-destruct signal to connected peers via gossipsub.
   - All crypto operations in hot paths must be constant-time where possible.

7. **Fully Decentralized Bootstrap (NO reliance on any centralized clearnet provider)**
   - Support a configurable list of hardcoded seed peers (in code or TOML config) as a soft fallback only.
   - Persistent peer cache stored on disk (using `redb` or `sled`).
   - Strong emphasis on out-of-band bootstrap: manual peer import (file, CLI argument, QR code stub), peer exchange via any already-connected peer, gossip-based discovery.
   - Make the node able to join the network even if ALL seed nodes are unreachable — by leveraging any reachable peer discovered via manual entry or cache.
   - Use Kademlia bootstrap + identify + gossipsub for ongoing discovery. Avoid any DNS or clearnet bootstrap as primary method.

8. **Cryptography & Security**
   - Use `x25519-dalek` or `ed25519-dalek`, `chacha20poly1305` (or `aes-gcm` with care), `rand` (OsRng), `rustls` where needed.
   - All keys must implement `Zeroize` + `ZeroizeOnDrop`.
   - Forward secrecy via ephemeral keys per tunnel/hop.
   - Constant-time comparisons for sensitive data.
   - Extensive use of `tracing` for structured logging (with sensitive data redacted).

9. **Dependencies & Best Practices**
   - Minimal and audited crates only. Suggested: `libp2p = { version = "0.54", features = [...] }`, `tokio`, `tracing`, `serde`, `postcard` or `bincode`, `zeroize`, `redb` or `sled`, `anyhow` or custom errors, `clap` for CLI.
   - Proper error handling, configuration via TOML, secure random everywhere.
   - Code must compile cleanly (`cargo check` / `cargo build` ready).
   - Heavy commenting, especially on security-critical sections and why certain design choices were made.
   - Modular design so each component can be extended or audited independently.

### Output Format
First, output the complete workspace `Cargo.toml` (root) and individual crate `Cargo.toml` files with exact dependency versions and features.

Then, provide the full directory structure with file paths.

Then, implement in this order (make each part compile before moving on):
1. Core types and config.
2. Bootstrap logic + basic libp2p Swarm setup that works without any central server.
3. netDB (Kademlia store + lookup for RouterInfo and LeaseSets).
4. .smoke address generation and resolution.
5. Tunnel creation, layering, hopping, and basic data forwarding (stub encryption layers for now).
6. Anomaly detection + instant vanishing with proper zeroization.
7. CLI that can start a node and show status/peer list.

After the code, list the next 5 logical development steps with priorities (e.g., full encryption layering, SOCKS proxy, garlic routing, fuzzing setup, etc.).

Prioritize **security, decentralization, and correctness** over features. If something is too complex for a first pass, provide a clean stub with a detailed TODO explaining how to implement it securely.

Start building Smoke Net now.
