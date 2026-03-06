# Nara Validator Setup

This document explains how to install and start a Nara validator node based on the Nara Solana distribution.

## 1. Get the Validator Binary

You can either build the validator from source or download a prebuilt release.

### Option A: Build from source

Source repository:

https://github.com/nara-chain/nara-solana

Follow the build instructions in that repository to compile the validator tools locally.

### Option B: Download a release package

Release list:

https://github.com/nara-chain/nara-solana/releases/

Example release package:

https://github.com/nara-chain/nara-solana/releases/download/v3.1.8/solana-release-x86_64-unknown-linux-gnu.tar.bz2

On a Linux x86_64 server, you can download and extract it like this:

```bash
curl -L -o solana-release.tar.bz2 \
  https://github.com/nara-chain/nara-solana/releases/download/v3.1.8/solana-release-x86_64-unknown-linux-gnu.tar.bz2

tar -xjf solana-release.tar.bz2
cd solana-release*
```

After extraction, confirm that the validator binary is available:

```bash
./agave-validator --version
```

## 2. Prepare the Server

Before starting the node, tune the server according to the validator system recommendations:

https://docs.anza.xyz/operations/setup-a-validator#system-tuning

At minimum, review the following before going live:

- File descriptor limits
- Kernel network buffers
- Disk performance and mount options
- Available RAM and swap policy
- Firewall rules for validator networking

Make sure the machine has stable network access and enough storage for ledger growth.

## 3. Generate Validator Keypairs

Create a directory for your keys and generate the validator identity keypair and vote account keypair:

```bash
mkdir -p keys

solana-keygen new -o keys/bootstrap-identity.json --no-passphrase
solana-keygen new -o keys/bootstrap-vote.json --no-passphrase
```

Notes:

- `bootstrap-identity.json` is the validator identity key.
- `bootstrap-vote.json` is the vote account keypair used by the validator.
- `--no-passphrase` is convenient for automated startup, but it reduces key protection. Use it only if your server security model allows it.

## 4. Start the Validator

Run the validator with the following command:

```bash
./agave-validator \
  --identity keys/bootstrap-identity.json \
  --vote-account keys/bootstrap-vote.json \
  --rpc-port 8899 \
  --gossip-port 8001 \
  --limit-ledger-size 50000000 \
  --private-rpc \
  --entrypoint 84.32.220.169:8001 \
  --expected-genesis-hash GENESIS_HASH \
  --max-genesis-archive-unpacked-size 100000000 \
  --full-rpc-api
```

## 5. Important Parameter Notes

- `--identity`: Path to the validator identity keypair.
- `--vote-account`: Path to the vote account keypair.
- `--rpc-port 8899`: Exposes the RPC service on port 8899.
- `--gossip-port 8001`: Uses port 8001 for cluster gossip communication.
- `--limit-ledger-size 50000000`: Limits on-disk ledger retention to help control storage usage.
- `--private-rpc`: Restricts RPC exposure instead of offering a public RPC endpoint.
- `--entrypoint 84.32.220.169:8001`: Bootstrap peer used to join the network.
- `--expected-genesis-hash GENESIS_HASH`: Replace `GENESIS_HASH` with the actual genesis hash published by the network.
- `--max-genesis-archive-unpacked-size 100000000`: Sets the maximum allowed unpacked size for the genesis archive.
- `--full-rpc-api`: Enables the full RPC API set.

## 6. Before Production Use

Review these items before considering the validator ready for production:

- Confirm that the entrypoint IP and genesis hash are current.
- Ensure the vote account is properly created and funded if the network requires it.
- If you plan to receive delegated stake, complete the separate stake account and delegation steps required by the network.
- Open only the ports you intend to expose.
- Back up all validator keys securely.

## 7. Basic Verification

After startup, verify that the process is running and the node is joining the cluster:

```bash
ps aux | grep agave-validator
```

You should also review the validator logs and confirm that the node is discovering peers, downloading ledger data, and progressing normally.

