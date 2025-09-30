# NWC One-to-Many (nwc-otm)

:::warning
This project will take all your money, don't use it in production.
:::

A CLI application that creates a one-to-many Nostr Wallet Connect service. It listens for wallet requests on specified relays and creates individual wallet services for each client, managing their balances separately while forwarding requests to an upstream wallet.

## Features

- **Multi-client support**: Creates separate wallet services for each client
- **Balance management**: Tracks individual client balances in a JSON file
- **Request forwarding**: Forwards valid requests to an upstream wallet
- **Balance validation**: Prevents payments exceeding client balances
- **Graceful shutdown**: Properly cleans up services on SIGINT

## Installation

```bash
bun install
```

## Usage

### Using CLI Arguments

```bash
bun run index.ts --relay <relay-url> --connect-uri <upstream-wallet-uri> --data <data-folder>
```

### Using Configuration File

Create a `config.json` file in the project root:

```json
{
  "relay": ["wss://relay.getalby.com/v1"],
  "connect-uri": "nostr+walletconnect://...",
  "data": "./data"
}
```

Then run:

```bash
bun run index.ts
```

### Arguments

- `--relay`, `-r`: Relay URL to listen on (can be specified multiple times)
- `--connect-uri`, `-c`: Upstream wallet connect URI (nostr+walletconnect://)
- `--data`, `-d`: Path to folder containing client data
- `--config`, `-f`: Path to config file (default: config.json)

**Note**: CLI arguments take precedence over config file values.

### Examples

#### Using CLI arguments:

```bash
bun run index.ts \
  --relay wss://relay.getalby.com/v1 \
  --connect-uri "nostr+walletconnect://..." \
  --data ./client-data
```

#### Using config file:

```bash
# Uses config.json in current directory
bun run index.ts

# Uses custom config file
bun run index.ts --config ./my-config.json

# Mix config file with CLI overrides
bun run index.ts --relay wss://additional-relay.com
```

## Balance File Format

The balance file is a JSON object mapping client public keys to their balance in millisatoshis:

```json
{
  "hex1...": 100000,
  "hex2...": 50000
}
```

If the file doesn't exist, it will be created automatically. Balances are updated in real-time as payments are made.

## Supported Methods

The service forwards the following wallet methods to the upstream wallet:

- `get_balance` - Returns the client's individual balance
- `get_info` - Returns upstream wallet info with client's balance
- `pay_invoice` - Pays a Lightning invoice (deducts from client balance)
- `pay_keysend` - Sends a keysend payment (deducts from client balance)
- `make_invoice` - Creates a new invoice (forwarded to upstream)
- `lookup_invoice` - Looks up an invoice (forwarded to upstream)
- `list_transactions` - Lists transactions (forwarded to upstream)

## How it Works

1. The service connects to the specified relays and listens for `WALLET_REQUEST_KIND` (23194) events
2. When a new client makes a request, a dedicated `WalletService` is created for that client
3. Each client service handles requests independently, checking balances before forwarding payments
4. Payment requests that exceed the client's balance are rejected
5. Successful payments update the client's balance in the JSON file

This project was created using `bun init` in bun v1.2.21. [Bun](https://bun.com) is a fast all-in-one JavaScript runtime.
