# SAS Viya Performance Benchmarking

This repository documents performance testing of library and table access times in the SAS Viya SIT environment using both SAS Enterprise Guide 8.5 (EG) and SAS Studio.

## Purpose

To evaluate and compare the responsiveness of legacy (non-ODBC) and AWS ODBC-based libraries. The findings help assess loading performance across different tools and connection types, with a focus on improving data access strategy within the Ministry.

## Tested Libraries

- DAD / DADODBC  
- CCRS / CCRSODBC  
- MASTER / MASTEROD  
- NACRS / NACRSODB

## Metrics Collected

- Library load time  
- Metadata refresh time  
- Table load time via manual UI interaction (stopwatch timing)  
- Table load time using programmatic benchmark macros (`obs=1` queries)

## Structure

- `docs/`: Markdown summary reports  
- `code/`: SAS macros used for formal benchmarking  
- `logs/`: Raw SAS log outputs from EG and Studio sessions  
- `manual/`: Stopwatch-based notes and table loading attempts

## Tools Used

- SAS Enterprise Guide 8.5 (32-bit)  
- SAS Studio (2024.09) on SIT Viya

## Status

Phase 1 (DAD/DADODBC) testing is complete. Remaining ODBC library testing (CCRS, MASTER, NACRS) is in progress. Full summary report will be updated upon completion.

## Author

Alex Meng  
Application Programmer (Co-op Student)  
Corporate I&IT Solutions & Integration Management Branch (MPBSDP)  
Health Services I&IT Cluster | GovTechON  
alex.meng@ontario.ca
