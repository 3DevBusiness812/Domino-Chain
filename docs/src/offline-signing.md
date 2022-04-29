---
title: Offline Transaction Signing
---

Some security models require keeping signing keys, and thus the signing
process, separated from transaction creation and network broadcast. Examples
include:

- Collecting signatures from geographically disparate signers in a
  [multi-signature scheme](https://spl.dominochain.com/token#multisig-usage)
- Signing transactions using an [airgapped](<https://en.wikipedia.org/wiki/Air_gap_(networking)>)
  signing device

This document describes using Domino's CLI to separately sign and submit a
transaction.

## Commands Supporting Offline Signing

At present, the following commands support offline signing:

- [`create-stake-account`](cli/usage.md#domino-create-stake-account)
- [`create-stake-account-checked`](cli/usage.md#domino-create-stake-account-checked)
- [`deactivate-stake`](cli/usage.md#domino-deactivate-stake)
- [`delegate-stake`](cli/usage.md#domino-delegate-stake)
- [`split-stake`](cli/usage.md#domino-split-stake)
- [`stake-authorize`](cli/usage.md#domino-stake-authorize)
- [`stake-authorize-checked`](cli/usage.md#domino-stake-authorize-checked)
- [`stake-set-lockup`](cli/usage.md#domino-stake-set-lockup)
- [`stake-set-lockup-checked`](cli/usage.md#domino-stake-set-lockup-checked)
- [`transfer`](cli/usage.md#domino-transfer)
- [`withdraw-stake`](cli/usage.md#domino-withdraw-stake)

- [`create-vote-account`](cli/usage.md#domino-create-vote-account)
- [`vote-authorize-voter`](cli/usage.md#domino-vote-authorize-voter)
- [`vote-authorize-voter-checked`](cli/usage.md#domino-vote-authorize-voter-checked)
- [`vote-authorize-withdrawer`](cli/usage.md#domino-vote-authorize-withdrawer)
- [`vote-authorize-withdrawer-checked`](cli/usage.md#domino-vote-authorize-withdrawer-checked)
- [`vote-update-commission`](cli/usage.md#domino-vote-update-commission)
- [`vote-update-validator`](cli/usage.md#domino-vote-update-validator)
- [`withdraw-from-vote-account`](cli/usage.md#domino-withdraw-from-vote-account)

## Signing Transactions Offline

To sign a transaction offline, pass the following arguments on the command line

1. `--sign-only`, prevents the client from submitting the signed transaction
   to the network. Instead, the pubkey/signature pairs are printed to stdout.
2. `--blockhash BASE58_HASH`, allows the caller to specify the value used to
   fill the transaction's `recent_blockhash` field. This serves a number of
   purposes, namely:
   _ Eliminates the need to connect to the network and query a recent blockhash
   via RPC
   _ Enables the signers to coordinate the blockhash in a multiple-signature
   scheme

### Example: Offline Signing a Payment

Command

```bash
domino@offline$ domino pay --sign-only --blockhash 5Tx8F3jgSHx21CbtjwmdaKPLM5tWmreWAnPrbqHomSJF \
    recipient-keypair.json 1
```

Output

```text

Blockhash: 5Tx8F3jgSHx21CbtjwmdaKPLM5tWmreWAnPrbqHomSJF
Signers (Pubkey=Signature):
  FhtzLVsmcV7S5XqGD79ErgoseCLhZYmEZnz9kQg1Rp7j=4vC38p4bz7XyiXrk6HtaooUqwxTWKocf45cstASGtmrD398biNJnmTcUCVEojE7wVQvgdYbjHJqRFZPpzfCQpmUN

{"blockhash":"5Tx8F3jgSHx21CbtjwmdaKPLM5tWmreWAnPrbqHomSJF","signers":["FhtzLVsmcV7S5XqGD79ErgoseCLhZYmEZnz9kQg1Rp7j=4vC38p4bz7XyiXrk6HtaooUqwxTWKocf45cstASGtmrD398biNJnmTcUCVEojE7wVQvgdYbjHJqRFZPpzfCQpmUN"]}'
```

## Submitting Offline Signed Transactions to the Network

To submit a transaction that has been signed offline to the network, pass the
following arguments on the command line

1. `--blockhash BASE58_HASH`, must be the same blockhash as was used to sign
2. `--signer BASE58_PUBKEY=BASE58_SIGNATURE`, one for each offline signer. This
   includes the pubkey/signature pairs directly in the transaction rather than
   signing it with any local keypair(s)

### Example: Submitting an Offline Signed Payment

Command

```bash
domino@online$ domino pay --blockhash 5Tx8F3jgSHx21CbtjwmdaKPLM5tWmreWAnPrbqHomSJF \
    --signer FhtzLVsmcV7S5XqGD79ErgoseCLhZYmEZnz9kQg1Rp7j=4vC38p4bz7XyiXrk6HtaooUqwxTWKocf45cstASGtmrD398biNJnmTcUCVEojE7wVQvgdYbjHJqRFZPpzfCQpmUN
    recipient-keypair.json 1
```

Output

```text
4vC38p4bz7XyiXrk6HtaooUqwxTWKocf45cstASGtmrD398biNJnmTcUCVEojE7wVQvgdYbjHJqRFZPpzfCQpmUN
```

## Offline Signing Over Multiple Sessions

Offline signing can also take place over multiple sessions. In this scenario,
pass the absent signer's public key for each role. All pubkeys that were specified,
but no signature was generated for will be listed as absent in the offline signing
output

### Example: Transfer with Two Offline Signing Sessions

Command (Offline Session #1)

```text
domino@offline1$ domino transfer Fdri24WUGtrCXZ55nXiewAj6RM18hRHPGAjZk3o6vBut 10 \
    --blockhash 7ALDjLv56a8f6sH6upAZALQKkXyjAwwENH9GomyM8Dbc \
    --sign-only \
    --keypair fee_payer.json \
    --from 674RgFMgdqdRoVtMqSBg7mHFbrrNm1h1r721H1ZMquHL
```

Output (Offline Session #1)

```text
Blockhash: 7ALDjLv56a8f6sH6upAZALQKkXyjAwwENH9GomyM8Dbc
Signers (Pubkey=Signature):
  3bo5YiRagwmRikuH6H1d2gkKef5nFZXE3gJeoHxJbPjy=ohGKvpRC46jAduwU9NW8tP91JkCT5r8Mo67Ysnid4zc76tiiV1Ho6jv3BKFSbBcr2NcPPCarmfTLSkTHsJCtdYi
Absent Signers (Pubkey):
  674RgFMgdqdRoVtMqSBg7mHFbrrNm1h1r721H1ZMquHL
```

Command (Offline Session #2)

```text
domino@offline2$ domino transfer Fdri24WUGtrCXZ55nXiewAj6RM18hRHPGAjZk3o6vBut 10 \
    --blockhash 7ALDjLv56a8f6sH6upAZALQKkXyjAwwENH9GomyM8Dbc \
    --sign-only \
    --keypair from.json \
    --fee-payer 3bo5YiRagwmRikuH6H1d2gkKef5nFZXE3gJeoHxJbPjy
```

Output (Offline Session #2)

```text
Blockhash: 7ALDjLv56a8f6sH6upAZALQKkXyjAwwENH9GomyM8Dbc
Signers (Pubkey=Signature):
  674RgFMgdqdRoVtMqSBg7mHFbrrNm1h1r721H1ZMquHL=3vJtnba4dKQmEAieAekC1rJnPUndBcpvqRPRMoPWqhLEMCty2SdUxt2yvC1wQW6wVUa5putZMt6kdwCaTv8gk7sQ
Absent Signers (Pubkey):
  3bo5YiRagwmRikuH6H1d2gkKef5nFZXE3gJeoHxJbPjy
```

Command (Online Submission)

```text
domino@online$ domino transfer Fdri24WUGtrCXZ55nXiewAj6RM18hRHPGAjZk3o6vBut 10 \
    --blockhash 7ALDjLv56a8f6sH6upAZALQKkXyjAwwENH9GomyM8Dbc \
    --from 674RgFMgdqdRoVtMqSBg7mHFbrrNm1h1r721H1ZMquHL \
    --signer 674RgFMgdqdRoVtMqSBg7mHFbrrNm1h1r721H1ZMquHL=3vJtnba4dKQmEAieAekC1rJnPUndBcpvqRPRMoPWqhLEMCty2SdUxt2yvC1wQW6wVUa5putZMt6kdwCaTv8gk7sQ \
    --fee-payer 3bo5YiRagwmRikuH6H1d2gkKef5nFZXE3gJeoHxJbPjy \
    --signer 3bo5YiRagwmRikuH6H1d2gkKef5nFZXE3gJeoHxJbPjy=ohGKvpRC46jAduwU9NW8tP91JkCT5r8Mo67Ysnid4zc76tiiV1Ho6jv3BKFSbBcr2NcPPCarmfTLSkTHsJCtdYi
```

Output (Online Submission)

```text
ohGKvpRC46jAduwU9NW8tP91JkCT5r8Mo67Ysnid4zc76tiiV1Ho6jv3BKFSbBcr2NcPPCarmfTLSkTHsJCtdYi
```

## Buying More Time to Sign

Typically a Domino transaction must be signed and accepted by the network within
a number of slots from the blockhash in its `recent_blockhash` field (~1min at
the time of this writing). If your signing procedure takes longer than this, a
[Durable Transaction Nonce](offline-signing/durable-nonce.md) can give you the extra time you
need.