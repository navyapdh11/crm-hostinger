# Live Trading & Knowledge Documentation

This document outlines the operational flow of the CRM Hostinger trading engine, covering signal generation, risk enforcement, and live execution.

## 1. Signal Lifecycle
1.  **Market Scan**: The `Scout` agent continuously monitors equity/crypto pairs for statistical divergence using the **Multivariate Kalman Filter**.
2.  **Regime Detection**: The `EGARCH(1,1)` model classifies the current market volatility (LOW to CRISIS).
3.  **Path Simulation**: The `MCTS` (Monte Carlo Tree Search) engine simulates 800+ future price paths based on current regime volatility and z-score spreads to calculate Expected Value (EV).
4.  **Consensus**: The `Analyst` agent cross-references the MCTS EV, EGARCH regime, and Kalman z-score.

## 2. Risk Management (The "Circuit Breaker")
*   **VaR/CVaR**: Limits are calculated before every trade.
*   **Black Swan Watch**: The `Sentinel` agent monitors for sudden flash crashes or liquidity voids, instantly signaling a "Halt" to the `Trader`.
*   **Hard Limits**: 
    *   Max daily trades: 10
    *   Max position notional: $25,000
    *   Drawdown threshold: 10% (auto-halt)

## 3. Live Execution Flow
1.  **Trader Approval**: Signals passed to the `Trader` agent for execution timing optimization.
2.  **Broker Routing**: The unified `ConnectionManager` routes the order to the appropriate provider (Alpaca, Binance, Bybit, Futu, IBKR, Tiger).
3.  **Persistence**: Every trade state is locked in `SQLite` via `core/persistence.py`.
4.  **Monitoring**: Real-time state updates are pushed via WebSocket to the dashboard.

## 4. Running Live Trades
### Safety Checklist
- [ ] Ensure `config.py` thresholds are calibrated for current volatility regime.
- [ ] Run `python main.py --validate` to verify connection and configuration health.
- [ ] Start the engine in monitor-only mode before enabling `ExecutionFactory`.

### Execution
```bash
# Start with crash recovery
./autodeploy.sh

# Watch logs for signal ingestion
./watcher.sh
```

**Warning:** Always maintain a kill-switch terminal open with `pkill -f "python main.py"` during initial live runs.
