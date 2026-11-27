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

## TokenScope gate

`.github/workflows/tokenscope.yml` builds the escrow from source with `cargo-build-sbf`, loads the
`.so` into the Mollusk SVM together with the real Token-2022 and Associated Token Account programs,
runs the specs in `tokenscope/`, and uploads the findings to code scanning (SARIF). No change to the
program is needed. Each spec describes the mints, the accounts (ATAs, the offer PDA, the vault), the
instruction bytes (Anchor discriminator + args) and the checks.

| Spec | Setup | Check | Result |
|---|---|---|---|
| `escrow-fee-on-wanted-token.json` | 2% transfer fee on mint B | maker receives `token_b_wanted_amount` | **fails**: wanted 1000, maker received 980 (20 withheld) |
| `escrow-fee-on-offered-token.json` | 2% transfer fee on mint A | `take_offer` / `refund_offer` succeed | **both revert** with Token-2022 error 35 |

Error 35 is `AccountHasWithheldTransferFees`: the vault receives the offered tokens minus the fee and
keeps the withheld fee, and Token-2022 refuses to close an account with withheld fees. Both
instructions close the vault, so the offer can be neither taken nor refunded until somebody runs
`HarvestWithheldTokensToMint` on the vault (permissionless; after it `take_offer` succeeds). The
tokens are not lost, but the program itself has no path out.

These are design limits of an educational example with fee-bearing Token-2022 mints, not an attack
on a deployed protocol. Run locally:

```sh
cargo install --git https://github.com/integral-830/tokenscope --rev c03bf25 tokenscope-cli --locked
(cd programs/escrow && cargo-build-sbf --sbf-out-dir ../../target/deploy)
tokenscope sbf-run --spec tokenscope/escrow-fee-on-wanted-token.json --format text
```
