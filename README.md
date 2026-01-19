<img width="1200" alt="Labs" src="https://user-images.githubusercontent.com/99700157/213291931-5a822628-5b8a-4768-980d-65f324985d32.png">

<p>
 <h3 align="center">Chainstack is the leading suite of services connecting developers with Web3 infrastructure</h3>
</p>

<p align="center">
  • <a target="_blank" href="https://chainstack.com/">Homepage</a> •
  <a target="_blank" href="https://chainstack.com/protocols/">Supported protocols</a> •
  <a target="_blank" href="https://chainstack.com/blog/">Chainstack blog</a> •
  <a target="_blank" href="https://docs.chainstack.com/quickstart/">Blockchain API reference</a> • <br> 
  • <a target="_blank" href="https://console.chainstack.com/user/account/create">Start for free</a> •
</p>

The project allows you to create bots for trading on pump.fun and letsbonk.fun. Its core feature is to snipe new tokens. Besides that, learning examples contain a lot of useful scripts for different types of listeners (new tokens, migrations) and deep dive into calculations required for trading.

For the full walkthrough, see [Solana: Creating a trading and sniping pump.fun bot](https://docs.chainstack.com/docs/solana-creating-a-pumpfun-bot).

For near-instantaneous transaction propagation, you can use the [Chainstack Solana Trader nodes](https://docs.chainstack.com/docs/trader-nodes).

For instant updates from the network, you can enable [Yellowstone gRPC Geyser plugin](https://docs.chainstack.com/docs/yellowstone-grpc-geyser-plugin) (Jito ShredStream enabled by default).

The official maintainers are in the [MAINTAINERS.md](MAINTAINERS.md) file. Leave your feedback by opening **Issues**.

---

**🚨 SCAM ALERT**: Issues section is often targeted by scam bots willing to redirect you to an external resource and drain your funds. I have enabled a GitHub actions script to detect the common patterns and tag them, which obviously is not 100% accurate. This is also why you will see deleted comments in the issues—I only delete the scam bot comments targeting your private keys. Not everyone is a scammer though, sometimes there are helpful outside devs who comment and I absolutely appreciate it.

**⚠️ NOT FOR PRODUCTION**: This code is for learning purposes only. We assume no responsibility for the code or its usage. Modify for your needs and learn from it (examples, issues, and PRs contain valuable insights).

---


## 🚀 Getting started

### Prerequisites
- Install [uv](https://github.com/astral-sh/uv), a fast Python package manager.

> If Python is already installed, `uv` will detect and use it automatically.

### Installation

#### 1️⃣ Clone the repository
```bash
git clone https://github.com/chainstacklabs/pump-fun-bot.git
cd pump-fun-bot
```

#### 2️⃣ Set up a virtual environment
```bash
# Create virtual environment
uv sync
.venv\Scripts\activate
# Activate (Unix/macOS)
source .venv/bin/activate  

# Activate (Windows)
.venv\Scripts\activate
```
> Virtual environments help keep dependencies isolated and prevent conflicts.

#### 3️⃣ Configure the bot
```bash
# Copy example config
cp .env.example .env  # Unix/macOS

# Windows
copy .env.example .env
```
Edit the `.env` file and add your **Solana RPC endpoints** and **private key**.

Edit `.yaml` templates in the `bots/` directory. Each file is a separate instance of a trading bot. Examine its parameters and apply your preferred strategy.

For example, to run the pump.fun bot, set `platform: "pump_fun"`; to run the bonk.fun bot, set `platform: "lets_bonk"`.

#### 4️⃣ Install the bot as a package
```bash
uv pip install -e .
```
> **Why `-e` (editable mode)?** Lets you modify the code without reinstalling the package—useful for development!

### Running the bot

```bash
# Option 1: run as installed package
pump_bot

# Option 2: run directly
uv run src/bot_runner.py
```

> **You're all set! 🎉** 

---

## Note on throughput & limits

Solana is an amazing piece of web3 architecture, but it's also very complex to maintain.

Chainstack is daily (literally, including weekends) working on optimizing our Solana infrastructure to make it the best in the industry.

That said, all node providers have their own setup recommendations & limits, like method availability, requests per second (RPS), free and paid plan specific limitations and so on.

So please make sure you consult the docs of the node provider you are going to use for the bot here. And obviously the public RPC nodes won't work for the heavier use case scenarios like this bot.

For Chainstack, all of the details and limits you need to be aware of are consolidated here: [Throughput guidelines](https://docs.chainstack.com/docs/limits) <— we are _always_ keeping this piece up to date so you can rely on it.

## Changelog

Quick note on a couple on a few new scripts in `/learning-examples`:

*(this is basically a changelog now)*

Also, here's a quick doc: [Listening to pump.fun migrations to Raydium](https://docs.chainstack.com/docs/solana-listening-to-pumpfun-migrations-to-raydium)

## Bonding curve state check

`get_bonding_curve_status.py` — checks the state of the bonding curve associated with a token. When the bonding curve state is completed, the token is migrated to Raydium.

To run:

`uv run learning-examples/bonding-curve-progress/get_bonding_curve_status.py TOKEN_ADDRESS`

## Listening to the Pump AMM migration

When the bonding curve state completes, the liquidity and the token graduate to Pump AMM (PumpSwap).

`listen_logsubscribe.py` — listens to the migration events of the tokens from bonding curves to AMM and prints the signature of the migration, the token address, and the liquidity pool address on Pump AMM.

`listen_blocksubscribe_old_raydium.py` — listens to the migration events of the tokens from bonding curves to AMM and prints the signature of the migration, the token address, and the liquidity pool address on Pump AMM (previously, tokens migrated to Raydium).

Note that it's using the [blockSubscribe]([url](https://docs.chainstack.com/reference/blocksubscribe-solana)) method that not all providers support, but Chainstack does and I (although obviously biased) found it pretty reliable.

To run:

`uv run learning-examples/listen-migrations/listen_logsubscribe.py`

`uv run learning-examples/listen-migrations/listen_blocksubscribe_old_raydium.py`

**The following two new additions are based on this question [associatedBondingCurve #26](https://github.com/chainstacklabs/pump-fun-bot/issues/26)**

You can take the compute the associatedBondingCurve address following the [Solana docs PDA](https://solana.com/docs/core/pda) description logic. Take the following as input *as seed* (order seems to matter):

- bondingCurve address
- the Solana system token program address: `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA`
- the token mint address

And compute against the Solana system associated token account program address: `ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL`.

The implications of this are kinda huge:
* you can now use `logsSubscribe` to snipe the tokens and you are not limited to the `blockSubscribe` method
* see which one is faster
* not every provider supports `blockSubscribe` on lower tier plans or at all, but everyone supports `logsSubscribe`

The following script showcase the implementation.

## Compute associated bonding curve

`compute_associated_bonding_curve.py` — computes the associated bonding curve for a given token.    

To run:

`uv run learning-examples/compute_associated_bonding_curve.py` and then enter the token mint address.

## Listen to new tokens

`listen_logsubscribe_abc.py` — listens to new tokens and prints the signature, the token address, the user, the bonding curve address, and the associated bonding curve address using just the `logsSubscribe` method. Basically everything you need for sniping using just `logsSubscribe` (with some [limitations](https://github.com/chainstacklabs/pump-fun-bot/issues/87)) and no extra calls like doing `getTransaction` to get the missing data. It's just computed on the fly now.

To run:

`uv run learning-examples/listen-new-tokens/listen_logsubscribe_abc.py`

So now you can run `compare_listeners.py` see which one is faster.

`uv run learning-examples/listen-new-tokens/compare_listeners.py`

Also here's a doc on this: [Solana: Listening to pump.fun token mint using only logsSubscribe](https://docs.chainstack.com/docs/solana-listening-to-pumpfun-token-mint-using-only-logssubscribe)

---

# Pump.fun bot development roadmap (March - April 2025, mostly completed)

~~As of March 21, 2025, the bot from the **refactored/main-v2** branch is signficantly better over the **main** version, so the suggestion is to FAFO with v2.~~

As of April 30, 2025, all changes from **refactored/main-v2** are merged into the **main** version.

| Stage | Feature | Comments | Implementation status
|-------|---------|----------|---------------------|
| **Stage 1: General updates & QoL** | Lib updates | Updating to the latest libraries | ✅ |
| | Error handling | Improving error handling | ✅ | 
| | Configurable RPS | Ability to set RPS in the config to match your provider's and plan RPS (preferably [Chainstack](https://console.chainstack.com/) 🤩) | Not started |
| | Dynamic priority fees | Ability to set dynamic priority fees | ✅ |
| | Review & optimize `json`, `jsonParsed`, `base64` | Improve speed and traffic for calls, not just `getBlock`. [Helpful overview](https://docs.chainstack.com/docs/solana-optimize-your-getblock-performance#json-jsonparsed-base58-base64).| ✅ | 
| **Stage 2: Bonding curve and migration management** | `logsSubscribe` integration | Integrate `logsSubscribe` instead of `blockSubscribe` for sniping minted tokens into the main bot | ✅ |
| | Dual subscription methods | Keep both `logsSubscribe` & `blockSubscribe` in the main bot for flexibility and adapting to Solana node architecture changes | ✅ |
| | Transaction retries | Do retries instead of cooldown and/or keep the cooldown | ✅ |
| | Bonding curve status tracking | Checking a bonding curve status progress. Predict how soon a token will start the migration process | ✅ | 
| | Account closure script | Script to close the associated bonding curve account if the rest of the flow txs fails | ✅ |
| | PumpSwap migration listening | pump_fun migrated to their own DEX — [PumpSwap](https://x.com/pumpdotfun/status/1902762309950292010), so we need to FAFO with that instead of Raydium (and attempt `logSubscribe` implementation) | ✅ |
| **Stage 3: Trading experience** | Take profit/stop loss | Implement take profit, stop loss exit strategies | ✅ |
| | Market cap-based selling | Sell when a specific market cap has been reached | Not started |
| | Copy trading | Enable copy trading functionality | Not started |
| | Token analysis script | Script for basic token analysis (market cap, creator investment, liquidity, token age) | Not started |
| | Archive node integration | Use Solana archive nodes for historical analysis (accounts that consistently print tokens, average mint to raydium time) | Not started |
| | Geyser implementation | Leverage Solana Geyser for real-time data stream processing | ✅ |
| **Stage 4: Minting experience** | Token minting | Ability to mint tokens (based on user request - someone minted 18k tokens) | ✅ |


## Development timeline
- Development begins: Week of March 10, 2025
- Implementation approach: Gradual rollout in separate branch
- Priority: Stages progress from simple to complex features
- Completion guarantee: Full completion of Stage 1, other stages dependent on feedback and feasibility

 uv sync

 >.venv\Scripts\activate

 uv run src/bot_runner.py

 
• Here’s every command I ran via the terminal in this session, in
  order:

  Get-ChildItem -Force
  Get-Content README.md
  rg --files -g 'pyproject.toml' -g 'uv.lock'
  Get-Content pyproject.toml
  uv sync
  uv sync
  uv pip install -e .
  pump_bot
  Get-Content src\bot_runner.py
  Get-Content src\trading\universal_trader.py
  uv run src\bot_runner.py
  Get-ChildItem logs | Sort-Object LastWriteTime -Descending | Select-
  Object -First 5
  Get-Content logs\bot-sniper-2_20260115_121159.log
  rg -n "[\u2705\u274c\U0001f50d\U0001f4dd\U0001f511]" src
  Get-Content src\utils\logger.py
  rg -n "token_info\.name|token_info\.symbol|Token found:" src
  Get-Content src\monitoring\universal_logs_listener.py
  rg -n "[^\x00-\x7F]" src
  Get-Content CLAUDE.md
  Get-Content logs\bot-sniper-2_*.log -Wait

    1. Activate the venv:

  .venv\Scripts\activate

  2. Run the bot:

  uv run src/bot_runner.py


   Directory of C:\Users\marti\Desktop\openai\pumpfun-bonkfun-bot-main\learning-examples\bonding-curve-progress

01/15/2026  10:45 AM    <DIR>          .
01/15/2026  10:45 AM    <DIR>          ..
01/15/2026  10:45 AM             7,129 get_bonding_curve_status.py
01/15/2026  10:45 AM             5,671 get_graduating_tokens.py
01/15/2026  10:45 AM             5,250 poll_bonding_curve_progress.py
               3 File(s)         18,050 bytes
               2 Dir(s)  67,351,740,416 bytes free

C:\Users\marti\Desktop\openai\pumpfun-bonkfun-bot-main\learning-examples\bonding-curve-progress>py get_bonding_curve_status.py

Token status:
--------------------------------------------------
Token mint:              5LCVhWUh2KZd1wZ7MefZGCjF3tvRsqMJjscJSAW9pump
Bonding curve:           C6GgbN948q2NzQpAXeBkA7raG8BPtSvt3Y9dmaJJctLR
Bump seed:               255
--------------------------------------------------

Bonding curve status:
--------------------------------------------------
Creator:             7hbvz77YRL8THaeH35bwQkpk9uommFMocJscuYBbZCKE
Mayhem Mode:         ❌ Disabled
Completed:           ✅ Migrated

Bonding curve reserves:
Virtual Token:       0
Virtual SOL:         0 lamports
Real Token:          0
Real SOL:            0 lamports
Total Supply:        1,000,000,000,000,000

Note: This bonding curve has completed and liquidity has been migrated to PumpSwap.


C:\Users\marti\Desktop\openai\pumpfun-bonkfun-bot-main\learning-examples>uv sync
Resolved 36 packages in 7ms
Audited 34 packages in 660ms

C:\Users\marti\Desktop\openai\pumpfun-bonkfun-bot-main\learning-examples>uv run learning-examples/mint_and_buy_v2.py
error: Failed to spawn: `learning-examples/mint_and_buy_v2.py`
  Caused by: The system cannot find the path specified. (os error 3)

C:\Users\marti\Desktop\openai\pumpfun-bonkfun-bot-main\learning-examples>uv run mint_and_buy_v2.py
Creating Token2022 token with:
  Name: Test Token V2
  Symbol: TEST2
  Mint: 93tmBtdw8aUUxLY81F68BRXKm9diUL6QZ2CkJDSwGmhW
  Creator: EyU98DxvB4Rt1Hia64wyTHCnUKckmyuv4oUfnB5rYFRv
  Mayhem mode: Enabled

Derived addresses:
  Bonding curve: 9UE8Vuk9cPdXFGtJdGDhb9Fotks7Bhfk1CtuVaTLaX3r
  Associated bonding curve: CN7nVJPpWjMrJHc36UHR15hiKb23VTHzUq1GfzMd4JCr
  User ATA: 2urW59o7j75yDYP9AZCGTqzvnpuZBVKbNYHJrNnyeC9s
  Creator vault: JDebwmsHcQ9KSfTLxBHqgfgZPLef7EwME1aTtwexd892
  Mayhem state: 6cHir4tQhZ1GJTqERLaeEsdTQnrvqmDv7kKpMKAiKnvV
  Mayhem token vault: BgjD8rusYXjZSRtFpUCGfuJRVybouxGwF8yov6RZqstF

Buy parameters:
  Buy amount: 1e-05 SOL
  Expected tokens: 354.090000
  Max SOL cost: 0.000013 SOL
Using mayhem mode fee recipient: GesfTA3X2arioaHp8bbKdjG9vJtskViWACZoYvxp4twS

Sending transaction...
Transaction sent: https://solscan.io/tx/cjpX2R1j4dKZonE6LtM3csf1VxhCqd5C2Xj9KP6SiGSuW2N1U32cnUsPyAhdAtBXCysFfqRJQr8kQHPTZeBYmKm
Waiting for confirmation...
Transaction confirmed!

C:\Users\marti\Desktop\openai\pumpfun-bonkfun-bot-main\learning-examples>uv run manual_buy.py
Waiting for a new token creation...
Subscribed to blocks mentioning program: 6EF8rrecthR5Dkzon8Nwu78hRvfCKubJ14M5uBEwF6P
New token created:
{
  "name": "Claude",
  "symbol": "CLAUDE",
  "uri": "https://ipfs.io/ipfs/bafkreihyiuq2g2ntppf2o3ookobz5d4typaal34iuf6ueappquywcoupke",
  "creator": "HkcRQUdJor2vmcSCNz33cWvNzy5eEzLjBywofLBpbnAE",
  "is_mayhem_mode": false,
  "mint": "BQZLpvLoPbLuXx3XTASsRn1LKaM2rniww3Yr44V1pump",
  "bondingCurve": "Hq7HKDdNySbTmaf3HxJpwenCdNz9cfHFLkraZ3mz2BRE",
  "associatedBondingCurve": "9Q7W6xd1t7CC54mdBvetrfs7H6ZPaNdLN8ro632236Ei",
  "user": "TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb",
  "token_program": "TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb",
  "is_token_2022": true
}
Waiting for 15 seconds for things to stabilize...
Bonding curve address: Hq7HKDdNySbTmaf3HxJpwenCdNz9cfHFLkraZ3mz2BRE
Token Program: TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb (Token2022)
Token price: 0.0000000280 SOL
Buying 0.000001 SOL worth of the new token with 30.0% slippage tolerance...
Transaction sent: https://explorer.solana.com/tx/5cr6yVhrJCNqSj6RoPLnmuvVM9JdraE4EZxDDSDV4y4LdBx5xwCzKAacrrJJBFEdW1J68tNpMg2h31QAYqMsoyX8
Transaction confirmed




C:\Users\marti\Desktop\openai\pumpfun-bonkfun-bot-main\learning-examples\listen-new-tokens>py listen_pumpportal.py
Listening for new token creations...

================================================================================
🎯 NEW TOKEN DETECTED (via PumpPortal)
================================================================================
Name:             Grandpa Steve
Symbol:           Grandpa
Mint:             89Uzp7NfrNLqu9qkTjL8mtLXJ8EmttVD5brWVCdgpump
Initial Buy:      35323.892708 SOL
Market Cap:       27.960834 SOL
Bonding Curve:    7sQjxPsdWHkxuUBAsWQcpjy38h1ZJD9xeBNWDttuh1jw
Creator:          6ZZNCUK7VTbghVFiVsg3ksTdWhkXNAMtnH6df5u3EGeW
Virtual SOL:      30.000988 SOL
Virtual Tokens:   1,072,964,676
URI:              https://ipfs.io/ipfs/bafkreibihvfjaicqtbe4sbpuosfn5uay7spr7bhlaswnq3rcyags2vppxe
Signature:        5TnUksQFw2oDCgmRwQLf5Qgd9EcXbpFYPjE8uEQyfW3WWq7YudYm4XHs2aY3gFc3L1ekWa6bNFiezMhwEPxptGp6
================================================================================
(pump-bot) C:\Users\marti\Desktop\openai\pumpfun-bonkfun-bot-main\learning-examples>uv run manual_buy.py



in\learning-examples\listen-new-tokens>uv run listen_pumpportal.py
Listening for new token creations...

================================================================================
🎯 NEW TOKEN DETECTED (via PumpPortal)
================================================================================
Name:             TESTICLE
Symbol:           $TESTICLE
Mint:             2mwVDhNAh45R2Zgq4NVmzUwT4X8ae6EMwHtCUxo6pump
Initial Buy:      3867868.413475 SOL
Market Cap:       28.161658 SOL
Bonding Curve:    4wXheNzmomyDDBTQafAFtSCC5YpqSawa1Qp5sYP7kQfT
Creator:          HuEYTtAUvTMxCLEixvfSHtYJAffvNWGQAh7P7jawZRF3
Virtual SOL:      30.108533 SOL
Virtual Tokens:   1,069,132,132
URI:              https://ipfs.io/ipfs/QmNWhAEXDdzDLdbUCuk8chWgSHmjpPTWudSnsJ9ErCs38y
Signature:        5pGKMfXbrGkKe4Dx9B86FfYKV3Nqf5WJ3tKuemPHWoNDsnWVPdZzTXnik88ksjsrBQaDYkCapcUrrRLnUuSmcB6n
=======================