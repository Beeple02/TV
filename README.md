# TV — TradingView PineScript Tools

A collection of PineScript v5 indicators for quantitative research.

---

## `indicators/data_exporter.pine`

Displays recent bar data in an on-chart table and exports it as CSV via TradingView alerts — no Premium subscription required.

### What it shows (currently)

| Column | Description |
|--------|-------------|
| Row#   | Bar number (most recent = 1) |
| Date   | Bar open date |
| Time   | Bar open time (exchange timezone) |
| Open   | Open price |
| Close  | Close price |

Rows are color-coded: green = bullish bar, red = bearish, gray = doji.

---

### How to load it in TradingView

1. Open TradingView → Pine Editor (bottom panel).
2. Paste the contents of `indicators/data_exporter.pine`.
3. Click **Add to chart**.
4. Configure settings (rows to show, position, decimals, etc.) via the indicator's settings gear.

---

### Getting data into Excel

#### Method 1: Alert CSV (recommended — all plans)

Each confirmed bar close fires an alert with a CSV-formatted message:
```
AAPL,2024-01-15,09:30,150.25,152.10
```

**Setup:**
1. In TradingView, open the **Alerts** panel (clock icon, right toolbar).
2. Click **+ Add Alert**.
3. Condition: select **DataExport** → **alert() function calls**.
4. Notifications: enable **TradingView website notifications**.
5. Click **Create**.

**Collecting the data:**
- Go to the Alerts panel → click the alert → **Alert History**.
- Each entry is one CSV row.
- Copy all entries → paste into a text file → save as `data.csv` → open in Excel.

> **For longer history:** point the alert at a **Webhook URL** (any plan that supports webhooks). A small local server (e.g. a Python Flask script listening on a public URL via ngrok) receives each POST and appends the body line to a `.csv` file. This bypasses TradingView's alert history cap.

> **For backtesting periods:** use TradingView's **Bar Replay** feature with the alert active to replay historical bars and collect the CSV rows in real time.

**Paste header:** when opening in Excel, the first row should be the column header. Add it manually:
```
Symbol,Date,Time,Open,Close
```

#### Method 2: On-chart table → manual copy

Good for quick spot-checks of a small number of bars. The table appears on the chart (default: bottom-right). Visually inspect or manually transcribe values.

---

### Adding more fields (High, Low, Volume, …)

All extension points are marked with `// ★ EXTENSIBILITY` comments in the source. To add **High** as an example — 5 lines, all in clearly marked sections:

**Step 1** — `col_headers` / `col_is_price`: append new column
```pine
col_headers  = array.from("Row#", "Date", "Time", "Open", "Close", "High")
col_is_price = array.from(false,   false,  false,  true,   true,    true)
```

**Step 2** — `N_COLS`: increment
```pine
int N_COLS = 3
```

**Step 3** — data capture: append `high`
```pine
row = array.from(open, close, high)
```

**Step 4** — CSV alert: append field
```pine
str.tostring(close, fmt) + "," + str.tostring(high, fmt)
```

The table rendering loop is already data-driven — no changes needed there.

Repeat the same pattern for `low` and `volume`.
