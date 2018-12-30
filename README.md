# utl-hash-vs-summary-min-and-max-for-four-variables-by-region-for-l89-million-obs
    Hash vs Summary Min and Max for 4 variables by region for 189,000,000 Obs Hash vs Summary Min and Max for 4 variables by region for 189,000,000 Obs

      Four Solutions
      Recent Additional HASH Solution by Paul Dorfman (on end with Pauls astute Comments)

       1.  Paul's HASH      73 seconds  (more flexible than summary?)
       2.  My HASH         237 seconds  (I need learn more about HASH)
       3.  Summary(class)   55 seconds
       4.  Summary(by)      46 seconds


    I did not make sure caches were empty and I did not run batch.
    I did make mutiple runs in different order.

    Terribly I/O bound?

    May be able to cut in half (23 seconds)

    github
    https://tinyurl.com/ybgsz9xf
    https://github.com/rogerjdeangelis/utl-hash-vs-summary-min-and-max-for-four-variables-by-region-for-l89-million-obs

    related post
    https://communities.sas.com/t5/SAS-Programming/calculate-first-maximum-result/m-p/523468

    SAS Forum
    https://tinyurl.com/y9ho8jwk
    https://communities.sas.com/t5/SAS-Programming/Using-HASH-tables-to-find-min-max-after-applying-multiple/td-p/333737

    my comparison
    https://goo.gl/KkCXe5
    https://communities.sas.com/t5/Base-SAS-Programming/Using-HASH-tables-to-find-min-max-after-applying-multiple/m-p/334009#M75363


    INPUT  179,725,000 observations
    ===============================

    Up to 40 obs from wre.wre180 total obs=179,725,000

    Obs    REGION  STORES   SALES INVENTORY RETURNS

      1    Africa      12   29761    191821     769
      2    Africa      12   29761    191821     769
      3    Africa      12   29761    191821     769
      4    Africa      12   29761    191821     769
      5    Africa      12   29761    191821     769


    REGION
    -------------------------

    Africa
    Asia
    Canada
    Central America/Caribbean
    Eastern Europe
    Middle East
    Pacific
    South America
    United States
    Western Europe


    EXAMPLE OUTPUT (40 Obs)
    ------------------------

    Up to 40 obs from WORK.OUT_01 total obs=40

    Obs    REGION   VARIABLE      MAX_VAL  MIN_VAL

      1    Africa   INVENTORY     1063251     3247
      2    Africa   RETURNS         10124       29
      3    Africa   SALES          360209      801
      4    Africa   STORES             25        1

      5    Asia     INVENTORY      469007      455
      6    Asia     RETURNS          2941       10
      7    Asia     SALES          149013      937
      8    Asia     STORES             21        1

    ....

    CREATE DATA
    ===========

    * create 180 million observations;
    data wre.wre180(bufsize=128k bufno=500 keep=region inventory returns sales stores);
      set sashelp.shoes;
      do i=1 to 455000;
        output;
      end;
    run;quit;

    *____
    | __ ) _   _
    |  _ \| | | |
    | |_) | |_| |
    |____/ \__, |
           |___/
    ;

    proc summary data=wre.wre180 max min nway;
    by region;
    var inventory returns sales stores;
    output out=maxmin(drop=_:)
        max=max_inventory max_returns max_sales max_stores
        min=min_inventory min_returns min_sales stores
    ;
    run;quit;

    NOTE: There were 179725000 observations read from the data set WRE.WRE180.
    NOTE: The data set WORK.MAXMIN has 10 observations and 9 variables.
    NOTE: PROCEDURE SUMMARY used (Total process time):
          real time           46.31 seconds
          user cpu time       30.45 seconds
          system cpu time     15.78 seconds
          memory              5245.25k
          OS Memory           69824.00k
          Timestamp           02/17/2017 08:04:25 PM
          Step Count                        54  Switch Count  1

    * ____ _
     / ___| | __ _ ___ ___
    | |   | |/ _` / __/ __|
    | |___| | (_| \__ \__ \
     \____|_|\__,_|___/___/

    ;

    proc summary data=wre.wre180 max min nway;
    class region;
    var inventory returns sales stores;
    output out=maxmin(drop=_:)
        max=max_inventory max_returns max_sales max_stores
        min=min_inventory min_returns min_sales stores
    ;
    run;quit;

    NOTE: There were 179725000 observations read from the data set WRE.WRE180.
    NOTE: The data set WORK.MAXMIN has 11 observations and 9 variables.
    NOTE: PROCEDURE SUMMARY used (Total process time):
          real time           55.32 seconds
          user cpu time       1:44.84
          system cpu time     21.73 seconds
          memory              10471.93k
          OS Memory           72648.00k
          Timestamp           02/17/2017 07:39:39 PM
          Step Count                        37  Switch Count  0

    *_   _           _
    | | | | __ _ ___| |__
    | |_| |/ _` / __| '_ \
    |  _  | (_| \__ \ | | |
    |_| |_|\__,_|___/_| |_|

    ;


    data _null_;
        set wre.wre180 end = eof;
        array Numvals _NUMERIC_;
        attrib variable format=$32.;
        if _n_ = 1 then do;
             call missing(max_val,min_val);
             declare hash h(ordered:'a');
             rc = h.definekey('REGION','variable');
             rc = h.definedata('REGION','variable','max_val','min_val');
             rc = h.definedone();
        end;
        *rc = h.find();
        *array Numvals _NUMERIC_;
        do i=1 to dim(numvals);

        variable=vname(numvals[i]);
            rc = h.find();
             if rc = 0 then do;
                  if numvals[i] > max_val then do;
                        max_val = numvals[i];
                        rc = h.replace();
                  end;
                  if numvals[i] < min_val then do;
                        min_val = numvals[i];
                        rc = h.replace();
                  end;

             end;
             else do;
                  max_val = numvals[i];
                  min_val = numvals[i];
                  rc = h.add();
             end;
             if eof then do;
                   rc = h.output(dataset:'out_01');
                stop;

             end;
        end;

    run;
    proc print data = out_01;
    run;

    NOTE: The data set WORK.OUT_01 has 40 observations and 4 variables.
    NOTE: There were 179725000 observations read from the data set WRE.WRE180.
    NOTE: DATA statement used (Total process time):
          real time           3:57.20
          user cpu time       3:41.56
          system cpu time     15.46 seconds
          memory              4190.84k
          OS Memory           67508.00k
          Timestamp           02/17/2017 07:45:22 PM
          Step Count                        38  Switch Count  8


    *____             _
    |  _ \ __ _ _   _| |
    | |_) / _` | | | | |
    |  __/ (_| | |_| | |
    |_|   \__,_|\__,_|_|

    ;

    * create 180 million observations;
    data wre180(bufsize=128k bufno=500 keep=region inventory returns sales stores);
      set sashelp.shoes;
      do i=1 to 455000;
        output;
      end;
    run;quit;

    NOTE: There were 395 observations read from the data set SASHELP.SHOES.
    NOTE: The data set WORK.WRE180 has 179725000 observations and 5 variables.
    NOTE: DATA statement used (Total process time):
          real time           1:03.28
          cpu time            17.36 seconds


    Methinks that we need to compare apples to apples.
    Your hash output data set structure is different from that of proc SUMMARY,
    and this is the reason the hash seems to underperform so starkly.
    First, let us replicate the SUMMARY output using the hash, you can try this:

    data _null_ ;
      dcl hash h () ;
      h.definekey  ("region") ;
      h.definedata ("region", "max_inventory", "max_returns", "max_sales", "max_stores", "min_inventory", "min_returns", "min_sales", "min_stores") ;
      h.definedone () ;
      do until (z) ;
        set wre180 end = z ;
        array var     inventory     returns     sales     stores ;
        array min min_inventory min_returns min_sales min_stores ;
        array max max_inventory max_returns max_sales max_stores ;
        if h.find() ne 0 then do over var ;
          min = constant ("big") ;
          max = 0 ;
        end ;
        else do over var ;
          if var < min then min = var ;
          if var > max then max = var ;
        end ;
        h.replace() ;
      end ;
      h.output (dataset: "want") ;
    run ;

    NOTE: The data set WORK.WANT has 10 observations and 9 variables.
    NOTE: There were 179725000 observations read from the data set WORK.WRE180.
    NOTE: DATA statement used (Total process time):
          real time           1:13.18
          cpu time            1:13.14


    I haven't tried it with 189 million input obs, but have with 39.5 million. The results, in real time seconds:

    Summary: 13.80
    My hash:  15.96
    Your hash: 43.16

    Now, how does the different data structure affects the run time so badly? The reason is
    that it forces you, for each input record, to call FIND and REPLACE as many times as there
    are analytical variables (in this case, 4 times), as otherwise you cannot obtain the
    VARIABLE values needed for the method calls. FIND and REPLACE are relatively expensive
    since they entail hash<->PDV data
     movement.

    Still, in this case SUMMARY outperforms the hash by ~13 per cent (albeit at the
    expense of twice the memory footprint), even though the log doesn't show it
    doing any multi-threading (and indeed, for this simple minimax task it's hardly necessary).
    The question is then "Why?", especially since a number of other scenarios a
    hash outperforms SUMMARY quite handily. The way I see it, th
    e drag is rooted in the fact that, unless you're using the SUMINC technique
    (not applicable here), hash data cannot be compared with something else directly
    (i.e. without copying it to the PDV first), nor can it be updated in directly in place -
    instead, a value updated in the PDV must to be used to overwrite a value in the hash table.
    My guess is that the values in the AVL tree un
    derlying SUMMARY are compared with the input values and updated directly.


