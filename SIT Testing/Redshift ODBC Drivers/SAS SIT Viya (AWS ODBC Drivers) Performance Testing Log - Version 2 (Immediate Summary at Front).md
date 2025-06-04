# SAS Viya Library and Table Performance Benchmarking

This report compares access performance between two SAS libraries — **DAD** (Progress ODBC) and **DADODBC** (AWS Redshift ODBC) — across two environments: **SAS Enterprise Guide 8.5** and **SAS Studio**. Testing was done using two approaches:

- **Manual (UI-based) Stopwatch Timing**
- **Programmatic Benchmarking via SAS Macros**

---

## Summary of Findings

- **SAS Studio is significantly faster** than EG 8.5 when accessing both libraries and tables, especially via programmatic methods.
- **UI-based table loads** are consistently slower because the interface likely fetches more than just one row (possibly full preview or metadata introspection).
- **Redshift (DADODBC)** loads slower than **Progress (DAD)** in general, but Redshift performance improves noticeably in SAS Studio.
- **Programmatic macros** using `obs=1` are ideal for precise benchmarking of access readiness and latency; they simulate a minimal, controlled read.
- **Initial EG access failures** on Redshift were resolved by restarting the application, suggesting transient network or authentication issues.
- **Performance Stats Overview** (for table access timings in <u>seconds</u>):

| Environment | Library | Mean   | Median | Std Dev | Variance |
| ----------- | ------- | ------ | ------ | ------- | -------- |
| EG 8.5      | DAD     | 69.63  | 72.82  | 50.05   | 2505.01  |
| EG 8.5      | DADODBC | 106.30 | 139.66 | 62.07   | 3852.61  |
| Studio      | DAD     | 0.30   | 0.39   | 0.17    | 0.03     |
| Studio      | DADODBC | 43.07  | 38.83  | 39.07   | 1526.76  |

---

## Manual Stopwatch Timing – SAS Enterprise Guide 8.5

### Library Load Times

- **DAD**: 120 seconds
- **DADODBC**: 80 seconds (initial), 60 seconds (after restart), average: **70 seconds**

### Table Load Times (via UI)

**DAD Library:**

| Table                  | Load Time |
| ---------------------- | --------- |
| DAD2324Q4              | 8m 50s    |
| DAD2425Q1              | 9m 36s    |
| DAD2425Q2              | 8m 55s    |
| FY1213_HIG_2013_MARTIN | 1m 07s    |

**DADODBC Library:**

| Table                  | Load Time                             |
| ---------------------- | ------------------------------------- |
| DAD2324Q4              | 8m 46s                                |
| DAD2425Q1              | 8m 46s                                |
| DAD2425Q2              | *Failed twice initially*, then 8m 50s |
| FY1213_HIG_2013_MARTIN | *Failed initially*, then 1m 22s       |

> After restarting EG, all tables loaded properly. The earlier failures appear to have been caused by a transient VPN or authentication error.

---

## Programmatic Timing – SAS Enterprise Guide 8.5

Timing was done using the following SAS macros:

```sas
%macro time_lib(libref=, libcode=);
    %let start=%sysfunc(datetime());
    &libcode
    %let end=%sysfunc(datetime());
    %put NOTE: Library &libref load time = %sysevalf(&end - &start) seconds;
%mend;

%macro time_table(lib=, table=);
    %let start=%sysfunc(datetime());
    proc sql;
        select * from &lib..&table (obs=1);
    quit;
    %let end=%sysfunc(datetime());
    %put NOTE: &lib..&table Load Time = %sysevalf(&end - &start) seconds;
%mend;
```

### Results – EG 8.5

| Phase              | Library | Table                  | Load Time (s) |
| ------------------ | ------- | ---------------------- | ------------- |
| Library assignment | DADODBC | *(Redshift)*           | **0.16**      |
| Metadata refresh   | DAD     | *(Progress)*           | **101.10**    |
| Table access       | DAD     | DAD2324Q4              | 53.08         |
|                    |         | DAD2425Q1              | 126.94        |
|                    |         | DAD2425Q2              | 92.57         |
|                    |         | FY1213_HIG_2013_MARTIN | 5.93          |
| Table access       | DADODBC | DAD2324Q4              | 137.81        |
|                    |         | DAD2425Q1              | 143.08        |
|                    |         | DAD2425Q2              | 136.50        |
|                    |         | FY1213_HIG_2013_MARTIN | 7.82          |

---

## Manual Stopwatch Timing – SAS Studio (Web)

### Library Load Times

- **DAD**: 61 seconds
- **DADODBC**: 61 seconds

### Table Load Times (via UI)

**DAD Library:**

| Table                  | Load Time |
| ---------------------- | --------- |
| DAD2324Q4              | 1m 05s    |
| DAD2425Q1              | 1m 20s    |
| DAD2425Q2              | 1m 07s    |
| FY1213_HIG_2013_MARTIN | 1m 09s    |

**DADODBC Library:**

| Table                  | Load Time |
| ---------------------- | --------- |
| DAD2324Q4              | 1m 06s    |
| DAD2425Q1              | 1m 07s    |
| DAD2425Q2              | 1m 07s    |
| FY1213_HIG_2013_MARTIN | 1m 07s    |

---

## Programmatic Timing – SAS Studio

### Results – SAS Studio

| Phase              | Library | Table                  | Load Time (s) |
| ------------------ | ------- | ---------------------- | ------------- |
| Library assignment | DADODBC | *(Redshift)*           | **0.27**      |
| Metadata refresh   | DAD     | *(Progress)*           | **61.91**     |
| Table access       | DAD     | DAD2324Q4              | 0.39          |
|                    |         | DAD2425Q1              | 0.39          |
|                    |         | DAD2425Q2              | 0.39          |
|                    |         | FY1213_HIG_2013_MARTIN | 0.03          |
| Table access       | DADODBC | DAD2324Q4              | 93.82         |
|                    |         | DAD2425Q1              | 27.78         |
|                    |         | DAD2425Q2              | 49.87         |
|                    |         | FY1213_HIG_2013_MARTIN | 2.80          |

---

## Final Notes

- The **`obs=1` SQL queries** used in macros measure readiness and latency for accessing a table’s schema or small record sample, instead of full UI preview performance.
- **UI loading**, particularly in EG, is impacted by background processes (e.g., metadata introspection, row sampling), making it less predictable and generally slower.
- **Redshift performance** is reasonable under programmatic queries in Studio but lags behind Progress in EG.
- **For future benchmarking or performance checks**, using SAS macros provides a more controlled and repeatable methodology for table readiness testing.


