# SqlPulse User Guide

**Version 2.8**  
Zakmu Technologies

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Installation](#2-installation)
3. [Interface Overview](#3-interface-overview)
4. [Tab System](#4-tab-system)
5. [Dashboard](#5-dashboard)
6. [Managing Connections](#6-managing-connections)
7. [Query Editor](#7-query-editor)
8. [SP Stress Tester](#8-sp-stress-tester)
9. [SP Performance Analyzer](#9-sp-performance-analyzer)
10. [Active Blockers](#10-active-blockers)
11. [Wait Stats](#11-wait-stats)
12. [Slow Queries](#12-slow-queries)
13. [Permission Manager](#13-permission-manager)
14. [Data Import](#14-data-import)
15. [Test History](#15-test-history)
16. [Comparing Runs](#16-comparing-runs)
17. [Exporting Results](#17-exporting-results)
18. [Settings & License](#18-settings--license)
19. [Frequently Asked Questions](#19-frequently-asked-questions)

---

## 1. Introduction

SqlPulse is a desktop toolkit for **SQL Server performance diagnostics and data management**. It lets you write and execute SQL queries, stress test stored procedures under concurrent load, analyse execution plans and index health, monitor real-time blocking chains, investigate server-wide wait statistics, import data from files and APIs, and compare results across runs — all without HTTP layers, scripting languages, or complex configuration.

**What SqlPulse answers:**

- How does this procedure perform when 25 users call it simultaneously?
- At what concurrency level does latency start to degrade?
- Why is this procedure slow — missing indexes, key lookups, implicit conversions, stale stats?
- Which sessions are blocking right now, and what are they waiting on?
- Is my server under CPU, I/O, lock, or memory pressure?
- How do I get data from a CSV, Excel, API, FTP, or SFTP source into SQL Server quickly?

---

## 2. Installation

SqlPulse is distributed as a standard Windows installer (`.exe`).

1. Download the latest installer from [zakmutechnologies.com](https://www.zakmutechnologies.com)
2. Run the installer and follow the prompts
3. Launch SqlPulse from the Start Menu or desktop shortcut
4. Enter your license key when prompted on first launch

SqlPulse requires a network connection to your SQL Server instance. It does not require any components to be installed on the SQL Server itself.

**Supported SQL Server versions:** SQL Server 2014 and later, Azure SQL Database, Azure SQL Managed Instance.

---

## 3. Interface Overview

SqlPulse uses a modern tabbed layout with a left sidebar for connections and tool navigation, a tab bar for switching between open tools, and a main content area.

```
┌──────────────────────────────────────────────────────────────────────┐
│  Title bar — logo, connection, version, theme toggle, Settings, Help │
├──────────────────────────────────────────────────────────────────────┤
│  🏠 Dashboard │ 🖊️ Query 1 ×│ ⚡ Stress Tester ×│ ＋                │  ← Tab bar
├──────────────┬───────────────────────────────────────────────────────┤
│   Server     │                                                       │
│   Explorer   │            Active tool panel                          │
│              │                                                       │
│  ────────    │  (Dashboard / Query Editor /                          │
│   Tool Nav   │   SP Stress Tester / Active Blockers / …)             │
│  🏠 Dashboard│                                                       │
│  🖊️ Query    │                                                       │
│  ⚡ Stress   │                                                       │
│  …           │                                                       │
└──────────────┴───────────────────────────────────────────────────────┘
```

| Area | Purpose |
|------|---------|
| **Title bar** | App title, active connection, version, theme toggle, Settings, Support |
| **Tab bar** | Open tools appear as tabs; click to switch, × to close, drag outside to pop out |
| **Server Explorer** | Create, manage, and switch between saved database connections |
| **Tool Navigation** | Click any tool to open it as a tab |
| **Main content area** | The active tab's interface |

### Theme

The **☾ / ☀** button in the top-right header switches between light and dark mode. Your preference is saved automatically.

---

## 4. Tab System

SqlPulse works like a browser — each tool opens in its own **tab** so you can keep multiple tools running side-by-side.

### Opening Tabs

- Click a tool in the **left sidebar** navigation
- Click a tool card on the **Dashboard**
- Click **＋** in the tab bar to open a new Query Editor tab

### Switching & Closing

- Click a tab to bring it to the front
- Click **×** on a tab to close it (the last tab cannot be closed)
- Singletons (Dashboard, Blockers, etc.) focus their existing tab if already open; only **Query Editor** supports multiple tabs

### Dot Indicators

The sidebar shows a **teal dot** (●) next to tools that have an open tab, so you can see at a glance what is running.

### Tear-Off — Pop a Tab into Its Own Window

Drag any tab **outside the main window boundary** and release. SqlPulse detects the cursor leaving the window and immediately creates a new independent window for that tool.

The new window:
- Opens at your cursor position
- Shows a minimal title bar (no tab bar, no chrome)
- Keeps the server explorer sidebar so you can pick a connection
- Runs completely independently — you can position it on a second monitor

---

## 5. Dashboard

The Dashboard is the home screen. It provides a quick-launch grid for all available tools. Click any card or its **Open** button to open that tool in a new tab.

Tools marked **Coming Soon** are visible but not yet active in this version.

---

## 6. Managing Connections

All database connections are managed in the **Server Explorer** at the top of the left sidebar.

### Adding a Connection

1. Click **+** at the top of the Server Explorer
2. Fill in the connection details:

| Field | Description |
|-------|-------------|
| **Profile name** | A friendly label, e.g. `Production` or `Dev-Local` |
| **Server** | Hostname, IP, or `server\instance` |
| **Database** | Target database name |
| **Authentication** | SQL Server, Windows (NTLM), Azure AD Password, or Azure Service Principal |
| **Port** | Default `1433` |
| **Encrypt** | Enable TLS encryption |
| **Trust server certificate** | Bypass certificate validation (useful for dev/self-signed) |

3. Click **Load** next to Database to browse available databases (optional)
4. Click **Add Server** to save

### Authentication Modes

| Mode | When to use |
|------|------------|
| **SQL Server** | Username + password login |
| **Windows (NTLM)** | Domain authentication using your Windows credentials |
| **Azure AD Password** | Entra ID username + password |
| **Azure Service Principal** | Tenant ID + Client ID + Client Secret for automated access |

### Selecting a Database

Click any saved server in the explorer to expand it, then click a database to make it the active connection. The active connection is shown in the connection pill at the top of the explorer.

---

## 7. Query Editor

The **Query Editor** is a full SQL editor with IntelliSense, object browser, and result export — similar to SSMS but built into SqlPulse.

### Opening the Query Editor

- Click **🖊️ Query Editor** in the sidebar
- Click **＋** in the tab bar for an additional query window
- You can have **multiple Query Editor tabs** open at once, each with its own SQL and results

### Interface Layout

```
┌──────────────────────────────────────────────────────────┐
│  Object Browser  │  Query 1 │ Query 2 │ +               │
│                  ├──────────────────────────────────────-│
│  Tables          │  ▶ Execute  Clear  💾 Save  F5·Ctrl+↵ │
│  Views           ├──────────────────────────────────────-│
│  Stored Procs    │                                        │
│  Functions       │   Monaco SQL Editor                   │
│  Saved Queries   │                                        │
│                  ├──────────────────────────────────────-│
│                  │  ═══ drag to resize ══════════════════ │
│                  ├──────────────────────────────────────-│
│                  │  Results │ Messages │  ⬇ CSV  ⬇ Excel │
│                  │  Grid…                                 │
└──────────────────┴────────────────────────────────────────┘
```

### Object Browser

The left panel lists all objects in the connected database:

- **Tables** — click once to select, **double-click** to generate `SELECT TOP 100 *`
- **Views** — double-click to generate a SELECT
- **Stored Procedures** — double-click to generate an `EXEC` with all parameters listed
- **Functions** — double-click to generate a function call
- **Saved Queries** — your saved SQL snippets (see [Saving Queries](#saving-queries))

Use the **Filter** box at the top to search across all object types.

Click **⟳** to refresh the object list after schema changes.

### Writing SQL

The editor uses **Monaco** (the same engine as VS Code) with:

- **SQL syntax highlighting**
- **IntelliSense**: table names, column names, views, stored procedures, functions, and schemas all auto-complete as you type
- When you double-click a table/view, column completions for that table are also registered

### Executing Queries

| Action | Result |
|--------|--------|
| **F5** | Execute current query |
| **Ctrl+Enter** | Execute current query |
| **▶ Execute** button | Execute current query |

Results appear below the editor. Multiple result sets (from batches or `EXEC`) each get their own tab. The result grid shows row numbers, column names, and `NULL` in italics.

### Query Tabs (within the editor)

Click **+** in the editor's inner tab bar to open another SQL buffer. Each tab has its own:
- SQL text
- Result grid
- Executing state
- Resizable results panel height

Tab names start at **Query 1, 2, 3…** and update to the saved name if you save the query.

### Resizable Results Panel

Drag the **horizontal divider** between the editor and results up or down to resize. Each query tab remembers its own height.

### Saving Queries

Press **Ctrl+S** or click the **💾 Save** button in the toolbar:

1. An inline name field appears — type a name and press **Enter** (or click Save)
2. The query is saved to SQLite and appears in the **Saved Queries** section of the object browser
3. Subsequent Ctrl+S on the same tab **updates** the saved query in place

To load a saved query: **double-click** it in the Saved Queries list — it loads into the active editor tab.

To delete a saved query: hover it in the list and click **✕**.

### Exporting Results

When the result grid has data, **⬇ CSV** and **⬇ Excel** buttons appear in the results bar:

- **CSV**: instant client-side download, no size limit
- **Excel**: dynamic xlsx generation, opens as a formatted workbook

---

## 8. SP Stress Tester

The SP Stress Tester lets you load test stored procedures under realistic concurrent traffic.

### Setting Up a Test

1. Open the **SP Stress Tester** from the sidebar or Dashboard
2. Select a stored procedure from the dropdown (or type its name)
3. Configure parameters — each parameter can be:
   - **Static** — a fixed value for every execution
   - **Random** — int range, float range, GUID, string length, or pick from a list
   - **CSV** — values cycled from a column in a CSV file you upload

### Test Configuration

| Setting | Description |
|---------|-------------|
| **Concurrency** | Number of simultaneous executions |
| **Total executions** | Total calls to make across all threads |
| **Delay between (ms)** | Pause between each thread's executions |
| **Ramp-up (s)** | Gradually increase from 1 to full concurrency over this duration |
| **Capture results** | Store the first N rows of each result set (for correctness checks) |

### Running the Test

Click **▶ Run Test**. Switch to the **● Live** tab to watch real-time metrics:

- **Throughput** — executions/second chart
- **Avg latency** — average round-trip time chart
- **Success rate** — % of executions that completed without error
- **P50 / P95 / P99** — latency percentiles

Click **■ Stop** to halt early. Results from partial runs are still saved.

### Concurrency Sweep

The **⚡ Sweep** tab runs the same procedure at multiple concurrency levels automatically and charts the results, letting you find the saturation point where latency degrades.

1. Configure tier levels (e.g. 1, 5, 10, 25, 50, 100)
2. Set executions per tier
3. Click **▶ Run Sweep**

The chart shows throughput and average latency across tiers. The sweet spot is usually just before latency starts climbing steeply.

---

## 9. SP Performance Analyzer

The SP Performance Analyzer inspects a stored procedure's execution plan, missing indexes, wait statistics, parameter sniffing, and statement-level hotspots.

### Modes

| Mode | Description |
|------|-------------|
| **Static** | Reads cached plan data from DMVs — no execution needed |
| **With Execution** | Runs the procedure, captures before/after wait stats, then analyses |

### Signals Reported

- **Execution stats** — avg elapsed, CPU, logical reads from plan cache
- **Missing indexes** — from the cached execution plan or `sys.dm_db_missing_index_*`
- **Key lookups** — clustered index lookups visible in the plan
- **Statement hotspots** — the slowest individual statements within the SP
- **Index usage** — seeks, scans, lookups on tables the SP touches
- **Plan warnings** — implicit conversions and missing join predicates
- **Parameter sniffing** — compiled vs runtime parameter values
- **Statistics staleness** — tables with high modification counters or old stats

---

## 10. Active Blockers

The Active Blockers tool shows real-time blocking chains on the connected SQL Server.

### Reading the Display

Sessions are shown in a tree: **head blocker** at the root, blocked sessions as children. Each row shows:

- Session ID, login, host, database
- Current command and wait type
- Wait time, blocking session
- SQL snippet currently executing

### Actions

- **Kill Session** — terminates the selected session (requires `ALTER ANY CONNECTION` permission)
- **⟳ Refresh** — manual refresh
- **Auto-refresh** — toggle automatic refresh every few seconds

---

## 11. Wait Stats

The Wait Stats tool shows server-wide wait statistics normalised per day of uptime, with categorisation and context.

### Categories

Waits are grouped into: CPU, I/O, Lock, Memory, Network, Parallelism, and Other. The category chart gives a quick overview of where the server is spending time.

### Reading the Table

| Column | Meaning |
|--------|---------|
| **Wait type** | SQL Server wait type name |
| **Wait / day (ms)** | Milliseconds of total wait normalised to a 24-hour period |
| **Tasks / day** | Number of waiting tasks per day |
| **Category** | High-level grouping |
| **Signal %** | Proportion of wait that is runnable (CPU) vs actual waiting |

High **Signal %** on a wait type indicates the server may be CPU-bound even for that wait.

### AI Analysis

Click **✨ Analyse with AI** to get a plain-English interpretation of the top waits, likely root causes, and recommended actions.

---

## 12. Slow Queries

The Slow Queries tool surfaces the most expensive queries from SQL Server's plan cache and shows currently-executing requests.

### Plan Cache Tab

Queries are ranked by total elapsed time, CPU, logical reads, or execution count. For each query you can see:

- Execution statistics (avg/total elapsed, CPU, reads, writes)
- The SQL text
- The database context

### Live Tab

The **Live** tab shows queries currently executing on the server (`sys.dm_exec_requests`). Auto-refresh keeps the list current. Useful for spotting runaway queries in real time.

---

## 13. Permission Manager

The Permission Manager audits and manages SQL Server logins, server roles, database roles, and object-level permissions.

### Auditing

Select a **login** from the list to see:

- Server roles assigned
- Database access and roles
- Object-level permissions (tables, views, procedures, functions)

### Applying Changes

Build a permission change using the action builder:

| Action type | Example |
|------------|---------|
| Grant role | Grant `db_datareader` to a login |
| Revoke role | Remove `db_owner` from a login |
| Grant permission | `GRANT EXECUTE ON [dbo].[MyProc] TO [user]` |
| Deny permission | `DENY DELETE ON [dbo].[Orders] TO [user]` |

Click **Preview** to see the exact T-SQL that will run, then **Execute** to apply.

---

## 14. Data Import

The Data Import tool loads data into SQL Server from CSV, Excel, JSON, flat files, and external sources. It handles type inference, column mapping, and bulk insert in a single workflow.

### Source Types

| Source | Formats |
|--------|---------|
| **File Upload** | CSV (`.csv`), Excel (`.xlsx`, `.xls`), JSON (`.json`) — up to 100 MB |
| **Paste Text** | Paste raw CSV or JSON text; auto-detected |
| **Flat File** | Any text file with a custom delimiter (comma, tab, pipe, semicolon, or custom) |
| **External Source** | API Call, FTP, SFTP — _Coming soon_ |

### Import Workflow

1. **Choose a data source** — click one of the four source cards
2. **Upload or paste** your data — for File Upload, drag and drop or click Browse
3. **Preview** — SqlPulse shows the first 5 rows and infers column types
4. **Configure** in the right panel:
   - **Destination** — new table (auto-created) or existing table
   - **Schema** — defaults to `dbo`
   - **Write mode** — Append (add rows), Overwrite (truncate first), or Replace (drop & recreate)
   - **Column mapping** — rename columns, change inferred types, skip columns
   - **On error** — Skip bad rows and continue, or Abort with full transaction rollback
5. Click **Import → N tables** — a global progress bar tracks each file/sheet
6. **Done summary** — shows rows inserted, skipped, and any errors

### Column Mapping

The mapping table shows:

| Column | Purpose |
|--------|---------|
| **Source** | Column name from your file |
| **Target** | Column name in SQL Server (editable) |
| **Type** | Inferred type (varchar, int, decimal, datetime, bit) — editable |
| **Sample values** | First few values from that column to verify inference |
| **Skip** | Exclude this column from the import |

Use **All** / **None** buttons to quickly include or exclude all columns.

### Multi-Sheet Excel and JSON Arrays

- **Excel**: each worksheet becomes a separate import item in the left list
- **JSON**: SqlPulse finds all importable arrays (including nested ones) and creates one item per array path, with dot-notation labels like `orders.items`

### History

Switch to the **History** tab to see the last 50 imports with source filename, target table, row counts, status, and timestamp.

---

## 15. Test History

The **History** tab inside the SP Stress Tester lists all past test runs and sweeps. Click any run to load its results. Use **Reuse Config** to pre-fill the test setup with that run's settings.

---

## 16. Comparing Runs

Select two test runs in the History tab and click **Compare** to see a side-by-side diff of key metrics: throughput, avg/P95/P99 latency, error rate, and concurrency. Differences are highlighted in green (improvement) or red (regression).

---

## 17. Exporting Results

### Stress Test Results

| Format | How |
|--------|-----|
| **PDF** | Click **Export PDF** in the Results panel |
| **Excel** | Click **Export Excel** in the Results panel |

### Query Editor Results

| Format | How |
|--------|-----|
| **CSV** | Click **⬇ CSV** in the results bar |
| **Excel** | Click **⬇ Excel** in the results bar |

### Sweep Results

Click **Export PDF** in the Sweep Results panel for a report with the throughput/latency chart and tier table.

---

## 18. Settings & License

Click **⚙ Settings** in the header to open the Settings panel.

### License Details

- **License Key** — last 4 digits shown
- **Status** — Active, Trial, Offline
- **Activations** — how many machines are using this key vs the maximum
- **Subscription** — billing interval, next renewal date, cancellation status

### Deactivating

Click **Deactivate License** to release this machine's activation. SqlPulse will close and require a license key on next launch. Use this before switching to a new machine.

### Updates

Click the **v2.x.x** version button in the header to check for updates. If one is available, a banner appears. You can download and install without leaving SqlPulse.

---

## 19. Frequently Asked Questions

**Why can't I connect to my SQL Server?**  
Check that the server address is correct, the port (default 1433) is reachable, and your credentials are valid. For Windows auth, ensure SqlPulse is running as a domain user with access. Try disabling encryption if you get certificate errors, or enable **Trust server certificate**.

**The object browser in Query Editor shows 0 tables.**  
This usually means the login doesn't have `VIEW DEFINITION` on the database. Try `GRANT VIEW DEFINITION TO [yourlogin]` or connect with a higher-privilege account. Open DevTools (Ctrl+Shift+I) → Console to see the exact SQL error.

**F5 runs the wrong query.**  
This was a known bug (fixed in v2.8). Update to the latest version.

**Tests run fine but production is slower — why?**  
SqlPulse runs from your local machine, so network latency to the SQL Server is included in all timings. Production traffic goes through different network paths. Run SqlPulse from a machine on the same network as the server for the most representative results.

**Can I run multiple tests simultaneously?**  
Each tab is independent. You can open multiple SP Stress Tester tabs and run separate tests, but be aware that concurrent tests compete for the same SQL Server resources.

**The sweep takes a long time.**  
Each tier runs `executionsPerTier` calls sequentially per thread. Reduce the number of tiers or executions per tier for faster sweeps. You can stop a sweep early with the **Stop** button without losing completed tiers.

**My CSV has special characters / non-ASCII text.**  
SqlPulse assumes UTF-8 encoding for text files. If your CSV uses a different encoding (e.g. Windows-1252), open it in a text editor, save as UTF-8, and re-import.

**How do I import data with a custom delimiter?**  
Select **Flat File** as the source type, choose your delimiter (comma, tab, pipe, semicolon, or enter a custom character), then browse to your file.

**Can I save queries permanently?**  
Yes — press **Ctrl+S** in the Query Editor or click **💾 Save** in the toolbar. Queries are stored in SqlPulse's local SQLite database and appear in the **Saved Queries** section of the object browser. They persist across sessions.

**Can I pop the Query Editor into a second monitor?**  
Yes — drag the Query Editor tab outside the main SqlPulse window. It detects the cursor leaving the window boundary and opens the tool in a new independent window that you can position anywhere.

**What permissions does SqlPulse need?**  
At minimum, the login needs:
- `EXECUTE` on stored procedures to test them
- `VIEW DATABASE STATE` for some DMV-based features (Wait Stats, SP Analyzer)
- `ALTER ANY CONNECTION` to kill sessions in Active Blockers
- `VIEW DEFINITION` for the Query Editor object browser

For Data Import: `INSERT` on target tables, `CREATE TABLE` if creating new tables.

---

*SqlPulse is developed by Zakmu Technologies. For support, visit [zakmutechnologies.com/contact](https://www.zakmutechnologies.com/contact).*
