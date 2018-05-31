# utl_drop_stock_in_a_month_with_less_than_four_non_zero_returns
utl_drop_stock_in_a_month_with_less_than_four_non_zero_returns.  Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.

    Drop stock in a month with less than four non-zero returns

    Millions is a very small dataset on a power workstation(todays laptops).
    Unless you are on a heaily utilized server.
    I don't think it matters how you programe it.

       Three Solutions

         1. DOW loop
            Failed WPS: ERROR: GROUPFORMAT specification not supported on BY statement in DATA step
            WPS is at 9.1? expect we will see it in the future?

         2. HASH
            Failed WPS: ERROR: Argument tag 'key' is not allowed on method SUM for object h1 of type HASH
            Probably a workaround but my HASh skills are limited.
            WPS is at 9.1? expect we will see it in the future?

         3. SQL
            Same result in WPS and SAS

    github
    https://communities.sas.com/t5/Base-SAS-Programming/Drop-stock-in-a-month-with-less-than-five-non-zero-returns/m-p/466316

    Patrick HASH
    https://communities.sas.com/t5/user/viewprofilepage/user-id/12447

    KSharp SQL
    https://communities.sas.com/t5/user/viewprofilepage/user-id/18408



    INPUT
    =====


    WORK.HAVE total obs=30

     STOCK      DATE       RET

       1     2018-03-01     0
       1     2018-03-02     1  1 non-zero
       1     2018-03-03     0
       1     2018-03-04     3  2nd non-zero
       1     2018-03-05     0
       1     2018-03-06     2  3rd non-zero  (only 3)
       1     2018-03-07     0  Sum of the non-zero returns = 3 < 4 so drop

       2     2018-03-01     2
       2     2018-03-02     3
       2     2018-03-03     4
       2     2018-03-04     5
       2     2018-03-05     6
       2     2018-03-06     7
       2     2018-03-07     8  Sum of the non-zero returns => 4 so keep
       2     2018-03-08     0

       1     2018-04-01     2
       1     2018-04-02     3
       1     2018-04-03     4
       1     2018-04-04     5
       1     2018-04-05     6
       1     2018-04-06     7
       1     2018-04-07     8  Sum of the non-zero returns => 4 so keep

       2     2018-04-01     2
       2     2018-04-02     3
       2     2018-04-03     4
       2     2018-04-04     5
       2     2018-04-05     6
       2     2018-04-06     7
       2     2018-04-07     8
       2     2018-04-08     0  Sum of the non-zero returns => 4 so keep


     PARTIAL EXAMPLE OUTPUT

       WORK.HAVE total obs=30

        STOCK     DATE       RET
         2     2018-03-01     2
         2     2018-03-02     3
         2     2018-03-03     4
         2     2018-03-04     5
         2     2018-03-05     6
         2     2018-03-06     7
         2     2018-03-07     8
         2     2018-03-08     0
       ....

    PROCESS
    =======

    1. DOW loop

       data want;

         retain cnt 0 stock date;
         format date yymm6.;

         do until(last.date);
           set have;
           by stock date groupformat notsorted;
           if ret ne 0 then cnt=cnt+1;
         end;

         do until(last.date);
           set have; /* data still very close to cpu due to cache? */
           by stock date groupformat notsorted;
           if cnt > 3 then output;
         end;

         cnt=0;
         drop cnt;

       run;quit;

    2. HASH

       %let treshold_cnt=4;
       data want(drop=_:);

         if _n_=1 then
           do;
             length _cnt 3;
             _cnt=1;
             dcl hash h1(suminc: '_cnt', multidata:'n');
             h1.defineKey('stock','date');
             h1.defineDone();
             do i=1 to nobs;
               set have nobs=nobs point=i;
               date=intnx('month',date,0,'b');
               if ret ne 0 then h1.ref();
             end;
           end;

           set have;
           h1.sum(key:stock, key:intnx('month',date,0,'b'), sum:_zero_ret_cnt);
           if _zero_ret_cnt ge &treshold_cnt then output;
       run;

    3. SQL  ( does not maintain input orser)

       proc sql;
         create
            table want as
         select
            *
         from
            have
         group
            by stock, put(date,yymm6.)
         having
            sum(ret ne 0) ge 4
      ;quit;

    OUTPUT
    =====

      First stock removed

      STOCK       DATE       RET

        2      2018-03-01     2
        2      2018-03-02     3
        2      2018-03-03     4
        2      2018-03-04     5
        2      2018-03-05     6
        2      2018-03-06     7
        2      2018-03-07     8
        2      2018-03-08     2

        1      2018-04-01     3
        1      2018-04-02     4
        1      2018-04-03     5
        1      2018-04-04     6
        1      2018-04-05     7
        1      2018-04-06     8
        1      2018-04-07     2

        2      2018-04-01     3
        2      2018-04-02     4
        2      2018-04-03     5
        2      2018-04-04     6
        2      2018-04-05     7
        2      2018-04-06     8
        2      2018-04-07     0
        2      2018-04-08     0

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    SAS see process


    %utl_submit_wps64('
    libname wrk sas7bdat "%sysfunc(pathname(work))";
    proc sql;
      create
         table wrk.want as
      select
         *
      from
         wrk.have
      group
         by stock, put(date,yymm6.)
      having
         sum(ret ne 0) ge 4
    ;quit;
    ');
