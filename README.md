# SqlPulse User Guide

**Version 1.9**  
Zakmu Technologies

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Installation](#2-installation)
3. [Interface Overview](#3-interface-overview)
4. [Managing Connections](#4-managing-connections)
5. [Selecting a Stored Procedure](#5-selecting-a-stored-procedure)
6. [Configuring Parameters](#6-configuring-parameters)
7. [Running a Stress Test](#7-running-a-stress-test)
8. [Live Progress](#8-live-progress)
9. [Test Results](#9-test-results)
10. [Concurrency Sweep](#10-concurrency-sweep)
11. [SP Performance Analyzer](#11-sp-performance-analyzer)
12. [Test History](#12-test-history)
13. [Comparing Runs](#13-comparing-runs)
14. [Exporting Results](#14-exporting-results)
15. [Settings & License](#15-settings--license)
16. [Frequently Asked Questions](#16-frequently-asked-questions)

---

## 1. Introduction

SqlPulse is a desktop stress testing tool for **SQL Server stored procedures**. It lets you measure how a stored procedure behaves under concurrent load — how many requests per second it can sustain, where latency degrades, and whether it fails under pressure.

Unlike general-purpose load testing tools, SqlPulse speaks native SQL Server. There is no HTTP layer, no scripting language to learn, and no complex configuration. You connect, pick a procedure, and run.

**What SqlPulse answers:**

- How does this procedure perform when 25 users call it simultaneously?
- At what concurrency level does latency start to degrade?
- Will this procedure hold up under peak traffic on release day?
- Is the improvement after my index change measurable and consistent?
- Why is this procedure slow — missing indexes, key lookups, implicit conversions, stale stats?

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

SqlPulse is organised into four main areas:

```
┌─────────────────────────────────────────────────────────────┐
│  Header bar — app title, run status, update notifications   │
├──────────────┬──────────────────────────────────────────────┤
│              │  Tabs: New Test │ Results │ ⚡ Sweep │ History │ 🔍 Analyze │
│   Server     ├──────────────────────────────────────────────┤
│   Sidebar    │                                              │
│              │            Main content area                 │
│  (saved      │                                              │
│ connections) │                                              │
│              │                                              │
├──────────────┴──────────────────────────────────────────────┤
│  Connection status bar (active server / database)           │
└─────────────────────────────────────────────────────────────┘
```

| Area | Purpose |
|------|---------|
| **Server Sidebar** | Create, manage, and switch between saved database connections |
| **New Test tab** | Configure and start a stress test |
| **Results tab** | View live progress during a run, and full results after |
| **⚡ Sweep tab** | Run an automated concurrency sweep across multiple worker tiers |
| **History tab** | Browse all past runs, compare results, re-run configurations |
| **🔍 Analyze tab** | Diagnose a stored procedure's performance using SQL Server's internal signals |
| **Status bar** | Always shows the currently active server and database |

---

## 4. Managing Connections

All database connections are managed in the **Server Sidebar** on the left.

### Adding a Connection

1. Click the **+** button at the top of the sidebar
2. Fill in the connection details:

| Field | Description |
|-------|-------------|
| **Profile name** | A friendly label, e.g. `Production` or `Dev-Local` |
| **Server** | Hostname, IP address, or `server\instance` |
| **Database** | The target database name |
| **Authentication** | Authentication mode — see [Authentication Modes](#authentication-modes) below |
| **Port** | Default is `1433` |
| **Encrypt** | Enable TLS encryption for the connection |
| **Trust server certificate** | Bypass certificate validation (useful for dev/self-signed certs) |

3. Click **Load** next to the Database field to browse available databases on the server (optional)
4. Click **Add Server** to save

### Authentication Modes

SqlPulse supports five authentication modes. The credential fields shown in the form change depending on the mode selected.

**SQL Server Authentication**  
Standard SQL login with a username and password. Works with any SQL Server instance.

| Field | Description |
|-------|-------------|
| Username | SQL login name (e.g. `sa`) |
| Password | SQL login password |
| Save password | Store the password locally for automatic reconnection |

**Windows Authentication (NTLM)**  
Authenticates using a Windows domain account. Commonly used in on-premises environments.

Enter the Windows credentials for the account you want to connect as — this is typically the same username and password you use to log into your computer or domain.

| Field | Description |
|-------|-------------|
| Username | Windows username — enter as `username` or `DOMAIN\username` |
| Domain | Active Directory domain name (optional if already included in the username) |
| Password | Windows account password |

> **Note:** If you enter `localhost` as the server, SqlPulse will automatically substitute your machine hostname — Windows Authentication does not support the `localhost` alias.

**Azure AD — Password**  
Authenticates against Microsoft Entra ID (formerly Azure Active Directory) using an organisational account and password. Requires the account to have MFA disabled or an app password configured.

| Field | Description |
|-------|-------------|
| Email (UPN) | Entra ID user principal name, e.g. `user@contoso.onmicrosoft.com` |
| Password | Entra ID account password |

**Azure AD — MFA (Interactive)**  
Opens a browser window for interactive sign-in, supporting all MFA methods (Authenticator app, SMS, FIDO2, etc.). The connection waits up to 2 minutes for sign-in to complete.

| Field | Description |
|-------|-------------|
| Email (UPN) | Optional — pre-fills the email field in the browser sign-in prompt |

> **Note:** There are no credentials to save for MFA — a browser prompt will appear each time a connection is opened. If you have the Azure CLI installed and are already signed in (`az login`), that credential will be used silently without a browser prompt.

**Azure AD — Service Principal**  
Authenticates using an Entra ID app registration. Intended for automated or non-interactive scenarios where a specific service identity should be used.

| Field | Description |
|-------|-------------|
| Tenant ID | The Azure AD tenant (directory) ID |
| Client ID | The application (client) ID of the app registration |
| Client Secret | The client secret value from the app registration |
| Save client secret | Store the secret locally for automatic reconnection |

> **Azure SQL requirement:** For any Azure AD authentication mode, enable **Encrypt** in the connection settings. Azure SQL does not accept unencrypted connections.

### Switching Connections

Click any saved connection in the sidebar to make it active. The status bar at the bottom updates to reflect the current connection. Switching connection resets the procedure selection.

### Editing or Deleting a Connection

Hover over a connection in the sidebar to reveal the edit (pencil) and delete (trash) icons.

---

## 5. Selecting a Stored Procedure

With an active connection, the **Procedure** panel at the top of the **New Test** tab lets you select which stored procedure to test.

1. Click **Load Procedures** — SqlPulse queries `sys.procedures` and populates the list
2. Select a procedure from the dropdown
3. Click **Load Parameters** — SqlPulse introspects the procedure signature and populates the parameter table

The procedure name always appears alongside the active tab title so you know what is being tested at a glance.

---

## 6. Configuring Parameters

Each parameter discovered from the procedure signature appears as a row in the parameter table. For each parameter, choose a **mode**:

### Static
A fixed value is used for every execution. Enter the value in the **Value** field.

```
@CustomerId   int   static   42
```

### Random
SqlPulse generates a random value within the constraints you specify. Options:

| Sub-type | Description |
|----------|-------------|
| **Integer** | Random integer between a min and max value |
| **Float** | Random decimal between a min and max value |
| **GUID** | New UUID per execution |
| **String** | Random alphanumeric string of a specified length |
| **List** | Pick randomly from a comma-separated list of values |

```
@CustomerId   int   random   min: 1, max: 100000
```

### CSV
Upload a CSV file and map a column to the parameter. SqlPulse rotates through the values in the file, cycling back to the top when the end is reached. This is the most realistic mode — use it with a sample of your actual production data.

```
@CustomerId   int   csv   customers.csv → customer_id column
```

> **Tip:** Use CSV mode with a representative data sample to avoid testing against artificially hot or cold cache states.

---

## 7. Running a Stress Test

In the **New Test** tab, configure the test parameters in the **Test Configuration** panel:

| Setting | Description | Default |
|---------|-------------|---------|
| **Concurrency** | Number of parallel workers making simultaneous requests | 10 |
| **Total executions** | Total number of SP calls across all workers | 100 |
| **Delay between executions (ms)** | Per-worker pause between successive calls. `0` means fire immediately after each result | 0 |
| **Ramp-up duration (seconds)** | Time over which workers are gradually started. `0` means all workers start simultaneously | 0 |
| **Capture result sets** | Records the rows returned by the SP (up to a configurable limit) | Off |
| **Max rows per execution** | When capture is enabled, the maximum rows stored per execution (1–10) | 5 |

Click **▶ Start Stress Test** to begin. SqlPulse switches automatically to the **Results** tab and begins streaming live metrics.

Click **■ Stop Test** at any time to halt the run. In-flight executions complete before stopping.

### Understanding Concurrency vs. Total Executions

- **Concurrency** controls how many workers run in parallel. A worker immediately starts its next execution as soon as the previous one completes.
- **Total executions** is the global counter. Once the combined execution count across all workers reaches this number, the test ends.

*Example:* Concurrency = 10, Total executions = 1000 → ten workers each run approximately 100 executions.

---

## 8. Live Progress

While a test is running, the **Results** tab shows a live dashboard:

### Progress Bar
Shows how many executions have completed out of the total planned.

### Metric Cards
Eleven live-updating cards:

| Metric | Description |
|--------|-------------|
| **Successes** | Executions that completed without error |
| **Failures** | Executions that returned an error |
| **Success rate** | Successes as a percentage of completed executions |
| **Avg round-trip** | Mean execution duration |
| **Min round-trip** | Fastest single execution |
| **Max round-trip** | Slowest single execution |
| **P50** | Median — 50% of executions were faster than this |
| **P95** | 95% of executions were faster than this |
| **P99** | 99% of executions were faster than this |
| **Throughput** | Executions per second across all workers |
| **Elapsed** | Wall-clock time since the test started |

### Live Charts
Two line charts update in real time (sampled once per second):

- **Throughput over time** — shows whether throughput is stable, ramping up, or degrading
- **Avg latency over time** — shows whether response time is stable or drifting

Charts appear after 3 or more data points have been collected.

---

## 9. Test Results

When a test completes (or is stopped), the full results panel loads below the live metrics.

### Run Information Card
A summary of the test configuration: procedure name, status, start/end times, elapsed duration, concurrency, planned vs. completed execution count, and the server/database used.

### Insights
SqlPulse automatically analyses the results and generates 2–5 observation cards. Each card is colour-coded:

| Colour | Meaning |
|--------|---------|
| **Green** | Positive finding (e.g. perfect success rate, consistent response times) |
| **Amber** | Warning that warrants investigation |
| **Blue** | Informational observation |

Example insights:

> ⚠ **High tail latency** — P99 (2.3 s) is 14× P50 (165 ms). Suggests occasional lock contention or non-covered index scans affecting a minority of requests.

> ✓ **Perfect success rate** — All 1,000 executions succeeded.

> ⚠ **Failures appear time-clustered** — 16 of 18 failures occurred in a 4-second window at t+42s. Suggests a cascading event rather than a per-row issue.

### Metric Grid
The same eleven metric cards from the live view, now showing final values.

### Frozen Charts
The throughput and latency charts from the live run, preserved as a permanent record of the run's shape.

### Duration Histogram
A bar chart bucketing all executions by duration into 10 equal-width ranges. Bars are coloured green (fast) through red (slow). The histogram reveals the distribution of execution times — whether the workload is tight and consistent or has a long tail.

Shown when a run has 10 or more recorded executions.

### Individual Executions Table
A paginated table (50 rows per page) of every recorded execution, showing:

- Execution index
- Start time and offset from run start
- Duration
- Success / failure indicator
- Error message (for failures)

The table is **sortable** by execution index, start time, duration, or status. It is **filterable** to show only successes or only failures. Click any row to expand it and view the exact parameter values used for that execution.

### Error Breakdown
When failures occur, a summary table groups identical error messages and shows their count. This makes it easy to see whether failures are caused by a single recurring issue or multiple distinct problems.

### Parameters Summary
A table listing each parameter, its type, and the mode used (static / random / CSV).

---

## 10. Concurrency Sweep

The **⚡ Sweep** tab runs your stored procedure at a series of concurrency levels automatically, then shows you how performance scales — and where it breaks down.

This answers the question: *"What is the optimal number of concurrent workers for this procedure?"*

### Configuration

**Tier presets:**

| Preset | Tiers |
|--------|-------|
| Light | 1, 5, 10, 25 |
| Standard | 1, 5, 10, 25, 50, 100 |
| Heavy | 1, 10, 25, 50, 100, 200 |
| Custom | Comma-separated list you define |

**Other settings:**

| Setting | Description |
|---------|-------------|
| **Executions per tier** | How many SP calls to run at each concurrency level |
| **Delay between executions** | Per-worker delay, same as a regular test |
| **Ramp-up duration** | Gradual worker startup, same as a regular test |

Click **⚡ Run Concurrency Sweep** to begin. Each tier runs sequentially. A progress bar shows the current tier and how many have completed.

### Sweep Results

Results update after each tier completes.

**Sweet Spot Card**

SqlPulse detects the point where additional concurrency stops paying off:

> ⚡ **Sweet Spot: 25 workers**  
> Peak throughput of 412 req/s at 25 workers. Beyond this, P95 degrades 68% while throughput gains only 8%.

**Dual-Axis Chart**

A single chart plots both:
- **Throughput (req/s)** — left Y-axis, cyan line
- **P95 latency (ms)** — right Y-axis, orange line

Both plotted against concurrency on the X-axis. The sweet spot is typically where the orange line begins to rise sharply while the cyan line flattens.

**Tier Results Table**

One row per completed tier showing: Workers | Throughput | Avg | P50 | P95 | P99 | Success Rate | Executions. The optimal tier is highlighted.

> **Note:** A sweep uses the currently selected procedure and parameter configuration from the **New Test** tab. Ensure your procedure and parameters are configured before switching to the Sweep tab.

---

## 11. SP Performance Analyzer

The **🔍 Analyze** tab diagnoses a stored procedure's performance using SQL Server's own internal signals — execution plan cache, missing index recommendations, index usage statistics, wait events, and query-level hotspots. No stress test required.

### Collection Modes

| Mode | Description |
|------|-------------|
| **Static** | Reads cached DMV data only. Safe — the SP is never executed. Works any time the procedure has been run at least once since the last SQL Server restart. |
| **With Execution** | Runs the SP once with the parameters you provide, then captures a live delta of server wait stats alongside all static signals. Use this when the plan cache is cold or you want to confirm current behaviour. |

### Running an Analysis

1. In the **🔍 Analyze** tab, type or select a stored procedure name
2. Choose **Static** or **With Execution** mode
3. In execution mode, click **Load Params** to populate the parameter table, then fill in values
4. Click **Run Analysis**

Results are organised into five tabs:

#### Summary

A severity score (0–100) and a prioritised list of the most important signals found. Start here — it tells you where to look first.

#### Statements

Per-statement execution statistics pulled from the plan cache. Shows each statement within the procedure ranked by total elapsed time, CPU usage, and logical reads. Use this to identify the exact line causing the most overhead.

Requires the procedure to have been executed at least once since the last SQL Server restart.

#### Warnings

Query plan warnings extracted directly from the cached execution plan:

| Warning | What it means |
|---------|---------------|
| **Implicit conversion** | A parameter or column type mismatch is forcing SQL Server to convert values at runtime, which can prevent index seeks and cause full scans |
| **No join predicate** | A join between two tables has no ON clause — produces a cross join and typically indicates a query bug |

#### Indexes

Three sections covering index health for the procedure's referenced tables:

**Missing Index Suggestions** — indexes SQL Server recommends creating. When the procedure's execution plan is cached, suggestions are read directly from that plan (SP-specific). When the plan is not cached, suggestions are sourced from the DMV filtered to tables this SP directly references. Each suggestion shows:
- Impact percentage — SQL Server's estimate of the query performance improvement if the index is created
- Key columns (equality and inequality predicates)
- Include columns
- A ready-to-run CREATE INDEX script

**Existing Indexes** — all indexes on the procedure's referenced tables, with usage counters (seeks, scans, lookups, updates) since the last SQL Server restart. Indexes with zero activity are flagged **UNUSED**. Indexes with significantly more scans than seeks are flagged **scan-heavy** (often a sign of a missing or poorly-designed index).

> A note below the table shows when SQL Server last restarted. Indexes marked UNUSED may have been active before that point.

**Key Lookups (Plan-Verified)** — detected directly from the execution plan XML. A key lookup means SQL Server found a row in a nonclustered index but then had to fetch additional columns from the clustered index, causing extra IO. Each entry shows the affected table, the driving nonclustered index, and how many times per query execution the lookup is estimated to occur. The fix is to extend the nonclustered index's INCLUDE clause to cover the fetched columns.

#### Execution

Cached execution statistics for the procedure: average duration, CPU time, logical reads, physical reads, writes, execution count, and last execution time.

In **With Execution** mode, a **Wait Stats Delta** section shows the server-wide wait types that accumulated during the execution window. This is an approximation — it includes waits from all concurrent queries on the server, not just this SP — but a dominant wait type is usually a meaningful signal.

### Exporting

Click **⬇ PDF Report** to export a full analyzer report including all signals, index scripts, and warnings.

### AI Analysis

The **Analyze with AI** button will provide a structured diagnosis — root cause classification, prioritised recommendations, suggested index scripts, and an optionally rewritten version of the procedure. This feature is currently marked *Coming Soon* while usage management is finalised.

---

## 12. Test History

The **History** tab lists the last 100 test runs, newest first.

### Columns

| Column | Description |
|--------|-------------|
| Time | When the run started |
| Connection | Server and database the run was executed against |
| Procedure | The stored procedure that was tested |
| Concurrency | Worker count used |
| Executions | Completed / planned |
| Success | Success rate percentage |
| Avg ms | Mean round-trip duration |
| Throughput | Executions per second |
| Status | completed / stopped / error |

### Filtering by Connection

If your history contains runs from multiple connections, a **Connection** dropdown appears above the table. Select a connection to show only runs from that server/database. The dropdown includes a run count per connection.

### Actions

| Button | Description |
|--------|-------------|
| **Details** | Load the full results for this run into the Results tab |
| **Reuse** | Restore the run's procedure, parameters, and connection to the New Test tab |
| **Refresh** | Reload history from disk |
| **Clear History** | Permanently delete all history records (with confirmation) |

---

## 12. Comparing Runs

Any two historical runs can be compared side by side.

1. Check the checkbox next to two runs in the History table
2. Click **Compare selected** — the comparison panel appears above the table
3. Click **Close** to dismiss it

### Comparison Panel

The panel shows Run A and Run B side by side with a **vs A** delta column for Run B. Each metric row shows:

- The raw value for each run
- A percentage delta with a direction indicator

Delta colouring:

- **Green** — Run B improved on Run A (lower is better for latency; higher is better for throughput/success rate)
- **Red** — Run B regressed relative to Run A

Metrics compared: Avg duration, P50, P95, P99, Min, Max, Throughput, Success rate, Execution count.

> **Common use case:** Run a test before and after an index change or query rewrite, then compare the two runs to measure the improvement.

---

## 13. Exporting Results

From the **Results** panel, two export formats are available:

### PDF Report

Click **⬇ PDF Report** to generate a professionally formatted PDF containing:

- SqlPulse branding and report header
- Test configuration summary
- Full performance metrics
- Parameters table
- Individual executions table
- Error breakdown (if applicable)
- Page numbers and footer

The PDF is suitable for sharing with stakeholders, filing in a ticket, or archiving for compliance.

### Excel Export

Click **⬇ Excel Export** to generate a `.xlsx` workbook with four sheets:

| Sheet | Contents |
|-------|----------|
| **Summary** | Test configuration and all performance metrics |
| **Parameters** | Each parameter, its type, and mode configuration |
| **Executions** | Every recorded execution with its parameter values, duration, and outcome |
| **Errors** | Error message breakdown (if applicable) |

The Excel export is useful for further analysis, charting in Excel, or importing into reporting tools.

---

## 14. Settings & License

Click **⚙ Settings** in the top-right header to open the settings modal.

### License Information

The settings panel shows your active license details:

| Field | Description |
|-------|-------------|
| License key | Masked key with last 4 characters visible |
| Status | Active / Trial / Offline / Expired |
| Activated | Date the license was first activated |
| Expires | Expiry date, or "Never" for perpetual licenses |
| Activations | Number of activated machines vs. your plan limit |
| Subscription | Billing interval and current period end date (subscription plans) |
| Registered machines | List of machines this license is activated on |

### Deactivating a License

Click **Deactivate License** to remove the activation from this machine. SqlPulse will close immediately. You can re-activate with the same key on a new machine or after reinstalling.

Manage activations, billing, and account details at [zakmutechnologies.com](https://www.zakmutechnologies.com).

### Updates

SqlPulse checks for updates automatically. When an update is available:

- An **Update available** pill appears in the header
- A banner appears below the header
- Click either to open the update modal

From the update modal you can download and install the update. Installation requires an app restart. Critical updates are marked as required and must be installed to continue using SqlPulse.

To manually check for updates, click the version number (`v1.7`) in the header.

---

## 15. Frequently Asked Questions

**Does SqlPulse store my database credentials?**  
Credentials are saved locally on your machine in the application's data directory. They are never transmitted to Zakmu Technologies or any third party.

**Does SqlPulse require any installation on the SQL Server?**  
No. SqlPulse connects as a normal SQL client using the `mssql` driver. No agents, extensions, or server-side components are required.

**What authentication modes are supported?**  
SqlPulse supports SQL Server Authentication, Windows Authentication (NTLM), Azure AD Password, Azure AD MFA (interactive browser), and Azure AD Service Principal. See [Authentication Modes](#authentication-modes) for details on each.

**Does SqlPulse work with Azure SQL?**  
Yes. SqlPulse works with Azure SQL Database and Azure SQL Managed Instance. Enable **Encrypt** in the connection settings for any Azure connection. For Azure AD authentication modes (Password, MFA, Service Principal) this is required.

**Does the MFA sign-in prompt appear every time?**  
Yes, unless you have the Azure CLI installed and are signed in (`az login`). In that case SqlPulse will use the CLI credential silently and no browser window will open.

**I get "Could not connect (sequence)" — what does that mean?**  
This means SqlPulse reached the server but the TCP/IP handshake failed before authentication. The most common cause is that TCP/IP is disabled on the SQL Server instance — this is the default on fresh SQL Server Express installations.

To fix it:
1. Open **SQL Server Configuration Manager** (search Start, or run `SQLServerManager16.msc` from Win+R — try `15`, `14`, `13` if 16 is not found)
2. Go to **SQL Server Network Configuration → Protocols for MSSQLSERVER**
3. Right-click **TCP/IP** → **Enable**
4. Go to **SQL Server Services** → right-click **SQL Server** → **Restart**

**I get "Login failed for user"**  
Check your username and password. For SQL Server Authentication, ensure the login exists and is enabled in SQL Server. For Windows Authentication, use your full domain credentials (`DOMAIN\username` and your Windows password).

**I get a certificate or TLS error**  
For on-premises SQL Server, uncheck **Encrypt (TLS)** — local servers typically do not have a TLS certificate configured. For Azure SQL, keep Encrypt enabled but also check **Trust server certificate** if you are using a private endpoint or self-signed certificate.

**Connection times out**  
Check that the server name and port (default 1433) are correct, and that any firewall between SqlPulse and the SQL Server allows inbound TCP connections on that port. For Azure SQL, ensure your public IP is added to the Azure SQL firewall rules (SqlPulse shows your outbound IP in the connection panel).

**How many executions can a single test run?**  
Up to 100,000 executions per test run.

**How many results are stored per run?**  
SqlPulse stores the last 1,000 individual execution records per run for display in the executions table. All metrics (including percentiles and throughput) are calculated from the full execution set.

**What does it mean when P99 is much higher than P50?**  
It means most executions are fast, but a small minority are significantly slower. A high P99/P50 ratio typically indicates intermittent lock contention, blocking queries, parameter-sensitive execution plans, or occasional cache misses. The Insights panel will flag this automatically and suggest likely causes.

**Can I run a sweep and a regular test at the same time?**  
No. One test can run at a time. The Sweep tab runs each tier sequentially using the same test infrastructure.

**What happens if I stop a sweep mid-way?**  
Any tiers that completed before the stop are preserved in the results panel. The sweet spot is recalculated based on the tiers available.

**Is my data safe if the app crashes during a test?**  
Any run that reached a terminal state (completed, stopped) before the crash is persisted to disk and will appear in History. A run that was actively in progress at the time of a crash will not be saved.

---

## Support

For assistance, contact the Zakmu Technologies support team:

- **Email:** support@zakmutechnologies.com
- **Web:** [zakmutechnologies.com/contact](https://www.zakmutechnologies.com/contact)

Please include your SqlPulse version number (visible in the header) and a description of the issue when contacting support.

---

*SqlPulse is a product of Zakmu Technologies. All rights reserved.*
