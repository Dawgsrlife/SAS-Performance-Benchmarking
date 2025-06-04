# SAS Viya Performance Testing Log

Comparing **SAS Enterprise Guide 8.5** to **SAS Studio (Web)**

---

## Environment

- **Server**: Stage Viya (PD1)
- **Libraries Examined**: `MASTER`, `NACRS`
- **Focus**: Library access latency and table open speed
- **Tools**: Manual stopwatch + `%sysfunc(datetime())` with `obs=1`

---

## 1. Informal Observations (Manual Timing)

These were conducted to gauge general latency and connection reliability.

### SAS Enterprise Guide 8.5

**MASTER Library**

- Library open: ~20 sec
- `D_GEOGRAPHY`: ~20 sec
- `D_POSTAL_CENSUS`: ~10 sec

**NACRS Library**

- Library open: ~25 sec
- `NACRS2223Q4`: ~65 sec
- `NACRS0809_CACS2012_CIHI`: ~10 sec
- `NACRS2324Q4_AUG`: ~73 sec (data view ~50 sec)
- `NACRS2425Q1`: ~60 sec
- `NACRS2425Q2`: ~67 sec

> !!! Occasionally encountered:  
> *"The connection to 'Stage Viya' has been lost..."*

---

### SAS Studio (Web)

**MASTER Library**

- Library open: 10 sec
- `D_GEOGRAPHY`: 13 sec
- `D_POSTAL_CENSUS`: 12 sec

**NACRS Library**

- Library open: 10 sec
- `NACRS2223Q4`: 10 sec
- `NACRS0809_CACS2012_CIHI`: 11 sec
- `NACRS2324Q4_AUG`: 11 sec
- `NACRS2425Q1`: 11 sec
- `NACRS2425Q2`: 11 sec

---

## 2. Formal Timing Tests (Programmatic)

Tests used `%sysfunc(datetime())` to log time to retrieve 1 record per table.

---

### Example SAS Code Snippet

```sas
%let start = %sysfunc(datetime());
proc sql;
    select * from NACRS.NACRS2223Q4 (obs=1);
quit;
%let end = %sysfunc(datetime());
%let elapsed = %sysevalf(&end - &start);
%put NOTE: Time to open NACRS2223Q4 = &elapsed seconds;
```

---

## 3. Programmatic Timing Results

### SAS Enterprise Guide 8.5

| Table                   | Time (s) |
| ----------------------- | -------- |
| MASTER library open     | 80.77    |
| D_GEOGRAPHY             | 5.14     |
| D_POSTAL_CENSUS         | 5.79     |
| NACRS library open      | 19.47    |
| NACRS2223Q4             | 21.17    |
| NACRS0809_CACS2012_CIHI | 11.21    |
| NACRS2324Q4_AUG         | 15.57    |
| NACRS2425Q1             | 6.25     |
| NACRS2425Q2             | 4.24     |

---

### SAS Studio (Web)

| Table                   | Time (s) |
| ----------------------- | -------- |
| MASTER library open     | 41.18    |
| D_GEOGRAPHY             | 0.63     |
| D_POSTAL_CENSUS         | 0.64     |
| NACRS library open      | 14.77    |
| NACRS2223Q4             | 0.28     |
| NACRS0809_CACS2012_CIHI | 0.04     |
| NACRS2324Q4_AUG         | 0.29     |
| NACRS2425Q1             | 0.27     |
| NACRS2425Q2             | 0.27     |

---

## 4. Timing Statistics (SAS Studio)

### MASTER Library (2 tables)

- **Mean**: 0.63 sec  
- **Median**: 0.63 sec  
- **Mode**: 0.63 sec  
- **Std. Dev**: 0.009 sec  
- **Variance**: 0.00008

### NACRS Library (5 tables)

- **Mean**: 0.23 sec  
- **Median**: 0.27 sec  
- **Mode**: 0.04 sec  
- **Std. Dev**: 0.11 sec  
- **Variance**: 0.012

---

## 5. Conclusion

Across both informal and formal performance testing on the Stage Viya (PD1) server, **SAS Studio (Web)** consistently outperformed **SAS Enterprise Guide 8.5**, especially for individual table access.

The formal timing tests using `obs=1` queries show significant reduction in load times across all concerned data sources, especially in the `NACRS` library.

---

## 6. Methodology & Code Style

- **Timer Used**: `%sysfunc(datetime())` before/after access.
- **Access Type**: `proc sql` with `(obs=1)` for lightweight query testing.
- **Environment**: Staging Viya PD1; both tools connected to same libraries.
- **Format**: Markdown written by Alex Meng, May 2025
- **Tools**: SAS Enterprise Guide 8.5 (32-bit), SAS Studio (web interface)

This benchmark-style test aims to provide an objective, script-backed snapshot of real-world performance behavior.

---


