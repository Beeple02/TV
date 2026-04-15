# TV — TradingView PineScript Tools

A collection of PineScript v5 indicators for quantitative research.

---

## `indicators/data_exporter.pine`

Displays recent bar data in an on-chart table and provides three paths to get that data into Excel.

### What it shows (currently)

| Column | Description |
|--------|-------------|
| Row#   | Bar number (most recent = 1) |
| Date   | Bar open date |
| Time   | Bar open time (exchange timezone) |
| Open   | Open price |
| Close  | Close price |

Rows are color-coded: green = bullish bar, red = bearish, gray = dojo.

---

### How to load it in TradingView

1. Open TradingView → Pine Editor (bottom panel).
2. Paste the contents of `indicators/data_exporter.pine`.
3. Click **Add to chart**.
4. Configure settings (rows to show, position, decimals, etc.) via the indicator's settings gear.

---

### Getting data into Excel — three methods

#### Method 1: Alert CSV history (all plans)

Best for collecting a few hundred bars automatically.

1. In TradingView, open the **Alerts** panel (clock icon, right toolbar).
2. Click **+ Add Alert**.
3. Condition: select **DataExport** → **alert() function calls**.
4. Notifications: enable **TradingView website notifications** (or a Webhook URL if you have a server).
5. Click **Create**.
6. Let the market run — or use **Bar Replay** to replay history and collect alerts.
7. After collection: Alerts panel → click the alert → **Alert History**.
8. Each entry is one CSV row: `AAPL,2024-01-15,09:30,150.25,152.10`.
9. Copy all entries → paste into a text editor → save as `data.csv` → open in Excel.

> **Note:** TradingView caps alert history entries depending on your plan (~200–2000). For large datasets, point the webhook at a small local server (e.g., Python Flask) that appends each POST body line to a `.csv` file.

#### Method 2: TradingView Premium — native CSV export

If you have a Premium plan, this is the fastest path.

1. Add the indicator to the chart (the hidden `plot()` calls register data series invisibly).
2. Right-click anywhere on the chart → **Export chart data...**
3. Check **Open** and **Close** (and any other fields you've added).
4. Download → open directly in Excel.

#### Method 3: On-chart table → manual copy

Good for quick spot-checks, not bulk export.

1. The table appears on the chart (default: bottom-right).
2. Visually inspect values.
3. For small datasets: screenshot + image-to-table tool, or manually transcribe.

---

### Adding more fields (High, Low, Volume, …)

All extension points are marked with `// ★ EXTENSIBILITY` comments in the source. To add **High** as an example:

**Step 1** — `col_headers`: append `"High"`
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

**Step 4** — CSV alert: uncomment
```pine
// + "," + str.tostring(high, fmt)   →   + "," + str.tostring(high, fmt)
```

**Step 5** — hidden plot: uncomment
```pine
// plot(i_plot_en ? high : na, title="High", display=display.none, editable=false)
```

The table rendering loop is already data-driven — no changes needed there.

Repeat the same pattern for `low` and `volume`.
