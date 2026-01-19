## Task Brief 1 - Align bot-sniper-2aa With learning-examples/manual_buy.py

Goal
Tune `bot-sniper-2aa` to mirror the small-amount, conservative behavior used in
`learning-examples/manual_buy.py` for safer testing.

Scope
- bots/bot-sniper-2aa-logs.yaml only (config changes)

Constraints
- Do not modify listeners, scanners, or core Solana client code.
- Do not change existing scoring logic.
- Keep changes limited to `bot-sniper-2aa` so other bots remain unaffected.

Changes Applied (Current State)
- `priority_fees.fixed_amount` and `hard_cap`: 900_000
- `trade.buy_amount`: 0.000008
- `trade.sell_slippage`: 0.15
- `filters.max_token_age`: 3
- `filters.max_top_holder_pct`: 90.0
- `retries.wait_after_creation`: 20
- `retries.wait_after_buy`: 10

Non-Goals
- No YAML restructuring beyond these values
- No code changes
- No changes to other bots in `bots/`

Success Criteria
- bot-sniper-2aa completes a buy/sell cycle using the updated config.
- Logs confirm the new timing and slippage values.

Testing (Remaining)
- Run bot-sniper-2aa again and confirm a full trade with the updated values.

## Task Brief 2 - Manual Buy/Sell Test Run (learning-examples)

Goal
Run `learning-examples/manual_buy.py` twice to validate the end-to-end flow
(buy + auto-sell) with the default test amount.

Purpose
Determine proper fees and params for the least expensive and most profitable
outcome incrementally as we learn.

Scope
- @learning-examples/manual_buy.py
- @learning-examples/manual_sell.py (called by manual_buy)

Constraints
- Use the default test buy amount (`0.000001` SOL).
- Do not run the main bot.
- Stop and report if RPC rate limits (HTTP 429) or other errors occur.
- Ensure `notesAI.txt` is updated with conclusions and results after each test run.
- Record trade outcomes in `notesAI.txt` (tx signature, mint, net SOL).

Success Criteria
- Two consecutive runs complete buy + auto-sell without errors.
- Console output captured and summarized (buy/sell tx signatures, timing).

Testing
- Run `uv run learning-examples/manual_buy.py` twice, one after the other.

## Task Brief 3 - Run bot-sniper-2 With Updated Config

Goal
Run `bot-sniper-2` (logs config) with the updated parameters and observe a full
buy/sell cycle for timing and fee evaluation.

Scope
- bots/bot-sniper-2-logs.yaml (already updated)
- bot runtime via `pump_bot`

Constraints
- Ensure only `bot-sniper-2` is enabled before running.
- Do not run other bots.
- Stop and report if RPC rate limits (HTTP 429) or other errors occur.
- Update `notesAI.txt` with conclusions and results after the run.

Success Criteria
- bot-sniper-2 completes one buy/sell cycle with the updated config.
- Logs captured and summarized (timing, fees, tx signatures).

Testing
- Run `uv run pump_bot` and monitor `logs/bot-sniper-2_*.log`.

## Task Brief 4 - Trend Scan With PumpPortal (learning-examples)

Goal
Use `learning-examples/listen-new-tokens/listen_pumpportal.py` to observe
new token trends and record recurring words/themes to inform metadata choices
for `learning-examples/mint_and_buy_v2.py`.

Scope
- learning-examples/listen-new-tokens/listen_pumpportal.py
- notesAI.txt (trend notes)

Constraints
- Do not run the main bot.
- Use PumpPortal listener only.
- Update `notesAI.txt` with observed trend words/themes and timestamps.

Success Criteria
- Trend words/themes captured in notesAI.txt.
- Findings referenced when choosing token name/symbol/URI for mint_and_buy_v2.py.

Testing
- Run `uv run learning-examples/listen-new-tokens/listen_pumpportal.py` and
  capture 10+ new token events before summarizing trends.

## Task Brief 5 - Mint and Buy Token Based on Trend Findings

Goal
Create and buy a new token using `learning-examples/mint_and_buy_v2.py` with
name/symbol/URI derived from Task 4 trend findings.

Scope
- learning-examples/mint_and_buy_v2.py
- notesAI.txt (record final metadata and result)

Constraints
- Use the default test buy amount (`0.000001` SOL).
- Do not run the main bot.
- Base name/symbol/theme on Task 4 trend notes.
- Update notesAI.txt with chosen metadata and tx signature.

Success Criteria
- Mint and buy succeeds with trend-based metadata.
- notesAI.txt captures chosen name/symbol/URI and tx signature.

Testing
- Run `uv run learning-examples/mint_and_buy_v2.py` and confirm the tx.

