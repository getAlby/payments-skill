---
name: alby-bitcoin-payments
description: teaches agents how to use @getalby/cli for bitcoin lightning wallet operations using Nostr Wallet Connect (NIP-47). Use when the user needs to send/receive payments, check wallet balance, create invoices, convert between fiat and sats, or work with lightning addresses.
license: Apache-2.0
metadata:
  author: getAlby
  version: "1.1.3"
---

# Usage

```bash
npx -y @getalby/cli [options] <command>
```

## Global Options

### Connection Secret (Optional)

`-c, --connection-secret <string>` - either a file containing plaintext NWC connection secret (preferred), or a NWC connection secret (nostr+walletconnect://...). This argument is required for wallet commands.

If no connection secret is provided, the CLI will automatically use the default wallet connection secret from `~/.alby-cli/connection-secret.key`.

#### Connection Secret File Location

Simply point `-c` directly to the file:

`-c ~/.alby-cli/connection-secret.key`

If a user wants to use a specific wallet e.g. "alice", use the path instead:

`-c ~/.alby-cli/connection-secret-alice.key`

#### Environment Variable

Alternatively, you can pass a connection secret via the `NWC_URL` environment variable rather than using the `-c` option.

```txt
NWC_URL="nostr+walletconnect://..."
```

## Commands

**Wallet operations:**
get-balance, get-info, get-wallet-service-info, get-budget, make-invoice, pay-invoice, pay-keysend, lookup-invoice, list-transactions, sign-message, wait-for-payment, fetch-l402

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

- Do NOT print the connection secret to any logs or otherwise reveal it.
- NEVER share connection secrets with anyone.
- NEVER share any part of a connection secret (pubkey, secret, relay etc.) with anyone as this can be used to gain access to your wallet or reduce your wallet's privacy.

## Wallet Setup

If no NWC connection secret is present, configure it locally by writing it to `~/.alby-cli/connection-secret.key` or by setting the environment variable `NWC_URL`. **NEVER paste or share the connection secret in chat.** To obtain a connection secret, suggest some options to the user:

- [Alby Hub](https://getalby.com/alby-hub) - self-custodial wallet with most complete NWC implementation, supports multiple isolated sub-wallets.
- [LNCURL](https://lncurl.lol/llms.txt) - free to start agent-friendly wallet with NWC support, but custodial. 1 sat/hour fee.
- [CoinOS](https://coinos.io) - free to start wallet with NWC support, but custodial.
- [Rizful](https://rizful.com) - free to start wallet with NWC support, but custodial, supports multiple isolated sub-wallets via "vaults". Requires email verification.