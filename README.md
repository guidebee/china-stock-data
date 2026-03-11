# Free China A-Share Daily Stock Data

Daily China stock market data for all A-share, B-share, STAR Market (科创板) and Beijing Stock Exchange (北交所) listed companies.
Updated automatically after each trading day.

---

## Exchanges & Trading Hours

| Exchange | Code | Trading Hours (CST / UTC+8) |
|---|---|---|
| Shanghai Stock Exchange (SSE / 上交所) | prefix `sh` | 09:30–11:30, 13:00–15:00 Mon–Fri |
| Shenzhen Stock Exchange (SZSE / 深交所) | prefix `sz` | 09:30–11:30, 13:00–15:00 Mon–Fri |
| Beijing Stock Exchange (BSE / 北交所) | prefix `bj` | 09:30–11:30, 13:00–15:00 Mon–Fri |

Markets are closed on Chinese public holidays.

---

## Data Directory

```
data/
├── company/
│   └── companies.json        # All listed companies with metadata
└── price/
    └── YYYY/
        └── MM/
            └── stock_price_YYYY_MM_DD.csv   # Daily OHLCV per trading day
```

---

## company/companies.json

A JSON array of all listed companies. Updated weekly.

**Fields:**

| Field | Type | Description |
|---|---|---|
| `symbol` | string | Full symbol with exchange prefix (e.g. `sh600000`) |
| `code` | string | Stock code without prefix (e.g. `600000`) |
| `name` | string | Company name in Chinese |
| `stock_type` | string | Market segment (see below) |
| `trade` | number | Last traded price (CNY) |
| `mktcap` | number | Total market capitalisation (CNY × 1000) |
| `nmc` | number | Circulating market cap (CNY × 1000) |
| `turnoverratio` | number | Turnover ratio (%) |

**Stock types:**

| stock_type | Description | Count |
|---|---|---|
| `sh_a` | Shanghai A-shares (上交所A股) | ~1,703 |
| `sz_a` | Shenzhen A-shares (深交所A股) | ~2,884 |
| `sh_b` | Shanghai B-shares (上交所B股) | ~41 |
| `sz_b` | Shenzhen B-shares (深交所B股) | ~38 |
| `kcb` | STAR Market / Sci-Tech Innovation Board (科创板) | ~604 |
| `hs_bjs` | Beijing Stock Exchange (北交所) | ~298 |

**Example entries:**

```json
[
  {
    "symbol": "sh600000",
    "code": "600000",
    "name": "浦发银行",
    "stock_type": "sh_a",
    "trade": 10.07,
    "mktcap": 33538979.1681,
    "nmc": 33538979.1681,
    "turnoverratio": 0.14922
  },
  {
    "symbol": "sz000001",
    "code": "000001",
    "name": "平安银行",
    "stock_type": "sz_a",
    "trade": 10.85,
    "mktcap": 21055421.24483,
    "nmc": 21055076.708505,
    "turnoverratio": 0.20057
  },
  {
    "symbol": "sz000002",
    "code": "000002",
    "name": "万 科Ａ",
    "stock_type": "sz_a",
    "trade": 4.66,
    "mktcap": 5559710.613486,
    "nmc": 4527842.227114,
    "turnoverratio": 0.51365
  },
  {
    "symbol": "sh688001",
    "code": "688001",
    "name": "华兴源创",
    "stock_type": "kcb",
    "trade": 37.28,
    "mktcap": 1657058.134704,
    "nmc": 1657058.134704,
    "turnoverratio": 2.79301
  },
  {
    "symbol": "bj920000",
    "code": "920000",
    "name": "安徽凤凰",
    "stock_type": "hs_bjs",
    "trade": 18.03,
    "mktcap": 165299.04,
    "nmc": 103841.846775,
    "turnoverratio": 0.7063
  }
]
```

---

## price/YYYY/MM/stock_price_YYYY_MM_DD.csv

One file per trading day. Each row is the aggregated daily OHLCV for one stock.
No header row. Fields:

```
symbol, date, open, close, high, low, volume, amount
```

| Field | Description |
|---|---|
| `symbol` | Full symbol with exchange prefix |
| `date` | Trading date (YYYY-MM-DD) |
| `open` | Opening price (CNY) |
| `close` | Closing price (CNY) |
| `high` | Daily high (CNY) |
| `low` | Daily low (CNY) |
| `volume` | Total shares traded |
| `amount` | Total turnover (CNY) |

**Example** (`stock_price_2026_03_11.csv`):

```
sh600000,2026-03-11,9.97,10.07,10.09,9.85,62853742,625372487.23
sz000001,2026-03-11,10.82,10.85,10.91,10.76,104882300,1137291048.5
sz000002,2026-03-11,4.64,4.66,4.69,4.61,87345200,406852341.8
sh688001,2026-03-11,37.1,37.28,37.85,36.9,5923100,221847320.4
bj920000,2026-03-11,17.9,18.07,18.2,17.87,413986,7453468
```

---

## Usage Examples

### Python / pandas

```python
import pandas as pd
import json

# Load company list
with open('data/company/companies.json') as f:
    companies = pd.DataFrame(json.load(f))

# Filter Shanghai A-shares only
sh_a = companies[companies['stock_type'] == 'sh_a']
print(sh_a[['symbol', 'code', 'name', 'mktcap']].head(10))

# Load a daily price file
cols = ['symbol', 'date', 'open', 'close', 'high', 'low', 'volume', 'amount']
df = pd.read_csv('data/price/2026/03/stock_price_2026_03_11.csv',
                 header=None, names=cols, parse_dates=['date'])
print(df.head())

# Load a date range
import os
from glob import glob

frames = []
for f in sorted(glob('data/price/2026/03/*.csv')):
    frames.append(pd.read_csv(f, header=None, names=cols, parse_dates=['date']))
all_march = pd.concat(frames)

# Get price history for a single stock
sh600000 = all_march[all_march['symbol'] == 'sh600000'].sort_values('date')
print(sh600000)
```

### Node.js

```js
const fs = require('fs');
const path = require('path');

// Load company list
const companies = JSON.parse(
  fs.readFileSync('data/company/companies.json', 'utf8')
);

// Load a daily price file
const lines = fs.readFileSync(
  'data/price/2026/03/stock_price_2026_03_11.csv', 'utf8'
).trim().split('\n');

const prices = lines.map(line => {
  const [symbol, date, open, close, high, low, volume, amount] = line.split(',');
  return { symbol, date, open: +open, close: +close,
           high: +high, low: +low, volume: +volume, amount: +amount };
});

console.log(prices.slice(0, 5));
```

---

## Related Projects

- [ASX Daily Data](https://github.com/guidebee/asx_data_daily) — Same format for Australian Securities Exchange
