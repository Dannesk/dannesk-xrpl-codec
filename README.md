# dannesk-xrpl-codec

[![Rust](https://img.shields.io/badge/Rust-2024_edition-000000?logo=rust&logoColor=white)](https://www.rust-lang.org)
[![ci](https://github.com/Dannesk/dannesk-xrpl-codec/actions/workflows/ci.yml/badge.svg)](https://github.com/Dannesk/dannesk-xrpl-codec/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-3B3F43)](LICENSE)

An encoder for the XRP Ledger's binary transaction format, for the transactions the
[Dannesk](https://github.com/Dannesk/app) wallet signs, and a codec for its addresses. Written
by hand from rippled's definitions, on `sha2`, `bs58` and `hex` alone.

- **Transactions**: Payment, OfferCreate, OfferCancel and TrustSet. `encode` gives the blob to
  submit, and `signing_hash` the hash a secp256k1 key signs.
- **Typed fields**: one constructor per field, so a field cannot be misspelt into silence.
  Fields are written in the ledger's canonical order whatever order they come in, and a field
  given twice is refused.
- **Exact amounts**: XRP in drops, and tokens as a decimal string with their currency and
  issuer. A value the format cannot hold exactly is refused, never cut short or rounded to zero.
- **Addresses**: a classic r-address decoded to the 20-byte account id a transaction carries,
  and X-addresses decoded and encoded with the destination tag they carry.

## Verified against the ledger

The tests re-encode validated mainnet transactions from their published fields and must arrive
at the ids the ledger gave them; one wrong byte anywhere and they do not. Between them they
cover all four transaction types, every field constructor, transactions signed with either key
type (secp256k1 and ed25519) and both kinds of amount.

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

A token amount is `Amount::issued("12.5", currency, issuer)`, with the currency as the 40 hex
digits of its 160-bit code. For a recipient a person types in, `xaddress::resolve` takes an
r-address or an X-address and returns the classic address with any tag it carries:

```rust
use dannesk_xrpl_codec::xaddress;

let x = xaddress::encode("rHb9CJAWyB4rj91VRWn96DkukG4bwdtyTh", Some(12345)).unwrap();
let to = xaddress::resolve(&x).unwrap();
assert_eq!(to.classic, "rHb9CJAWyB4rj91VRWn96DkukG4bwdtyTh");
assert_eq!(to.tag, Some(12345));
```

## Scope

The transactions Dannesk signs, each signed by a single key: multi-signing is not supported.
Other transaction types and fields are not in the crate yet. An ed25519 key signs the unhashed
signing data rather than `signing_hash`, and the crate does not expose that data yet.
X-addresses are mainnet only: a testnet X-address is refused rather than decoded to an address
that would receive real funds.

## Status

The Dannesk desktop wallet has signed with this codec since Dannesk 0.1.2. It has not been independently audited.

## License

MIT. See [LICENSE](LICENSE).
