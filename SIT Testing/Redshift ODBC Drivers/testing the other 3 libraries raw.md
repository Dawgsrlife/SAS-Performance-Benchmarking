## 🕒 Stopwatch-Based Manual Load Time Comparison

These results reflect manual stopwatch timing when loading large tables via the user interface in each SAS tool.

| Library  | SAS Studio (s) | SAS EG 8.5 (s) |
| -------- | -------------- | -------------- |
| DADODBC  | 67             | 61             |
| DAD      | 69             | 61             |
| MASTEROD | 94             | 81             |
| MASTER   | 98             | 71             |
| NACRSODB | 71             | 74             |
| NACRS    | 89             | 74             |
| CCRSODBC | 104            | 274            |
| CCRS     | 94             | 269            |

### 🔍 Key Notes:

- DAD and NACRS libraries performed similarly across both tools.
- MASTER showed slightly slower load times in SAS Studio.
- CCRS loaded significantly faster in SAS Studio (~1:44) than in EG 8.5 (~4:34), likely due to improved data preview and rendering performance.
- Stopwatch-based results show less variability than the `obs=1` macro tests and reflect real-world wait time perception.

_These results complement the macro-based `obs=1` load timings and give a fuller picture of actual user experience._

# 📊 SAS Viya Redshift ODBC Library Performance Benchmark – June 5, 2025

**Environment**: SIT Viya Stage Server  
**Tools Compared**: SAS Studio vs SAS Enterprise Guide (EG) 8.5  
**Method**: `proc sql (obs=1)` select query per table  
**Libraries Tested**: DADODBC, CCRSODBC, MASTEROD, NACRSODB  
**Tables per Library**: 2 samples each  

## 🔬 Load Times by Library

| Library  | Table       | SAS Studio (s) | SAS EG (s) |
| -------- | ----------- | -------------- | ---------- |
| DADODBC  | DAD2324Q4   | 1.371          | 3.574      |
|          | DAD2425Q1   | 1.006          | 3.116      |
| CCRSODBC | CCRS2324Q1  | 1.065          | 3.136      |
|          | CCRS2324Q2  | 0.998          | 2.977      |
| MASTEROD | OHMREXT     | 2.299          | 3.316      |
|          | OHMRYRT     | 2.393          | 3.578      |
| NACRSODB | NACRS2324Q1 | 0.747          | 2.903      |
|          | NACRS2324Q2 | 0.809          | 2.908      |

## 🔍 Observations

- **Studio is consistently faster** across all ODBC libraries tested.
- **Average Studio load time** ≈ 1.46s vs **EG** ≈ 3.19s — a **>2x speed improvement**.
- Results are consistent across all four schemas, suggesting a tool-level optimization difference.

# SAS Viya Library and Table Performance Benchmarking

This report compares the performance of two SAS libraries — **DAD** (non-Redshift ODBC) and **DADODBC** (Redshift ODBC) — across two interfaces: **SAS Enterprise Guide 8.5 (EG)** and **SAS Studio**. Both **manual stopwatch timing** and **programmatic benchmarking** were used.

## Summary of Findings

- **SAS Studio is consistently faster than EG** when accessing the same libraries and tables
- **Programmatic benchmarking runs significantly faster** than using the UI, since it fetches only 1 row (`obs=1`)
- **DAD (non-Redshift ODBC)** showed better table performance than **DADODBC** (Redshift ODBC) in Studio
- **Initial failures in EG** were likely caused by transient VPN or session issues. After restarting EG, the same tables loaded successfully
- **Mean/Median/StdDev** show lower variation in Studio compared to EG, indicating more stable behavior

---

## Manual Timing – SAS Enterprise Guide 8.5

### Library Load Times

- DAD: 120 seconds
- DADODBC: 80 seconds (initial), 60 seconds (after restart), average: **70 seconds**

### Table Load Times (via UI)

**DAD:**

