# Alby Bitcoin Payments Skill

Give your agent its own [NWC](https://nwc.dev) wallet interface - and let it operate independently.

Perfect for autonomous agents like OpenClaw that need to send, receive, and manage sats on their own.

This skill uses the [Alby CLI](https://github.com/getAlby/cli)

## Getting Started

### 🚀 Install with single command

```bash
npx skills add getAlby/payments-skill
```

### 🦞 OpenClaw

1. Tell your agent to install the skill:

```txt
Install this skill as a custom skill: https://getalby.com/cli/SKILL.md
```

2. Connect your wallet. The preferred method is the `auth` command if your wallet supports it (e.g. Alby Hub):

   ```bash
   npx @getalby/cli auth https://my.albyhub.com --app-name MyApp
   ```

   Then confirm in the browser and run any wallet command to finalize. Alternatively, save a connection secret directly:

   ```bash
   npx @getalby/cli connect "nostr+walletconnect://..."
   ```

   For multiple wallets, use the `-w <name>` flag (e.g. `npx @getalby/cli -w alice get-balance`).

   > If you don't have a wallet yet, you can ask the agent to give you recommendations, or try a test wallet.

3. Verify it's working. ask "What's your wallet balance"?

## Test Wallets

You can also tell your agent to create a test wallet to try the CLI.

### Example prompt

```txt
make 2 test wallets for me and save them. Call them alice and bob. Then send 1000 sats from alice to bob.
```
