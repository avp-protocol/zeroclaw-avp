<p align="center">
  <img src="https://raw.githubusercontent.com/avp-protocol/spec/main/assets/avp-shield.svg" alt="AVP Shield" width="80" />
</p>

<h1 align="center">zeroclaw-avp</h1>

<p align="center">
  <strong>ZeroClaw SecretBackend integration for AVP</strong><br>
  Drop-in replacement · Same API · Hardware security
</p>

<p align="center">
  <a href="https://crates.io/crates/zeroclaw-avp"><img src="https://img.shields.io/crates/v/zeroclaw-avp?style=flat-square&color=00D4AA" alt="Crates.io" /></a>
  <a href="https://github.com/avp-protocol/zeroclaw-avp/actions"><img src="https://img.shields.io/github/actions/workflow/status/avp-protocol/zeroclaw-avp/ci.yml?style=flat-square" alt="CI" /></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-Apache_2.0-blue?style=flat-square" alt="License" /></a>
</p>

---

## Overview

`zeroclaw-avp` implements ZeroClaw's `SecretBackend` trait using the Agent Vault Protocol. Replace ZeroClaw's default credential storage with AVP — get hardware-grade security without changing your agent code.

## Installation

```toml
[dependencies]
zeroclaw-avp = "0.1"
```

## Quick Start

```rust
use zeroclaw::Agent;
use zeroclaw_avp::AvpSecretBackend;

fn main() -> zeroclaw::Result<()> {
    // Create AVP-backed secret store
    let secrets = AvpSecretBackend::from_config("avp.toml")?;

    // Use with ZeroClaw agent
    let agent = Agent::builder()
        .secret_backend(secrets)
        .build()?;

    // Secrets are now stored in AVP vault instead of ~/.zeroclaw/credentials
    agent.run()
}
```

## Migration from Default Backend

```bash
# Export existing secrets
zeroclaw secrets export > secrets.json

# Import into AVP
avp import secrets.json --backend keychain

# Update zeroclaw.toml
echo '[secrets]
backend = "avp"
config = "avp.toml"' >> zeroclaw.toml

# Clean up
rm secrets.json
zeroclaw secrets clear  # Remove old plaintext secrets
```

## Configuration

### zeroclaw.toml

```toml
[secrets]
backend = "avp"
config = "avp.toml"  # Path to AVP config
```

### avp.toml

```toml
[backend]
type = "keychain"  # or "file", "hardware", "remote"

[workspace]
name = "zeroclaw-default"
```

## Backend Selection

```rust
use zeroclaw_avp::{AvpSecretBackend, Backend};

// OS Keychain (recommended)
let secrets = AvpSecretBackend::with_backend(Backend::Keychain)?;

// Hardware secure element (maximum security)
let secrets = AvpSecretBackend::with_backend(Backend::Hardware {
    device: "/dev/ttyUSB0".into(),
})?;

// Remote vault (team environments)
let secrets = AvpSecretBackend::with_backend(Backend::Remote {
    url: "https://vault.company.com".into(),
})?;
```

## API Compatibility

`AvpSecretBackend` implements the full `SecretBackend` trait:

| Method | AVP Operation |
|--------|---------------|
| `get(key)` | RETRIEVE |
| `set(key, value)` | STORE |
| `delete(key)` | DELETE |
| `list()` | LIST |
| `exists(key)` | LIST + filter |

## Security Comparison

| Backend | Infostealer | Host Compromise | Memory Dump |
|---------|:-----------:|:---------------:|:-----------:|
| ZeroClaw default | ✗ | ✗ | ✗ |
| AVP File | ✗ | ✗ | ✗ |
| AVP Keychain | ✓ | ✗ | ✗ |
| AVP Hardware | ✓ | ✓ | ✓ |

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Apache 2.0 — see [LICENSE](LICENSE).

---

<p align="center">
  <a href="https://github.com/avp-protocol/spec">AVP Specification</a> ·
  <a href="https://github.com/zeroclaw/zeroclaw">ZeroClaw</a>
</p>
