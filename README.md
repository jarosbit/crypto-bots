# 🤖 Crypto Bots

Open-source experiments for CEX/DEX trading: grid, momentum, funding-farming, and simple market data tools.

## 🚀 Modules
- `bots/grid_bot.py` – minimal grid executor (paper-trading ready)
- `bots/momentum_bot.py` – simple MA crossover signaler
- `tools/funding_monitor.py` – fetch & log funding rates (design)
- `common/exchange.py` – unified exchange wrapper (stub)

## ⚙️ Stack
Python 3.10+, `ccxt` (optional), `pandas` (optional). Start with paper mode.

## 🏁 Quick Start
```bash
python bots/grid_bot.py --symbol BTC/USDT --grid 20 --step 0.4 --cash 1000 --paper
📜 Roadmap

 Paper engine + PnL report

 Plug CCXT for live trading (keys via .env)

 Backtests (csv candles)

 Risk layer: max drawdown, position caps

⚠️ Disclaimer

For educational purposes only. No financial advice. Use paper trading first.

---

# 3) `.gitignore` (создай файл с таким именем)

Python

pycache/
*.py[cod]
.egg-info/
.venv/
.env
.env.
.cache/
.ipynb_checkpoints
.DS_Store


---

# 4) `LICENSE` (MIT)



MIT License

Copyright (c) 2025 Jaros

Permission is hereby granted, free of charge, to any person obtaining a copy
...
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND...


(Можно выбрать “Add license → MIT” прямо в UI, чтобы не копировать вручную.)

---

# 5) Папки и заготовки кода

## `bots/grid_bot.py`
```python
#!/usr/bin/env python3
import argparse, math, time, random

def paper_price_stream(start=50000, drift=0.0, vol=0.003):
    p = start
    while True:
        p *= (1 + drift + random.uniform(-vol, vol))
        yield round(p, 2)

def run(symbol, grid, step, cash, paper=True):
    price_gen = paper_price_stream()
    base_qty = 0.0
    # построим уровни относительно первой цены
    p0 = next(price_gen)
    levels = [p0 * (1 + step/100)**i for i in range(-grid//2, grid//2+1)]
    levels.sort()
    deals = 0
    while deals < 50:  # короткий демо-ран
        p = next(price_gen)
        # простая логика: покупка на нижних, продажа на верхних
        for lv in levels:
            if p <= lv and cash > 0:
                qty = max(0.0001, cash / (grid*2) / p)
                cash -= qty * p
                base_qty += qty
                deals += 1
                print(f"BUY {symbol} @{p} qty={qty:.6f} cash={cash:.2f} base={base_qty:.6f}")
            elif p >= lv and base_qty > 0:
                qty = base_qty / (grid*2)
                base_qty -= qty
                cash += qty * p
                deals += 1
                print(f"SELL {symbol} @{p} qty={qty:.6f} cash={cash:.2f} base={base_qty:.6f}")
        time.sleep(0.1)
    # итог
    equity = cash + base_qty * p
    print(f"\nEND price={p} cash={cash:.2f} base={base_qty:.6f} equity={equity:.2f}")

if __name__ == "__main__":
    ap = argparse.ArgumentParser()
    ap.add_argument("--symbol", default="BTC/USDT")
    ap.add_argument("--grid", type=int, default=20)
    ap.add_argument("--step", type=float, default=0.4, help="step in % between levels")
    ap.add_argument("--cash", type=float, default=1000.0)
    ap.add_argument("--paper", action="store_true")
    args = ap.parse_args()
    run(args.symbol, args.grid, args.step, args.cash, paper=args.paper)

bots/momentum_bot.py
#!/usr/bin/env python3
import csv, statistics, argparse

def sma(xs, n):
    return [None if i+1<n else sum(xs[i+1-n:i+1])/n for i in range(len(xs))]

def load_csv(path):
    with open(path) as f:
        r = csv.DictReader(f)
        return [float(row["close"]) for row in r]

if __name__ == "__main__":
    ap = argparse.ArgumentParser()
    ap.add_argument("--csv", required=True, help="candles CSV with 'close'")
    ap.add_argument("--fast", type=int, default=20)
    ap.add_argument("--slow", type=int, default=50)
    args = ap.parse_args()

    closes = load_csv(args.csv)
    f = sma(closes, args.fast); s = sma(closes, args.slow)
    signals = []
    for i in range(len(closes)):
        if f[i] is None or s[i] is None: continue
        if f[i] > s[i] and f[i-1] <= s[i-1]: signals.append(("BUY", i))
        if f[i] < s[i] and f[i-1] >= s[i-1]: signals.append(("SELL", i))
    print(*signals[:20], sep="\n")

tools/funding_monitor.py (каркас)
# design: poll exchanges every N minutes, log to csv, alert on threshold
# to implement later with ccxt / API clients

common/exchange.py (каркас)
# place for a thin wrapper around ccxt / http clients
