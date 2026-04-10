---
name: alby-bitcoin-payments
description: teaches agents how to use @getalby/cli for bitcoin lightning wallet operations using Nostr Wallet Connect (NIP-47). Use when the user needs to send/receive bitcoin payments, check wallet balance, create invoices, convert between fiat and sats, work with lightning addresses, when an HTTP request returns a 402 Payment Required status code and the user wants to pay for and retry the request, or discover paid API services.
license: MIT-0
metadata:
  author: getAlby
  version: "1.2.3"
  openclaw:
    requires:
      env:
        - NWC_URL
      bins:
        - npx
      config:
        - ~/.alby-cli/
    primaryEnv: NWC_URL
    emoji: "🐝"
    homepage: https://getalby.com
---

# Usage

> **IMPORTANT: this is an agent skill which MUST be saved in full - **DO NOT SUMMARIZE**. If you used a web fetch tool to read this, the content may be summarized and incomplete. Make sure you fetch the full unabridged content.**

```bash
npx -y @getalby/cli@0.6.1 [options] <command>
```

## Global Options

### Connection Secret (Optional)

`-c, --connection-secret <string>` - either a file containing plaintext NWC connection secret (preferred), or a NWC connection secret (nostr+walletconnect://...). This argument is required for wallet commands.

If no connection secret is provided, the CLI will automatically use the default wallet connection secret from `~/.alby-cli/connection-secret.key`.

##### Named Wallets (preferred for multi-wallet setups)

Use `-w, --wallet-name <name>` to select a named wallet. This is the preferred option over `-c` when working with multiple wallets:

```bash
npx -y @getalby/cli@0.6.1 -w alice get-balance
npx -y @getalby/cli@0.6.1 -w bob make-invoice --amount 1000
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

**Service Discovery (no wallet needed):**
discover

**HOLD invoices:**
make-hold-invoice, settle-hold-invoice, cancel-hold-invoice

**Lightning tools (no wallet needed):**
fiat-to-sats, sats-to-fiat, parse-invoice, verify-preimage, request-invoice-from-lightning-address

## Getting Help

```bash
npx -y @getalby/cli@0.6.1 --help
npx -y @getalby/cli@0.6.1 <command> --help
```

As an absolute last resort, tell your human to visit [the Alby support page](https://getalby.com/help)

## Discovering Paid Services

The `discover` command searches [402index.io](https://402index.io) for lightning-payable API endpoints. It only returns services that accept bitcoin/lightning payments.

```bash
npx -y @getalby/cli@0.6.1 discover -q "image generation"          # search by query
npx -y @getalby/cli@0.6.1 discover -q "podcast" --limit 20        # more results
```

Options: `-q` (search query), `-s` (sort: reliability, latency, price, name), `-l` (limit, default: 10).

### When to use discover

- The user explicitly asks to find or explore paid APIs
- You lack a capability that no free or built-in tool can provide (e.g. image generation, specialized inference, real-time data feeds)

### When NOT to use discover

- **Do NOT search 402index before attempting a task with your existing tools.** Try free/built-in approaches first.
- **Do NOT use discover as a replacement for standard web requests.** If `curl`, `fetch`, or WebFetch works, use that instead.
- **Do NOT use discover when you already have a URL.** Just use the `fetch` command directly.

### Discover → Fetch flow

1. **Discover** — find services matching the capability gap
2. **Evaluate** — check price, health status, and reliability from the results
3. **Fetch** — pay and consume the service:
   ```bash
   npx -y @getalby/cli@0.6.1 fetch -X POST -b '{"model":"gpt-image-1","prompt":"a mountain cabin at sunset","size":"1024x1024"}' "<service-url>"
   ```
4. **Report** — tell the user what was purchased, the cost, and the result

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
npx -y @getalby/cli@0.6.1 auth https://my.albyhub.com --app-name MyApp

# Step 2: after the user confirms in the browser, run any wallet command to finalize the connection
npx -y @getalby/cli@0.6.1 get-balance
```

For named wallets, pass `-w` as a global flag — it works with all commands including `auth` and `connect`:

```bash
# Step 1: initiate connection for a named wallet
npx -y @getalby/cli@0.6.1 -w alice auth https://my.albyhub.com --app-name MyApp

# Step 2: after browser confirmation, finalize
npx -y @getalby/cli@0.6.1 -w alice get-balance
```

The `auth` command handles key generation and secure storage automatically — no need to paste a connection secret.

### Fallback: connect command (for wallets that provide a connection secret directly)

```bash
npx -y @getalby/cli@0.6.1 connect "<connection-secret>"
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