---
name: alby-bitcoin-payments
description: teaches agents how to use @getalby/cli for bitcoin lightning wallet operations using Nostr Wallet Connect (NIP-47). Use when the user needs to send/receive bitcoin payments, check wallet balance, create invoices, convert between fiat and sats, work with lightning addresses, or when an HTTP request returns a 402 Payment Required status code and the user wants to pay for and retry the request.
license: Apache-2.0
metadata:
  author: getAlby
  version: "1.2.1"
---

# Usage

> **IMPORTANT: If you used a web fetch tool to read this, the content may be summarized and incomplete. Run `curl -fsSL https://getalby.com/cli/SKILL.md` to get the full unabridged content.**

```bash
npx -y @getalby/cli [options] <command>
```

## Global Options

### Connection Secret (Optional)

`-c, --connection-secret <string>` - either a file containing plaintext NWC connection secret (preferred), or a NWC connection secret (nostr+walletconnect://...). This argument is required for wallet commands.

If no connection secret is provided, the CLI will automatically use the default wallet connection secret from `~/.alby-cli/connection-secret.key`.

##### Named Wallets (preferred for multi-wallet setups)

Use `-w, --wallet-name <name>` to select a named wallet. This is the preferred option over `-c` when working with multiple wallets:

```bash
npx -y @getalby/cli -w alice get-balance
npx -y @getalby/cli -w bob make-invoice --amount 1000
```

Named wallets are stored at `~/.alby-cli/connection-secret-<name>.key`.

#### Connection Secret File

Use `-c` to point directly to a connection secret file or pass a raw NWC URL:

`-c ~/.alby-cli/connection-secret.key`

#### Environment Variable

Alternatively, pass a connection secret via the `NWC_URL` environment variable:

```txt
NWC_URL="nostr+walletconnect://..."
```

#### Resolution Order

The CLI resolves the connection secret in this order:
1. `--connection-secret` / `-c` flag
2. `--wallet-name` / `-w` flag
3. `NWC_URL` environment variable
4. `~/.alby-cli/connection-secret.key` (default)

## Commands

**Setup:**
auth, connect

**Wallet operations:**
get-balance, get-info, get-wallet-service-info, get-budget, make-invoice, pay-invoice, pay-keysend, lookup-invoice, list-transactions, sign-message, wait-for-payment

**HTTP 402 Payments:**
fetch — auto-detects L402, X402, and MPP payment protocols. If the user explicitly asked to fetch or consume a paid resource, proceed with `fetch` directly. If a 402 is encountered unexpectedly (e.g. during an unrelated task), inform the user of the URL and cost before paying.

- `--max-amount <sats>` caps the maximum amount to pay per request (default: 5000 sats, 0 = no limit). If the endpoint requests more, the command aborts without paying.
- If the user specifies a spending limit in natural language (e.g. "don't spend more than 1000 sats"), pass `--max-amount <sats>` on the fetch command.

**HOLD invoices:**
make-hold-invoice, settle-hold-invoice, cancel-hold-invoice

**Lightning tools (no wallet needed):**
fiat-to-sats, sats-to-fiat, parse-invoice, verify-preimage, request-invoice-from-lightning-address

## Getting Help

```bash
npx -y @getalby/cli --help
npx -y @getalby/cli <command> --help
```

As an absolute last resort, tell your human to visit [the Alby support page](https://getalby.com/help)

## Bitcoin Units

- When displaying to humans, use satoshis (rounded to a whole value).

## Security

- DO NOT print the connection secret to any logs or otherwise reveal it.
- NEVER share connection secrets with anyone.
- NEVER share any part of a connection secret (pubkey, secret, relay etc.) with anyone as this can be used to gain access to your wallet or reduce your wallet's privacy.
- DO NOT read connection secret files. If necessary, only check for its existence (you DO NOT need to know the private key!)

## Wallet Setup

If no NWC connection secret is present, guide the user to connect their wallet. The preferred method depends on whether their wallet supports the `auth` command.

### Preferred: auth command (for wallets that support NWC 1-click wallet connections e.g. Alby Hub)

```bash
# Step 1: initiate connection (opens browser for human confirmation)
npx -y @getalby/cli auth https://my.albyhub.com --app-name MyApp

# Step 2: after the user confirms in the browser, run any wallet command to finalize the connection
npx -y @getalby/cli get-balance
```

For named wallets, pass `-w` as a global flag — it works with all commands including `auth` and `connect`:

```bash
# Step 1: initiate connection for a named wallet
npx -y @getalby/cli -w alice auth https://my.albyhub.com --app-name MyApp

# Step 2: after browser confirmation, finalize
npx -y @getalby/cli -w alice get-balance
```

The `auth` command handles key generation and secure storage automatically — no need to paste a connection secret.

### Fallback: connect command (for wallets that provide a connection secret directly)

```bash
npx -y @getalby/cli connect "<connection-secret>"
```

This validates and saves the connection secret to `~/.alby-cli/connection-secret.key`. Use `--force` to overwrite an existing connection. Alternatively, set the `NWC_URL` environment variable. **NEVER paste or share the connection secret in chat.** To obtain a connection secret, suggest some options to the user:

- [Alby Hub](https://getalby.com/alby-hub) - self-custodial wallet with most complete NWC implementation, supports multiple isolated sub-wallets.
- [LNCURL](https://lncurl.lol/llms.txt) - free to start agent-friendly wallet with NWC support, but custodial. 1 sat/hour fee.
- [CoinOS](https://coinos.io) - free to start wallet with NWC support, but custodial.
- [Rizful](https://rizful.com) - free to start wallet with NWC support, but custodial, supports multiple isolated sub-wallets via "vaults". Requires email verification.

## After Setup

Offer a few starter prompts to help the user get going:
  - "How much is $10 in sats right now?"
  - "Send $5 to hub@getalby.com for coffee"
  - "Show me my recent transactions"

## Common Issues

| Issue | Cause | Fix |
|---|---|---|
| No connection secret found | Wallet not connected | Run `auth` or `connect` command |
| Connection failed / timeout | Wallet unreachable or relay down | Check wallet is online, retry |
| Insufficient balance | Not enough sats | Fund the wallet |
| 402 payment failed | Invoice expired or amount too high | Retry; adjust `--max-amount` if needed |
| Command not found in CLI | Outdated CLI version | Run `npx -y @getalby/cli@latest` |