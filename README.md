# tokenscope-escrow-demo

Live demo of [TokenScope](https://github.com/integral-830/tokenscope) checking a Solana program it
did not write.

The program under test is the Anchor escrow from the Solana Foundation
[program-examples](https://github.com/solana-developers/program-examples/tree/main/tokens/escrow/anchor)
repository (MIT, copied unmodified from commit `f766367`). It uses `token_interface` and
`transfer_checked`, so it accepts Token-2022 mints.

| Path | What |
|---|---|
| `programs/escrow/` | upstream escrow source, unmodified |
| `Cargo.toml`, `Anchor.toml`, `Cargo.lock` | upstream workspace files |

The pull requests in this repository add TokenScope as a CI security gate.