| Table                  | Load Time |
| ---------------------- | --------- |
| DAD2324Q4              | 8m 50s    |
| DAD2425Q1              | 9m 36s    |
| DAD2425Q2              | 8m 55s    |
| FY1213_HIG_2013_MARTIN | 1m 07s    |

**DADODBC:**

| Table                  | Load Time                               |
| ---------------------- | --------------------------------------- |
| DAD2324Q4              | 8m 46s                                  |
| DAD2425Q1              | 8m 46s                                  |
| DAD2425Q2              | failed twice, then 8m 50s after restart |
| FY1213_HIG_2013_MARTIN | failed, then 1m 22s after restart       |

---

## Programmatic Timing – SAS Enterprise Guide 8.5

Macros used:

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

**EG 8.5 Results:**

| Phase              | Library | Table                  | Load Time (s) |
| ------------------ | ------- | ---------------------- | ------------- |
| Library assignment | DADODBC | —                      | 0.16          |
| Metadata refresh   | DAD     | —                      | 101.10        |
| Table access       | DAD     | DAD2324Q4              | 53.08         |
| Table access       | DAD     | DAD2425Q1              | 126.94        |
| Table access       | DAD     | DAD2425Q2              | 92.57         |
| Table access       | DAD     | FY1213_HIG_2013_MARTIN | 5.93          |
| Table access       | DADODBC | DAD2324Q4              | 137.81        |
| Table access       | DADODBC | DAD2425Q1              | 143.08        |
| Table access       | DADODBC | DAD2425Q2              | 136.50        |
| Table access       | DADODBC | FY1213_HIG_2013_MARTIN | 7.82          |

---

## Manual Timing – SAS Studio

### Library Load Times

- DAD: 61 seconds
- DADODBC: 61 seconds

### Table Load Times (via UI)

**DAD:**

| Table                  | Load Time |
| ---------------------- | --------- |
| DAD2324Q4              | 1m 05s    |
| DAD2425Q1              | 1m 20s    |
| DAD2425Q2              | 1m 07s    |
| FY1213_HIG_2013_MARTIN | 1m 09s    |

**DADODBC:**

| Table                  | Load Time |
| ---------------------- | --------- |
| DAD2324Q4              | 1m 06s    |
| DAD2425Q1              | 1m 07s    |
| DAD2425Q2              | 1m 07s    |
| FY1213_HIG_2013_MARTIN | 1m 07s    |

---

## Programmatic Timing – SAS Studio

Same macros used.

**Studio Results:**

| Phase              | Library | Table                  | Load Time (s) |
| ------------------ | ------- | ---------------------- | ------------- |
| Library assignment | DADODBC | —                      | 0.27          |
| Metadata refresh   | DAD     | —                      | 61.91         |
| Table access       | DAD     | DAD2324Q4              | 0.39          |
| Table access       | DAD     | DAD2425Q1              | 0.39          |
| Table access       | DAD     | DAD2425Q2              | 0.39          |
| Table access       | DAD     | FY1213_HIG_2013_MARTIN | 0.03          |
| Table access       | DADODBC | DAD2324Q4              | 93.82         |
| Table access       | DADODBC | DAD2425Q1              | 27.78         |
| Table access       | DADODBC | DAD2425Q2              | 49.87         |
| Table access       | DADODBC | FY1213_HIG_2013_MARTIN | 2.80          |

---

## Statistics Summary

**DAD Tables (EG Program):**

- Mean: ~69.13s
- Median: ~72.83s
- Std Dev: ~48.47s
- Variance: ~2350.93

**DAD Tables (Studio Application):**

- Mean: ~0.30s
- Median: 0.39s
- Std Dev: ~0.16s
- Variance: ~0.03

These values show much higher consistency in Studio compared to EG.

---

## Observations

- Manual UI loads are always slower, likely due to how the interface pulls preview rows and metadata
- Programmatic tests are more controlled and efficient
- EG introduces occasional instability or load failures that do not occur in Studio
- Library assignment time is effectively negligible for Redshift ODBC but significantly longer for DAD (non-Redshift ODBC)

---
