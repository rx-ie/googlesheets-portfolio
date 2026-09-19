# googlesheets-portfolio

Set up a Google Sheets portfolio tracker and send yourself a weekly roundup of price movements, portfolio allocation changes, and relevant news for your holdings.

This project combines:

- a Google Sheets dashboard with custom Apps Script functions for market data
- a weekly email generator that summarizes portfolio changes and research
- AI-powered email drafting via Gemini and news aggregation via Tavily

## What this does

The workbook is designed to help you track a portfolio in Google Sheets and automatically produce a concise weekly report. It can:

- pull live prices and NAV/book value for SGX and global tickers
- calculate current portfolio value and P/L
- monitor allocation changes from one snapshot to the next
- summarize the biggest movers in the portfolio
- fetch sector or stock-relevant news from Tavily
- draft a polished weekly email using Gemini

## Repository structure

- `dashboard-setup` — Apps Script code for market-data custom functions used inside Google Sheets
- `weeklyemail` — Apps Script code for the weekly email workflow, research retrieval, and fallback report generation
- `README.md` — project overview and setup instructions
- `LICENSE` — MIT license

## Core features

### Dashboard setup

The `dashboard-setup` script adds custom functions that work directly in Google Sheets, including:

- `YAHOO_PRICE(ticker, refresh_switch)`
- `GLOBAL_NAV(ticker, refresh_switch)`
- `SGX_PE(ticker, refresh_switch)`
- `YAHOO_CHART(ticker, refresh_switch)`
- `GLOBAL_HISTORICAL(ticker)`

These functions are designed to support:

- SGX tickers like `SGX:A7RU`
- international tickers like `SPYL.DE`, `SPYL.L`, `NASDAQ:GRAB`
- quick portfolio and valuation lookups without leaving Sheets

### Weekly portfolio email

The `weeklyemail` script:

- reads the `Dashboard` sheet
- validates required portfolio columns
- compares the current portfolio against the previous saved snapshot
- fetches grouped research from Tavily for relevant asset categories
- builds a prompt for Gemini
- sends an HTML portfolio update email to a configured recipient
- stores the last snapshot so future email summaries are meaningful

The automation is designed to run on a scheduled weekly basis, typically after market close on Friday.

## Recommended Google Sheets layout

Create a sheet named `Dashboard` with a header row like this:

- Ticker
- Asset Name
- P / L %
- Current Price
- NAV
- Quantity (Units)
- Avg Cost / W.A.C.
- Total Capital Invested
- Current Value
- P/ L
- Type

The scripts expect these column names to be present.

Example rows can include holdings like:

- `SGX:A7RU`
- `SPYL.DE`
- `MSFT`
- `VOO`

## Setup instructions

### 1. Create the Google Apps Script project

1. Open a Google Sheet.
2. Go to Extensions → Apps Script.
3. Create a new script project.
4. Paste the contents of `dashboard-setup` into a script file.
5. Create a second script file and paste the contents of `weeklyemail`.

### 2. Configure the sheet name and recipient

In `weeklyemail`, update the configuration object:

```javascript
const PORTFOLIO_CONFIG = {
  SHEET_NAME: 'Dashboard',
  RECIPIENT_EMAIL: '[YOUR_EMAIL_HERE]',
  GEMINI_MODEL: 'gemini-3.5-flash-lite',
  TIMEZONE: 'Asia/Singapore',
  SEND_HOUR_FRIDAY_SGT: 18,
  TAVILY_MAX_RESULTS: 5,
  TAVILY_SEARCH_DEPTH: 'basic',
  SNAPSHOT_PROPERTY_KEY: 'PORTFOLIO_LAST_SNAPSHOT'
};
```

Set:

- `SHEET_NAME` to the exact name of your portfolio sheet
- `RECIPIENT_EMAIL` to the email that should receive the report
- `TIMEZONE` to your local timezone if needed

### 3. Add API keys

Use the helper `setPortfolioApiKeys()` to set script properties, or add them manually in Apps Script:

- `GEMINI_API_KEY`
- `TAVILY_API_KEY`

The script expects these values in Script Properties.

### 4. Customize the news groups

The `TAVILY_GROUPS` array contains placeholder examples such as:

```javascript
const TAVILY_GROUPS = [
  {
    groupName: 'Singapore Blue Chips & REITs',
    query: '("Asset A" OR "Asset B" OR "Asset C" OR "Asset D") latest news OR earnings OR business update OR refinancing OR rates OR guidance past week Singapore SGX'
  }
];
```

Replace these placeholders with your actual holdings and asset names. This is how the script decides what company or sector news to fetch.

### 5. Schedule the email

Run this function once from Apps Script:

```javascript
createWeeklyPortfolioTrigger();
```

This creates a weekly Friday trigger at the configured time.

You can also trigger a preview manually with:

```javascript
previewWeeklyPortfolioReport();
```

### 6. Test the portfolio summary

After the script is connected to the dashboard, run:

```javascript
sendWeeklyPortfolioEmail();
```

This sends the portfolio update email immediately and saves a snapshot for later comparisons.

## Example formula usage

After the dashboard script is installed, you can use formulas like:

```excel
=YAHOO_PRICE(A2, $E$1)
=GLOBAL_NAV(A2, $E$1)
=SGX_PE(A2, $E$1)
=YAHOO_CHART(A2, $E$1)
=GLOBAL_HISTORICAL(A2)
```

The exact formulas depend on your workbook layout, but the custom functions are designed to work directly in cells.

## How the weekly report works

Each week, the script:

1. reads the current holdings from the `Dashboard` sheet
2. compares them to the previous saved snapshot
3. identifies the largest price movers and allocation shifts
4. fetches grouped research from Tavily
5. asks Gemini to write a concise HTML email
6. sends the email and stores the new snapshot

If Gemini fails, the script falls back to a plain HTML summary built from the portfolio data and research results.

## Notes and limitations

- This project relies on Yahoo Finance and Tavily APIs, which may change behavior over time.
- Custom functions run in Google Apps Script and may be subject to quotas and rate limits.
- News quality depends on the accuracy of your TAVILY_GROUPS queries.
- The script is intended for personal portfolio tracking and informational use.

## Disclaimer

This project is for personal portfolio tracking and educational purposes only. It is not financial advice.

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.

## Future enhancements

Possible improvements include:

- better portfolio tax and dividend tracking
- support for more exchange-specific tickers
- a more structured dashboard summary
- direct CSV export or archival snapshots
- improved news grouping based on actual holdings

If you want to extend this project, the easiest starting point is to tailor the `TAVILY_GROUPS` queries to your own holdings and refine the dashboard columns to match your portfolio structure.