## Task Brief 6 - Trend Match Bot (bot-sniper-3a)

Goal
Create a trend-matching bot config that filters new tokens using the recurring
words/themes from Task 4, so we can observe which trends convert best and use
them when minting tokens.

Scope
- bots/bot-sniper-3a-logs.yaml (new config)
- notesAI.txt (record match outcomes)

Constraints
- Use `match_string` derived from Task 4 trends.
- Keep other settings conservative and aligned with bot-sniper-2aa.
- Do not run multiple bots at once.
- Update notesAI.txt after each run with matched tokens and outcomes.

Success Criteria
- bot-sniper-3a runs with trend-based `match_string`.
- Logs show matched tokens and buy/sell outcomes.

Testing
- Run bot-sniper-3a via `pump_bot` and monitor `logs/bot-sniper-3a_*.log`.

## Task Brief 7 - Hourly Rotation Test (Pump.fun bots)

Goal
Run one Pump.fun bot at a time for up to 10 minutes; if it does not complete a
buy+sell within that window, stop it and move to the next bot. Repeat for the
selected set over the next hour and summarize outcomes.

Scope
- Pump.fun bots only (select ~6 configs)
- logs/ (review buy/sell outcomes)
- notesAI.txt (record results and adjustments)

Constraints
- Run only one bot at a time.
- If no buy+sell within 10 minutes, stop and proceed to next bot.
- Do not shorten timeouts just to finish within 10 minutes.
- Only adjust configs based on outcome quality (fills, slippage, net result).
- Record results in notesAI.txt after each bot window (bots do not auto-log).

Success Criteria
- Each selected bot is given a 10-minute window.
- notesAI.txt captures: bot name, result (buy+sell or no trade), tx signatures,
  and any follow-up adjustments.

Testing
- Run `uv run pump_bot` and stop after 10 minutes if no buy+sell.

Rotation Order (Pump.fun)
1) bot-sniper-2-logs.yaml
2) bot-sniper-2aa-logs.yaml
3) bot-sniper-3a-trends.yaml
4) bot-sniper-7.yaml
5) bot-sniper-5-logsUpdate.yaml
6) bot-sniper-6gpt.yaml

Excluded from rotation
- bot-sniper-2a-logs.yaml (no top-holder filter; noisier data)
- bot-sniper-3-blocks.yaml (extreme_fast + blocks; more aggressive)

## Task Brief 8 - bot-sniper-2a Fee Reduction Test

Goal
Run `bot-sniper-2a` with reduced priority fees to see if lower fees improve
net outcomes on small trades while keeping other params unchanged.

Changes Applied (Current State)
- `priority_fees.fixed_amount`: 200_000 (original 1_000_000 commented)
- `priority_fees.hard_cap`: 200_000 (original 1_000_000 commented)

Scope
- bots/bot-sniper-2a-logs.yaml only
- logs/ and notesAI.txt for results

Constraints
- Do not modify other parameters for this task.
- Run only one bot at a time.
- Stop after 10 minutes if no buy+sell occurs.
- Log results in notesAI.txt (buy/sell txs, net effect, and any errors).

Success Criteria
- bot-sniper-2a completes at least one buy+sell within the 10-minute window, or
  a clear "no trade" outcome is recorded.

Current Trend Match (Applied)
- match_string: "bear"

## Task Brief 9 - Lower-Fee bot-sniper-3 Run and Profitability Check

Goal
Reduce per-trade fees for `bot-sniper-3` (lowerfees config) and verify whether
profit/loss is driven by price movement rather than priority fees.

Scope
- bots/bot-sniper-3-lowerfees.yaml (priority fee baseline)
- bots/bot-sniper-3-lowerfeeshold.yaml (tp/sl exit strategy)
- bots/bot-sniper-3-blocks.yaml (kept disabled)
- bot runtime via `pump_bot`
- logs/ and notesAI.txt for results

Constraints
- Ensure only `bot-sniper-3-lowerfees` is enabled before running.
- Do not run other bots.
- Stop and report if RPC rate limits (HTTP 429) or other errors occur.
- Update `notesAI.txt` with conclusions and results after each run, including
  Solscan fee lines when available.
 - When testing TP/SL, use take_profit=0.18, stop_loss=0.10, price_check=5s.

Success Criteria
- bot-sniper-3 lowerfees completes buy/sell cycles with priority fee = 0.
- Logs captured and summarized (timing, fees, tx signatures).
- Fees reduced enough that PnL reflects price movement rather than fees alone.

Testing
- Run `uv run pump_bot` and monitor `logs/bot-sniper-3_*.log`.
