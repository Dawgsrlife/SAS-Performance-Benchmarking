# SAS Viya Performance Testing Log

### Environment: SIT Viya Libraries

---

## Informal Observations

The following observations are based on manual inspection and are not formally timed. These were conducted for initial access verification:

- **MASTER Library**
  
  - Open time: ~5 minutes (eyeballed)
  - **Inaccessible tables**: `D_GEOGRAPHY`, `D_POSTAL_CENSUS`

- **NACRS Library**
  
  - Open time: ~2 minutes (eyeballed)
  - `NACRS2324Q4`: ~7 minutes to load (eyeballed)
  - Other NACRS folders were inaccessible at the time

---

## Formal Performance Testing

Each test uses the following structure to log runtime for accessing 1 row from a target table:

```sas
%let start = %sysfunc(datetime());

/* Access test - Read 1 row */
proc sql;
    select * 
    from <library>.<table_name> (obs=1);
quit;

%let end = %sysfunc(datetime());
%let elapsed = %sysevalf(&end - &start);
%put NOTE: Time to open <table_name> = &elapsed seconds;
```

---

## Test Results

### Library Load Timing

- **NACRS Library**
  
  - **Elapsed**: `61.1083500385284` seconds
  
  - **Script**:
    
    ```sas
    %let start = %sysfunc(datetime());
    
    proc datasets lib=NACRS nolist;
        contents data=_all_ out=work._nacrs_meta(keep=memname) noprint;
    quit;
    
    %let end = %sysfunc(datetime());
    %let elapsed = %sysevalf(&end - &start);
    %put NOTE: Time to open NACRS library = &elapsed seconds;
    ```

---

### Table Load Timings

#### NACRS2425Q2

- **Elapsed**: `132.584540128707` seconds  

- **Script**:
  
  ```sas
  %let start = %sysfunc(datetime());
  
  proc sql;
      select * 
      from NACRS.NACRS2425Q2 (obs=1);
  quit;
  
  %let end = %sysfunc(datetime());
  %let elapsed = %sysevalf(&end - &start);
  %put NOTE: Time to open NACRS2425Q2 = &elapsed seconds;
  ```

#### NACRS2324Q1

- **Elapsed**: `0.29541993141174` seconds  

- **Script**:
  
  ```sas
  %let start = %sysfunc(datetime());
  
  proc sql;
      select * 
      from NACRS.NACRS2324Q1 (obs=1);
  quit;
  
  %let end = %sysfunc(datetime());
  %let elapsed = %sysevalf(&end - &start);
  %put NOTE: Time to open NACRS2324Q1 = &elapsed seconds;
  ```

#### NACRS2324Q2

- **Elapsed**: `0.29487991333007` seconds  

- **Script**:
  
  ```sas
  %let start = %sysfunc(datetime());
  
  proc sql;
      select * 
      from NACRS.NACRS2324Q2 (obs=1);
  quit;
  
  %let end = %sysfunc(datetime());
  %let elapsed = %sysevalf(&end - &start);
  %put NOTE: Time to open NACRS2324Q2 = &elapsed seconds;
  ```

#### NACRS2425Q1

- **Elapsed**: `143.970570087432` seconds  

- **Script**:
  
  ```sas
  %let start = %sysfunc(datetime());
  
  proc sql;
      select * 
      from NACRS.NACRS2425Q1 (obs=1);
  quit;
  
  %let end = %sysfunc(datetime());
  %let elapsed = %sysevalf(&end - &start);
  %put NOTE: Time to open NACRS2425Q1 = &elapsed seconds;
  ```

#### NACRS2425Q2 (Retest)

- **Elapsed**: `0.28826999664306` seconds  

- **Script**:
  
  ```sas
  %let start = %sysfunc(datetime());
  
  proc sql;
      select * 
      from NACRS.NACRS2425Q2 (obs=1);
  quit;
  
  %let end = %sysfunc(datetime());
  %let elapsed = %sysevalf(&end - &start);
  %put NOTE: Time to open NACRS2425Q2 = &elapsed seconds;
  ```

---

## Additional Notes

- **SAS EG 8.5 instability**: Connection to SIT Viya was lost ~5–8 times during testing. Restarting SAS EG was required to reconnect.
- **Inconsistent performance**: Certain tables that initially took >2 minutes to access were later readable in <1 second. This variance may stem from session state, caching, or backend performance.

---

*End of SIT Viya Testing Log*


