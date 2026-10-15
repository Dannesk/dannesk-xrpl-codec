# dannesk-xrpl-codec

[![Rust](https://img.shields.io/badge/Rust-2024_edition-000000?logo=rust&logoColor=white)](https://www.rust-lang.org)
[![ci](https://github.com/Dannesk/dannesk-xrpl-codec/actions/workflows/ci.yml/badge.svg)](https://github.com/Dannesk/dannesk-xrpl-codec/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-3B3F43)](LICENSE)

An encoder for the XRP Ledger's binary transaction format, for the transactions the
[Dannesk](https://github.com/Dannesk/app) wallet signs, and a codec for its addresses. 


## Example

```rust
use dannesk_xrpl_codec::*;

/// A signed XRP payment, as the blob to submit. `sign` signs the 32-byte
/// hash with the sender's secp256k1 key and returns the DER signature.
fn payment(
    from: &str,
    to: &str,
    public_key: &[u8],
    sign: impl Fn(&[u8; 32]) -> Vec<u8>,
) -> Result<Vec<u8>, String> {
    let mut fields = vec![
        transaction_type(TransactionType::Payment),
        account(from)?,
        destination(to)?,
        amount(Amount::xrp(1_000_000)?), // 1 XRP, in drops
        fee(12)?,                        // drops
        sequence(1),
        flags(0),
        signing_pub_key(public_key),
    ];
    let signature = sign(&signing_hash(&fields)?);
    fields.push(txn_signature(&signature));
    encode(&fields)
}
```

## License

MIT. See [LICENSE](LICENSE).
