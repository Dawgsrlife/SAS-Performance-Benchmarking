# SAS Viya Library and Table Performance Benchmarking

This document compares the performance of two SAS libraries — **DAD** (Progress ODBC) and **DADODBC** (AWS Redshift ODBC) — across two environments: **SAS Enterprise Guide 8.5 (EG)** and **SAS Studio (web)**. Both **manual stopwatch testing** and **programmatic benchmarking** were used.

---

## 🕒 Manual Timing – SAS Enterprise Guide 8.5

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

| Table                  | Load Time                                        |
| ---------------------- | ------------------------------------------------ |
| DAD2324Q4              | 8m 46s                                           |
| DAD2425Q1              | 8m 46s                                           |
| DAD2425Q2              | *(initially failed twice)*, 8m 50s after restart |
| FY1213_HIG_2013_MARTIN | *(initially failed)*, 1m 22s after restart       |

> ⚠️ Failures appeared to be caused by a transient network or VPN issue. After restarting EG, all tables loaded successfully.

---

## 🧪 Programmatic Timing – SAS Enterprise Guide 8.5

The following SAS macros were used:

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

### 📊 EG 8.5 – Results

| Phase              | Library | Table                  | Load Time (s) |
| ------------------ | ------- | ---------------------- | ------------- |
| Library assignment | DADODBC | *(Redshift ODBC)*      | **0.16**      |
| Metadata refresh   | DAD     | *(Progress ODBC)*      | **101.10**    |
| Table access       | DAD     | DAD2324Q4              | 53.08         |
| Table access       | DAD     | DAD2425Q1              | 126.94        |
| Table access       | DAD     | DAD2425Q2              | 92.57         |
| Table access       | DAD     | FY1213_HIG_2013_MARTIN | 5.93          |
| Table access       | DADODBC | DAD2324Q4              | 137.81        |
| Table access       | DADODBC | DAD2425Q1              | 143.08        |
| Table access       | DADODBC | DAD2425Q2              | 136.50        |
| Table access       | DADODBC | FY1213_HIG_2013_MARTIN | 7.82          |

---

## 🕒 Manual Timing – SAS Studio

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

## 🧪 Programmatic Timing – SAS Studio

Macros reused from earlier.

### 📊 SAS Studio – Results

| Phase              | Library | Table                  | Load Time (s) |
| ------------------ | ------- | ---------------------- | ------------- |
| Library assignment | DADODBC | *(Redshift ODBC)*      | **0.27**      |
| Metadata refresh   | DAD     | *(Progress ODBC)*      | **61.91**     |
| Table access       | DAD     | DAD2324Q4              | 0.39          |
| Table access       | DAD     | DAD2425Q1              | 0.39          |
| Table access       | DAD     | DAD2425Q2              | 0.39          |
| Table access       | DAD     | FY1213_HIG_2013_MARTIN | 0.03          |
| Table access       | DADODBC | DAD2324Q4              | 93.82         |
| Table access       | DADODBC | DAD2425Q1              | 27.78         |
| Table access       | DADODBC | DAD2425Q2              | 49.87         |
| Table access       | DADODBC | FY1213_HIG_2013_MARTIN | 2.80          |

---

## 🔍 Observations

- **UI-based table loading is significantly slower** than programmatic access due to the UI likely fetching full metadata and sample records.
- **SAS Studio consistently outperforms EG** in both metadata refresh and table access speed.
- **Programmatic access timings reflect only a 1-row query** (`obs=1`), which drastically reduces load time vs full table preview in the UI.
- **Initial failures on DADODBC** appear transient — subsequent tests succeeded after restarting SAS EG.

---

## 📛 Naming Note

The benchmarking script used throughout this report is referred to internally as:

**`table-access-timer.sas`**

You can reuse this macro-based script across Viya environments for consistent performance testing.

---


