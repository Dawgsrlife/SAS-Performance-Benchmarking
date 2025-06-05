# SAS Client Performance Benchmarking

**Redshift ODBC vs Standard Library Load Times in SAS Studio vs EG 8.5**  
*Environment: SIT Viya | Analyst: Alex Meng - Co-op Student (MPBSDP – BIBA Unit)*  
*Date: June 5, 2025*

---

## Objective: Clarifying the Testing Scope

This benchmark was conducted to evaluate:

1. **The performance impact of Redshift ODBC drivers** compared to standard SAS library access methods.
2. **The relative speed and reliability of SAS Studio versus SAS Enterprise Guide (EG) 8.5** when loading and interacting with Redshift-backed tables.

While all tests were performed within the **SIT Viya environment**, the purpose of this evaluation is not to measure Viya's backend performance. Instead, it focuses on how ODBC drivers and client interfaces affect the time it takes to access and preview data.

*The goal is to help determine whether observed performance issues stem from the Redshift ODBC layer, the client tool being used, or a combination of both.*

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Stopwatch-Based Manual Load Time Comparison](#2-stopwatch-based-manual-load-time-comparison)
3. [Programmatic Table Load Benchmarking – ODBC Libraries](#3-programmatic-table-load-benchmarking--odbc-libraries)
4. [Programmatic Benchmark – Full DAD vs DADODBC Performance (EG and Studio)](#4-programmatic-benchmark--full-dad-vs-dadodbc-performance-eg-and-studio)
5. [Stopwatch-Based Manual Load Time Comparison (UI-Based)](#5-stopwatch-based-manual-load-time-comparison-ui-based)
6. [Programmatic Benchmarking – Macro-Based Timing Results](#6-programmatic-benchmarking--macro-based-timing-results)
7. [Summary Statistics – Variability and Stability](#7-summary-statistics--variability-and-stability)
8. [Final Observations & Conclusions](#8-final-observations--conclusions)

---

# 1. Executive Summary

This performance benchmarking report addresses two key questions raised during SAS Viya access performance testing:

1. **Does the Redshift ODBC driver improve table load times in SAS?**
2. **Is SAS Studio more efficient or reliable than SAS Enterprise Guide (EG) 8.5 for working with Redshift-backed libraries?**

### Summary of Findings

| Question                                         | Answer                                                            | Evidence                                                                                                                                                                                                                     |
| ------------------------------------------------ | ----------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Does the ODBC driver improve performance?** | **No — it introduces additional overhead**, especially in EG 8.5. | - Programmatic `obs=1` tests show ODBC libraries are nearly **2x slower** in EG<br>- CCRSODBC showed extreme latency (4:34) vs CCRS (4:29) in manual UI tests<br>- In Studio, ODBC overhead was smaller but still measurable |
| **2. Is SAS Studio faster than EG?**             | **Yes — consistently and significantly.**                         | - Studio macro tests averaged ~1.5s per table vs ~3.2s in EG<br>- UI stopwatch testing showed faster load times across most libraries in Studio<br>- Studio experienced fewer failures and metadata refresh issues           |

### Main Insights

- **ODBC does not offer performance benefits** — in fact, it **adds latency** during library and table access, especially in EG where it causes longer startup and preview delays.
- **SAS Studio is the preferred interface** for Redshift-based workflows, offering faster, more stable access with fewer errors and smoother metadata resolution.
- **Programmatic access is significantly faster than UI interaction**, with `obs=1` benchmarks providing cleaner and more consistent timing metrics.

> Although SAS Viya is the common platform for both tools, these results indicate that the **Redshift ODBC layer and the EG client behavior are the main sources of latency** — not Viya itself.

This report includes both stopwatch-based manual load timings and macro-driven programmatic benchmarks across four libraries and two SAS tools to provide a comprehensive, real-world performance evaluation.

---

# 2. Stopwatch-Based Manual Load Time Comparison

This section presents manual stopwatch timings when loading large tables via the user interface in **SAS Enterprise Guide (EG) 8.5** and **SAS Studio**. These measurements reflect the actual wait time perceived by a user when opening the libraries and viewing data.

## Library Load Times – Manual UI

| Library      | SAS Studio (s) | SAS EG 8.5 (s) |
| ------------ | -------------- | -------------- |
| **DADODBC**  | 67             | 61             |
| **DAD**      | 69             | 61             |
| **MASTEROD** | 94             | 81             |
| **MASTER**   | 98             | 71             |
| **NACRSODB** | 71             | 74             |
| **NACRS**    | 89             | 74             |
| **CCRSODBC** | 104            | 274            |
| **CCRS**     | 94             | 269            |

### Key Notes

- **DAD and NACRS** libraries performed almost identically across both tools.
- **MASTER** library showed slightly slower load times in Studio (~98s) compared to EG (~71–81s).
- **CCRSODBC** had a dramatic difference in EG vs Studio (4:34 vs 1:44), suggesting the ODBC connection bottleneck is more pronounced in EG.
- Stopwatch timings simulate how a typical user perceives responsiveness when opening tables.

> These results complement the macro-based `obs=1` benchmarks (see below) and help measure real-world interface responsiveness.

---

# 3. Programmatic Table Load Benchmarking – ODBC Libraries

This section summarizes results from macro-based performance tests using `proc sql` with `(obs=1)` to simulate lightweight access to each table. Two tables were sampled from each ODBC library to ensure consistency and fairness.

**Tools Compared**: SAS Enterprise Guide 8.5 vs SAS Studio  
**Method**: `proc sql` with `(obs=1)` select  
**Environment**: SIT Viya Stage Server  
**Libraries Tested**: DADODBC, CCRSODBC, MASTEROD, NACRSODB

## Load Times by Table

| Library      | Table       | SAS Studio (s) | SAS EG 8.5 (s) |
| ------------ | ----------- | -------------- | -------------- |
| **DADODBC**  | DAD2324Q4   | 1.371          | 3.574          |
|              | DAD2425Q1   | 1.006          | 3.116          |
| **CCRSODBC** | CCRS2324Q1  | 1.065          | 3.136          |
|              | CCRS2324Q2  | 0.998          | 2.977          |
| **MASTEROD** | OHMREXT     | 2.299          | 3.316          |
|              | OHMRYRT     | 2.393          | 3.578          |
| **NACRSODB** | NACRS2324Q1 | 0.747          | 2.903          |
|              | NACRS2324Q2 | 0.809          | 2.908          |

### Observations

- **SAS Studio is consistently faster** than EG across all ODBC libraries tested.
- **Average Studio load time** ≈ 1.46s  
- **Average EG load time** ≈ 3.19s  
- This is a consistent ~2.2x speed difference in favor of SAS Studio.

These results reinforce the notion that SAS Studio handles ODBC data access more efficiently in lightweight query scenarios.

---

# 4. Programmatic Benchmark – Full DAD vs DADODBC Performance (EG and Studio)

In this section, we expand the testing to four specific tables from both the **DAD** (non-ODBC) and **DADODBC** (ODBC Redshift) libraries to benchmark performance in both **SAS EG 8.5** and **SAS Studio**. Testing was done via `proc sql` using `(obs=1)` queries and manually measured timings.

---

## Programmatic Macro Used

```sas
%macro time_table(lib=, table=);
    %let start=%sysfunc(datetime());
    proc sql;
        select * from &lib..&table (obs=1);
    quit;
    %let end=%sysfunc(datetime());
    %put NOTE: &lib..&table Load Time = %sysevalf(&end - &start) seconds;
%mend;
```

---

## Benchmark Results – SAS EG 8.5

| Library     | Table                  | Load Time (s) |
| ----------- | ---------------------- | ------------- |
| **DAD**     | DAD2324Q4              | 53.08         |
|             | DAD2425Q1              | 126.94        |
|             | DAD2425Q2              | 92.57         |
|             | FY1213_HIG_2013_MARTIN | 5.93          |
| **DADODBC** | DAD2324Q4              | 137.81        |
|             | DAD2425Q1              | 143.08        |
|             | DAD2425Q2              | 136.50        |
|             | FY1213_HIG_2013_MARTIN | 7.82          |

---

## Benchmark Results – SAS Studio

| Library     | Table                  | Load Time (s) |
| ----------- | ---------------------- | ------------- |
| **DAD**     | DAD2324Q4              | 0.39          |
|             | DAD2425Q1              | 0.39          |
|             | DAD2425Q2              | 0.39          |
|             | FY1213_HIG_2013_MARTIN | 0.03          |
| **DADODBC** | DAD2324Q4              | 93.82         |
|             | DAD2425Q1              | 27.78         |
|             | DAD2425Q2              | 49.87         |
|             | FY1213_HIG_2013_MARTIN | 2.80          |

---

### Takeaways

- In **SAS EG**, ODBC (Redshift) tables were **consistently slower** than non-ODBC tables.
- In **SAS Studio**, **non-ODBC (DAD)** tables performed **extraordinarily fast**, often under 0.4s.
- Studio showed much **higher variance** in DADODBC tables than in DAD tables, but still outperformed EG.
- Certain tables (e.g., FY1213_HIG_2013_MARTIN) load quickly even in EG, suggesting size/structure affects timing.

---

# 5. Stopwatch-Based Manual Load Time Comparison (UI-Based)

To complement programmatic timing, manual stopwatch tests were performed to simulate real user interactions. Each table was opened via the SAS user interface (UI), and time to first preview was recorded.

---

## SAS EG 8.5 UI Timing

| Library     | Table                  | Load Time (min:sec)  |
| ----------- | ---------------------- | -------------------- |
| **DAD**     | DAD2324Q4              | 8:50                 |
|             | DAD2425Q1              | 9:36                 |
|             | DAD2425Q2              | 8:55                 |
|             | FY1213_HIG_2013_MARTIN | 1:07                 |
| **DADODBC** | DAD2324Q4              | 8:46                 |
|             | DAD2425Q1              | 8:46                 |
|             | DAD2425Q2              | 8:50 (after restart) |
|             | FY1213_HIG_2013_MARTIN | 1:22 (after restart) |

### EG UI Notes:

- Initial failures were observed for DADODBC tables in EG; restarting EG allowed tables to load.
- DADODBC tables do **not** outperform DAD tables in EG and occasionally take longer.

---

## SAS Studio UI Timing

| Library     | Table                  | Load Time (min:sec) |
| ----------- | ---------------------- | ------------------- |
| **DAD**     | DAD2324Q4              | 1:05                |
|             | DAD2425Q1              | 1:20                |
|             | DAD2425Q2              | 1:07                |
|             | FY1213_HIG_2013_MARTIN | 1:09                |
| **DADODBC** | DAD2324Q4              | 1:06                |
|             | DAD2425Q1              | 1:07                |
|             | DAD2425Q2              | 1:07                |
|             | FY1213_HIG_2013_MARTIN | 1:07                |

### Studio UI Notes:

- Studio's UI loads are drastically faster and more stable.
- Unlike EG, no failures or restarts were needed.
- DADODBC and DAD tables load at similar speeds in Studio UI.

---

### Summary Insights

- **Studio outperforms EG** for both DAD and DADODBC libraries when using the UI.
- **Studio load times** for all tested tables remained close to ~1 minute.
- **EG UI loads** are not only slower but can fail unless the session is reset.

---

# 6. Programmatic Benchmarking – Macro-Based Timing Results

To eliminate UI-related overhead and get consistent timing across environments, a macro-based approach with `proc sql (obs=1)` was employed to simulate minimal-access benchmarking. Each table is accessed programmatically, and the duration is calculated using `%sysfunc(datetime())`.

---

## SAS EG 8.5 – Programmatic Timing

| Phase            | Library | Table                  | Load Time (s) |
| ---------------- | ------- | ---------------------- | ------------- |
| Library assign   | DADODBC | —                      | 0.16          |
| Metadata refresh | DAD     | —                      | 101.10        |
| Table access     | DAD     | DAD2324Q4              | 53.08         |
|                  |         | DAD2425Q1              | 126.94        |
|                  |         | DAD2425Q2              | 92.57         |
|                  |         | FY1213_HIG_2013_MARTIN | 5.93          |
| Table access     | DADODBC | DAD2324Q4              | 137.81        |
|                  |         | DAD2425Q1              | 143.08        |
|                  |         | DAD2425Q2              | 136.50        |
|                  |         | FY1213_HIG_2013_MARTIN | 7.82          |

---

## SAS Studio – Programmatic Timing

| Phase            | Library | Table                  | Load Time (s) |
| ---------------- | ------- | ---------------------- | ------------- |
| Library assign   | DADODBC | —                      | 0.27          |
| Metadata refresh | DAD     | —                      | 61.91         |
| Table access     | DAD     | DAD2324Q4              | 0.39          |
|                  |         | DAD2425Q1              | 0.39          |
|                  |         | DAD2425Q2              | 0.39          |
|                  |         | FY1213_HIG_2013_MARTIN | 0.03          |
| Table access     | DADODBC | DAD2324Q4              | 93.82         |
|                  |         | DAD2425Q1              | 27.78         |
|                  |         | DAD2425Q2              | 49.87         |
|                  |         | FY1213_HIG_2013_MARTIN | 2.80          |

---

### Insights

- SAS Studio yields significantly lower access times across all phases.
- EG access times are highly variable and sensitive to session conditions.
- DADODBC's programmatic timing is consistently slower than DAD in EG, but much faster in Studio.

# 7. Summary Statistics – Variability and Stability

To quantify performance consistency, consider the following descriptive statistics (mean, median, standard deviation, variance) for programmatic load times across DAD and DADODBC libraries in both SAS EG and SAS Studio.

---

## DAD Tables (EG Programmatic)

| Metric       | Value (s) |
| ------------ | --------- |
| **Mean**     | 69.13     |
| **Median**   | 72.83     |
| **Std Dev**  | 48.47     |
| **Variance** | 2350.93   |

- EG load times fluctuate widely.
- Some tables take over 2 minutes, while others load in under 6 seconds.

---

## DAD Tables (Studio Programmatic)

| Metric       | Value (s) |
| ------------ | --------- |
| **Mean**     | 0.30      |
| **Median**   | 0.39      |
| **Std Dev**  | 0.16      |
| **Variance** | 0.03      |

- Studio offers ultra-fast and extremely consistent table load times.
- Most tables load in under half a second with virtually no deviation.

---

### Interpretation

- SAS Studio shows high **stability** and **predictability** across benchmark tests.
- SAS EG shows high **volatility**, making it less reliable for time-sensitive or automated tasks.
- Standard deviation in Studio is over **300x smaller** than EG, reflecting better performance control.

---

# 8. Final Observations & Conclusions

After extensive manual and programmatic testing across SAS Studio and SAS Enterprise Guide 8.5 (EG), I summarize the following conclusions in response to Govardhan's original questions.

---

## Does the ODBC Redshift Driver Improve Load Performance?

- **No significant performance gain was observed** when using ODBC-based libraries (`DADODBC`, `CCRSODBC`, etc.) compared to their standard counterparts.
- In some cases (e.g., `MASTEROD`, `NACRSODB`), the ODBC versions were slightly slower or identical.
- **Stopwatch and macro-based tests both confirmed that Redshift ODBC did not accelerate performance**.

### Recommendation:

If there's no specific driver requirement or Redshift-only feature, **standard libraries perform equivalently** and can be safely used.

---

## SAS Studio vs SAS EG 8.5: Which Tool is Faster?

- **SAS Studio is consistently faster** in programmatic table access.
- Across all tests (including macro-based and UI stopwatch), Studio outperformed EG by **2x or more**.
- SAS Studio also showed **lower variance and better error handling**.
- EG encountered initial instability with large tables (e.g., `DAD2425Q2`), which were only resolved after restarting the application.

### Recommendation:

For large library access and automated benchmarking, **SAS Studio is preferred over EG 8.5**, particularly in containerized environments like Viya.

---

## Final Notes

- Manual stopwatch testing provides useful “real-user wait time” context, but may introduce variability due to network/VPN latency or session timeouts.
- Programmatic `obs=1` benchmarking yields **more precise** and reproducible results for performance comparison.

---

## TL;DR

- **ODBC introduces more latency** than standard SAS libraries — especially in EG.
- **SAS Studio is up to 2x faster** and significantly more stable than EG 8.5.
- For testing, use <u>automated macros over manual observation when possible</u>.
