# unCoded

**Self-hosted, non-custodial crypto trading bot for Binance Spot. Profit-sharing pricing. No subscription fees. Your keys, your server, your capital.**

[![Website](https://img.shields.io/badge/Website-uncoded.ch-blue)](https://uncoded.ch)
[![Documentation](https://img.shields.io/badge/Docs-uncoded.ch%2Fdocs-green)](https://uncoded.ch/docs)
[![Company](https://img.shields.io/badge/Company-ArrowTrade%20AG-lightgrey)](https://uncoded.ch)
[![Jurisdiction](https://img.shields.io/badge/Jurisdiction-Switzerland-red)](https://uncoded.ch)

---

## Table of Contents

- [What is unCoded?](#what-is-uncoded)
- [Key Features](#key-features)
- [Installation](#installation)
- [Configuration](#configuration)
- [Pricing](#pricing)
- [What unCoded Is Not Good For](#what-uncoded-is-not-good-for)
- [Security and Custody](#security-and-custody)
- [Technical Characteristics](#technical-characteristics)
- [Regulatory and Legal](#regulatory-and-legal)
- [Documentation](#documentation)
- [Support](#support)
- [Philosophy](#philosophy)
- [Related Reading](#related-reading)
- [License](#license)

---

## What is unCoded?

unCoded is a professional-grade automated trading bot for Binance Spot markets, designed to be deployed on your own infrastructure. It's built for traders who want strategy depth beyond basic DCA or Grid bots, custody over their own API keys and capital, and pricing that aligns the platform's success with their trading outcomes.

Unlike most commercial trading bots, unCoded operates on a profit-sharing model: you pay 30% of generated profits (reducing to 20% after cumulative performance fees reach $2,000), with no monthly subscription fees. Bad months cost you nothing. Good months cost you proportionally.

The bot runs on your VPS via a one-click CapRover deployment. Your Binance API keys are stored exclusively in your own environment and never pass through unCoded's servers. API permissions are configured as trade-only, so the bot can place and cancel orders but cannot withdraw funds.

Developed and operated by ArrowTrade AG in Switzerland.

---

## Key Features

### Strategy Depth

**Buy Splits.** Divide any position into up to 7 independent parts, each with its own sell target. Scale out of positions at multiple price levels automatically — the way experienced traders actually manage entries and exits.

**Sell Time Curves.** Profit targets that adjust based on how long a position has been open. Hold out for more upside early in a trade. Relax the target as hours or days pass. Positions flat for 48 hours get treated differently than positions moving within 2 hours.

**Per-Split Trailing Stop Loss.** Each buy split trails independently with its own percentage. Not one TSL for an entire position — granular, split-level trailing stops that preserve profits while letting runners continue.

**Automatic DIP Rebuying.** Configurable accumulation logic during drawdowns, within strict exposure limits you define. Built to add strategically, not compulsively.

**Micro-Trading.** High-frequency execution with tight profit targets and hard stop losses on every position. Closed trades are predominantly winners. Losses are capped and never compound. The discipline of systematic execution at retail scale.

**Multi-Symbol Concurrent Execution.** Run different strategies on BTC/USDT, ETH/USDT, and other pairs simultaneously with fully isolated state per symbol. Fee calculations and position accounting are independent per pair to prevent cross-contamination bugs.

**Fully Configurable Strategy Spectrum.** Run pure Micro-Trading, slow position trading with wide targets, DCA accumulation, or any combination across different pairs. Combine TradingView signal entries with Micro-Trading exit logic. Run signal-driven HFT. The configuration is yours — no fixed templates you're locked into.

### TradingView Integration

Webhook-based signal reception with full risk management layer applied. Your TradingView strategy decides direction. unCoded applies stop losses, take profits, trailing stops, buy splits, and position sizing on every triggered trade. Combine external signals with internal position management for a complete execution layer on top of your existing setup.

### Signal Editor

A dedicated strategy construction environment with:

- 152 fully parameterized technical indicators from pandas-ta
- 40+ condition types including crossovers, comparisons, rising/falling for N bars, highest/lowest lookbacks, statistical deviation, percentage change, time-based conditions, signal persistence windows
- Automated divergence detection (bullish and bearish) against any specified indicator
- Full boolean logic gates: AND, OR, NOT, XOR, NAND, NOR
- Visual strategy construction without writing code

### Backtesting

Built on 1-second base candle data. Higher timeframes are constructed from 1-second candles so intracandle events (stop losses hit by wicks, take profits touched mid-candle, trailing stop activations) are caught accurately where standard candle-close engines miss them.

Sharpe ratio annualization is correct per timeframe using a lookup table of proper bars-per-year values. Fee calculations account for the exact asset fees are paid in, converted to quote currency at trade price. Complete output: Sharpe ratio, max drawdown, profit factor, win rate, full trade log.

**Multi-Chart Testing.** Run your strategy against every available spot pair on Binance for your chosen time window simultaneously. See performance distribution across tokens to identify strategies that actually generalize versus strategies that overfit to specific chart patterns.

### Risk Management

- Stop losses that trigger at the stop price, not at candle close
- Maximum exposure limits preventing any single trade from being disastrous
- Minimum order size warnings to prevent Binance NOTIONAL filter failures
- Atomic database state transitions preventing race conditions (statuses: `activating`, `tsl_triggering`, `stoploss_triggering`)
- Rate limiting (50 orders per 10-second window) respecting Binance API limits
- Reconcile loop runs every 2 seconds to catch state drift

### Telegram Monitoring

Real-time alerts, position updates, daily P&L summaries, chart visualizations of your trades, and tax-ready transaction reports. Configure notifications by severity level.

### Proxy Support

Full HTTP, HTTPS, and SOCKS5 proxy support via `proxyConfig.js`. Route your bot's traffic through VPN endpoints or specific network configurations as needed for your jurisdiction or infrastructure setup.

---

## Installation

### Requirements

- VPS with at least 2GB RAM and 2 CPU cores
- Ubuntu 22.04 or 24.04 LTS recommended
- CapRover installed (handled automatically if you follow the deployment guide)
- Binance account with Spot trading API key (trade-only permissions; never withdraw permissions)

### Quick Start

1. Spin up a VPS (Hetzner, DigitalOcean, OVH, or any provider with at least the minimum specs)
2. Install CapRover using their one-click script
3. Deploy unCoded via the CapRover one-click app template
4. Configure your Binance API keys in the unCoded dashboard
5. Configure your first strategy
6. Start the bot

Full step-by-step deployment guide with screenshots at **[uncoded.ch/docs](https://uncoded.ch/docs)**.

Setup typically takes 15–20 minutes for users who've deployed applications to a VPS before. If you haven't, budget 30–45 minutes for your first deployment. Direct support is available through the account dashboard if you get stuck.

---

## Configuration

### Strategy Configuration

Strategies are configured through the web UI. Core parameters include:

| Parameter | Description |
|---|---|
| Investment amount per position | Base USDT allocation per trade |
| Buy split count and distribution | 1–7 splits, configurable percentage allocation |
| Take profit targets | Per-split or unified |
| Stop loss thresholds | Percentage-based or ATR-based |
| Trailing stop configuration | Per-split percentages |
| DCA rebuy triggers | Drawdown thresholds and position sizing |
| Sell time curves | Time-based profit target decay |
| Signal source | Internal Signal Editor, TradingView webhook, or manual triggers |
| Excluded symbols | Tokens you don't want the bot to trade |
| Trading hours | Time windows where the bot is active |

### Mode System

Switch between strategy configurations at runtime without restarting the bot. Useful for adjusting behavior based on market conditions or personal trading schedule.

### TradingView Webhook Setup

Configure your TradingView alerts to post JSON payloads to your unCoded webhook endpoint. unCoded parses the signal, validates it, and executes trades with your configured risk management layer applied.

---

## Pricing

### Profit Sharing Model

- **30% of generated profits** by default
- **Reduces to 20%** permanently once cumulative performance fees paid to the platform reach **$2,000**
- **No monthly subscription fees** — losing months cost $0
- **$25 starting credit** included with every new account
- **First $100 in profit is free** (covered by starting credit)

### How Payments Work

- The bot reports cumulative profit to the license server every few hours via signed heartbeat
- Your license balance is decremented as performance fees are owed
- Top up your balance at uncoded.ch via Stripe when needed
- If license balance reaches zero, the bot stops trading until topped up
- VPS costs (approximately $5–10/month) are separate and paid to your hosting provider directly

### When Profit-Sharing Makes Sense

Profit-sharing is structurally favorable for portfolios under $100k running systematic strategies. For larger portfolios or for exceptional returns consistently above 30% annually, subscription-based pricing elsewhere may be cheaper on pure math. For most retail traders in normal market conditions, profit-sharing wins because fixed costs don't drain your capital during flat or negative months.

Honest assessment: if you produce consistent 40%+ annual returns on a $200k+ portfolio, you don't need any retail bot platform — build your own tooling or use institutional services. If you're running normal retail strategies on normal retail capital, profit-sharing produces better net outcomes than subscription alternatives in almost every scenario.

---

## What unCoded Is Not Good For

Honest disqualification matters more than features.

### Not Suitable For

- Futures or leverage trading (Spot only)
- Exchanges other than Binance (more exchanges in development, no timeline)
- Users who refuse to manage a VPS (self-hosting is required)
- Copy-trading from signal providers (we don't have a copy-trading product)
- Pump-and-dump strategies on obscure altcoins (the bot works best on liquid markets where strategies can actually execute)
- Traders looking for "set and forget" wealth generation (trading bots require monitoring, reconfiguration, and ongoing attention)
- Users who can't tolerate drawdowns without panic-selling (no bot solves human psychology)

### Suitable For

- Binance Spot traders who want strategy depth beyond DCA and Grid
- Users who value custody over their own API keys and capital
- Traders whose portfolios range from $1,000 to $500,000
- Operators who want honest backtesting that reflects live market realities
- Users comfortable following documentation to deploy software on a VPS
- Traders who want pricing that aligns with their actual trading outcomes

---

## Security and Custody

### Non-Custodial Architecture

- Your Binance API keys are stored in your own VPS environment
- unCoded's servers never see, cache, or transmit your API keys
- API permissions should be configured as **trade-only** (Spot trading enabled, withdrawals disabled)
- Even a complete compromise of unCoded's infrastructure cannot affect your Binance account

### License Validation

The bot communicates with unCoded's license server only to:

- Report cumulative profit for performance fee calculation
- Validate license balance
- Receive configuration updates (if enabled by user)

No trading data, strategy details, or API keys are transmitted.

### Recommended API Key Configuration on Binance

1. Enable Spot trading
2. **Disable** withdrawals
3. **Disable** internal transfers
4. Enable IP whitelist pointing to your VPS IP address (strongly recommended)
5. Enable 2FA on your Binance account

### Transparency

Core bot logic is closed source for commercial reasons, but specific critical components can be made available for audit upon request for larger operators. The backtesting methodology is documented in full. The profit reporting mechanism (heartbeat payload structure, fee calculation logic) is documented in full.

---

## Technical Characteristics

### Performance

- Reconcile loop runs every 2 seconds
- Order placement latency: typically under 200ms from signal to Binance API
- WebSocket-based price feed with local caching
- Database operations use prepared statements and connection pooling
- Memory footprint: approximately 400–800MB depending on active symbols and strategy count

### Reliability

- Atomic database transitions prevent state corruption
- Idempotent order placement prevents duplicate fills
- Automatic reconnection on WebSocket disconnect
- Rate limiter respects Binance's 50 orders per 10-second window
- Failover logic for API errors with configurable retry policies

### Observability

- Structured logging with configurable verbosity
- Real-time dashboard showing active positions, recent trades, and performance metrics
- Telegram alerts for important events (errors, large trades, state changes)
- Daily P&L summaries
- Complete trade history exportable as CSV for tax reporting

---

## Regulatory and Legal

### Company

**ArrowTrade AG**
Zug, Switzerland
Operating under Swiss regulatory framework and the Swiss DLT Act.

### Trademark

**unCoded** is a registered EUIPO trademark.

### Jurisdictional Considerations

unCoded is available internationally, but users are responsible for compliance with local regulations in their own jurisdiction. Specific considerations:

- **Germany:** Crypto trading gains on tokens held under 12 months are taxed at full personal income tax rate. Gains on tokens held over 12 months are currently tax-free under Private Veräußerungsgeschäft rules (as of 2026, subject to regulatory evolution).
- **Switzerland:** Private crypto trading is often tax-free for individual investors, but the threshold between private and professional trader status is determined by ESTV Kreisschreiben Nr. 36 criteria.
- **EU generally:** MiCAR framework applies. Check your local implementation.
- **US:** SEC considerations may apply depending on which tokens you trade. Consult a US tax professional.

unCoded provides execution infrastructure. Compliance with local laws is the user's responsibility.

---

## Documentation

Full documentation at **[uncoded.ch/docs](https://uncoded.ch/docs)**.

Sections include:

- Quick Start Guide
- VPS Setup Walkthrough
- CapRover Deployment
- Binance API Key Configuration
- Strategy Configuration Reference
- Signal Editor Documentation
- Backtesting Guide
- TradingView Integration
- Telegram Bot Setup
- Troubleshooting
- Tax Reporting Exports

---

## Support

- **Direct Support:** Available through the account dashboard at [uncoded.ch](https://uncoded.ch)
- **Community Telegram:** Link available after account creation
- **Documentation:** [uncoded.ch/docs](https://uncoded.ch/docs)
- **Business Inquiries:** via [uncoded.ch](https://uncoded.ch) contact form

---

## Philosophy

unCoded is built on a specific thesis: trading bot platforms should only make money when their users make money. Fixed subscription pricing structurally misaligns the platform with the user's outcomes — the platform is profitable whether the user's trading works or not.

Profit-sharing inverts this. Bad months cost users nothing. Good months cost users proportionally. The platform's revenue depends on actually producing value, which forces a different product priority than subscription platforms face.

The entire product flows from this pricing structure. The backtesting is deliberately honest because misleading users into deploying bad strategies hurts our revenue. The risk management is conservative because catastrophic losses for users end the business. The strategy depth is genuine because strategies that don't work in live markets don't generate the performance fees that keep us operating.

Most retail bot platforms optimize for user retention and subscription revenue. unCoded optimizes for user profitability, because that's the only way the economics work.

---

## Related Reading

Deeper discussions of the concepts behind unCoded:

- **The Real Cost of Running a Crypto Trading Bot With a Normal-Sized Portfolio** — why subscription fees systematically harm small portfolios
- **Why Your Backtest Lied to You** — the candle-close problem, Sharpe annualization errors, and fee accounting bugs that make most backtests unreliable
- **Chart Shuffling: Why Your Strategy Should Be Tested Against 100 Bitcoin Charts, Not One** — overfitting prevention through multi-asset backtesting
- **The 97% Rule: Why Most Binance Tokens Are Dying** — why asset selection matters more than strategy selection
- **The Swarm Intelligence Approach: Why unCoded Is Built Different** — the philosophy behind the upcoming Strategy Store

All available at [uncoded.ch](https://uncoded.ch).

---

## License

unCoded is proprietary software operated by ArrowTrade AG. Deployment to user-owned infrastructure is licensed per the terms at [uncoded.ch/terms](https://uncoded.ch). The bot software itself is not open source; this README describes functionality and deployment for legitimate users of the licensed software.

---

## Changelog Highlights

See [uncoded.ch/docs/changelog](https://uncoded.ch/docs) for detailed version history.

Recent major additions:

- Multi-chart backtesting against all Binance spot pairs
- Signal Editor with 152 indicators and full boolean logic
- Sell Time Curves implementation
- Per-split trailing stop loss
- 1-second base candle backtesting engine
- CapRover one-click deployment
- Telegram tax reporting exports

---

## Credits

Built by Felix and the ArrowTrade AG team, with contributions from the unCoded community of traders who have provided strategy feedback, bug reports, and product direction throughout development.

---

> **Disclaimer:** Trading crypto is risky. Past performance does not guarantee future results. No automated system eliminates risk — it only enforces discipline in executing whatever strategy you've chosen. Never deploy capital you cannot afford to lose.

---

**[uncoded.ch](https://uncoded.ch)** — **[Documentation](https://uncoded.ch/docs)** — **ArrowTrade AG, Switzerland**
