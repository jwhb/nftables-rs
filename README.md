# nftables-rs &emsp; [![Latest Version]][crates.io]

[Latest Version]: https://img.shields.io/crates/v/nftables.svg
[crates.io]: https://crates.io/crates/nftables

Safe abstraction for nftables JSON API ([libnftables-json](https://manpages.debian.org/testing/libnftables1/libnftables-json.5.en.html)).
It can be used to create nftables rulesets in Rust and parse existing nftables rulesets from JSON.
This library can also interact with local nftables system with helper functions for reading and applying rulesets.

nftables-rs is inspired by [nftnl-rs](https://github.com/mullvad/nftnl-rs), which directly accesses the nf_tables kernel subsystem to work with nftables.
The goal of this library is to provide access to the complete expressiveness of the nftables schema.

```toml
[dependencies]
nftables = "0.2.0"
```

## Example

Here are some examples that show use cases of this library.
Check out the `tests/` directory for more usage examples.

### Apply ruleset to nftables

This example applies a ruleset that creates and deletes a table to nftables.

```rust
use nft::{batch::Batch, helper, schema, types};

/// Applies a ruleset to nftables.
fn test_apply_ruleset() {
    let ruleset = example_ruleset();
    nft::helper::apply_ruleset(&ruleset, None, None).unwrap();
}

fn example_ruleset() -> schema::Nftables {
    let mut batch = Batch::new();
    batch.add(schema::NfListObject::Table(schema::Table::new(
        types::NfFamily::IP,
        "test-table-01".to_string(),
    )));
    batch.delete(schema::NfListObject::Table(schema::Table::new(
        types::NfFamily::IP,
        "test-table-01".to_string(),
    )));
    batch.to_nftables()
}
```

### Parse/Generate nftables ruleset in JSON format

This example compares nftables' native JSON out to the JSON payload generated by this library.

```rust
fn test_chain_table_rule_inet() {
    // nft add table inet some_inet_table
    // nft add chain inet some_inet_table some_inet_chain '{ type filter hook forward priority 0; policy accept; }'
    let expected: Nftables = Nftables {
        objects: vec![
            NfObject::CmdObject(NfCmd::Add(NfListObject::Table(Table {
                family: NfFamily::INet,
                name: "some_inet_table".to_string(),
                handle: None,
            }))),
            NfObject::CmdObject(NfCmd::Add(NfListObject::Chain(Chain {
                family: NfFamily::INet,
                table: "some_inet_table".to_string(),
                name: "some_inet_chain".to_string(),
                newname: None,
                handle: None,
                _type: Some(NfChainType::Filter),
                hook: Some(NfHook::Forward),
                prio: None,
                dev: None,
                policy: Some(NfChainPolicy::Accept),
            }))),
        ],
    };
    let json = json!({"nftables":[{"add":{"table":{"family":"inet","name":"some_inet_table"}}},{"add":{"chain":{"family":"inet","table":"some_inet_table","name":"some_inet_chain","type":"filter","hook":"forward","policy":"accept"}}}]});
    println!("{}", &json);
    let parsed: Nftables = serde_json::from_value(json).unwrap();
    assert_eq!(expected, parsed);
}
```

## License

Licensed under either of

* Apache License, Version 2.0
  ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
* MIT license
  ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.

## Maintainers

This project is currently maintained by the following developers:

|       Name       |      Email Address     |                GitHub Username               |
|:----------------:|:----------------------:|:--------------------------------------------:|
| Jasper Wiegratz  | wiegratz@uni-bremen.de |       [@jwhb](https://github.com/jwhb)       |
