# Forex Trade Executor — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A private Telegram bot that automates forex trade execution based on strategy signals, sends real-time notifications to the admin, and provides manual order control. Handles order lifecycle events, daily summaries, and admin commands.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- forex trading admin

## Success criteria

- Admin receives real-time notifications for all trade executions and errors
- Manual commands execute confirmed orders with 100% accuracy
- Daily summaries are delivered at configured times

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: admin, command: /start) — Authenticate admin and confirm bot readiness
- **/status** (command, actor: admin, command: /status) — Show current open positions and trade status
- **/open** (command, actor: admin, command: /open) — Manually open a new trade with confirmation prompt
  - inputs: instrument, side, size
  - outputs: trade confirmation
- **/close** (command, actor: admin, command: /close) — Close an open position with confirmation
  - inputs: trade_id
  - outputs: closure confirmation
- **/summary** (command, actor: admin, command: /summary) — Trigger immediate P&L summary report
- **/setsummarytime** (command, actor: admin, command: /setsummarytime) — Configure daily summary time

## Flows

### signal_handling
_Trigger:_ strategy_signal_received

1. Validate signal parameters
2. Place order via broker API
3. Send order confirmation to admin
4. Monitor execution status

_Data touched:_ StrategySignal, TradeOrder

### order_lifecycle
_Trigger:_ execution_update

1. Receive broker execution report
2. Send fill/error notification
3. Update trade status
4. Send summary on closure

### daily_summary
_Trigger:_ scheduled_daily

1. Aggregate open positions
2. Calculate P&L
3. Format summary message
4. Send to admin

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Admin** _(retention: persistent)_ — Bot owner/administrator account
  - fields: telegram_chat_id, summary_time_utc
- **StrategySignal** _(retention: persistent)_ — Trading instruction from strategy
  - fields: pair, side, size, stop_loss, take_profit, timestamp
- **TradeOrder** _(retention: persistent)_ — Executed or pending trade
  - fields: instrument, side, size, price, status, open_time, close_time
- **ExecutionReport** _(retention: persistent)_ — Broker order execution details
  - fields: trade_id, fill_price, fill_time, error_message
- **AdminSettings** _(retention: persistent)_ — Configuration preferences
  - fields: daily_summary_time_utc

## Integrations

- **Telegram** (required) — Admin notifications and commands
- **Broker Trading API** (required) — Order execution and market data
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Authenticate admin chat ID
- Configure daily summary time
- View open positions
- Manually open/close trades
- Trigger immediate summary

## Notifications

- Order placement confirmation
- Fill execution report
- Error alerts with retry status
- Daily P&L summary at configured time

## Permissions & privacy

- All commands require admin authentication
- API keys stored securely
- Trade history retained indefinitely by default

## Edge cases

- Failed broker API authentication
- Invalid signal parameters
- Admin chat ID spoofing attempts
- Concurrent order execution conflicts

## Required tests

- End-to-end signal processing flow with error injection
- Daily summary delivery at midnight UTC
- Admin command confirmation workflow

## Assumptions

- Single admin account model
- Market orders default unless limit price specified
- 3 automatic retry attempts on API errors
