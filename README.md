# utl_sum_and_standard_deviation_of_the_next_five_earnigs_by_group
Sum and standard deviation of the next five earnigs by group.  Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.


    Sum and standard deviation of the next five earnigs by group

     Two more solutions added

     Same result in WPS and SAS for all thre solutions.

     Three Solutions

        1. Double Sort solution
        2. DOW solution
        3. Mark Keintz (merge solution)

    github (you will have better luck copying and pasting the .sas file (excact dup of readme)
    https://github.com/rogerjdeangelis/utl_sum_and_standard_deviation_of_the_next_five_earnigs_by_group

    see
    https://communities.sas.com/t5/Base-SAS-Programming/Summing-over-intervals-of-time-within-a-group/m-p/466268

    You could do it without the sorts with a DOW loop?
    A little too lazy.

    INPUT
    =====

     WORK.HAVE total obs=12
                                    |      RULES (add these variables)_
     COMPANY_                       |
        ID       YEAR    EARNINGS   |   SUM                          STD
                                    |
        123      2000       5.5     |    36   6.7+7.1+7.2+6.9+8.1   0.539  std(of 6.7,7.1,7.2,6.9,8.1)
        123      2001       6.7     |     .                           .
        123      2002       7.1     |     .                           .
        123      2003       7.2     |     .                           .
        123      2004       6.9     |     .                           .
        123      2005       8.1     |     .                           .
                                    |
        456      2000      15.5     |    96                         4.979
        456      2001      16.7     |    .                            .
        456      2002      17.1     |    .                            .
        456      2003      17.2     |    .                            .
        456      2004      16.9     |    .                            .
        456      2005      28.1     |    .                            .


    PROCESS
    =======


    1. Double Sort solution
    ------------------------

     proc sort data=have out=havSrt;
       by company_id descending year;
     run;quit;

     data havRev;

       retain earn1-earn5;
       array earns[2001:2005] earn1-earn5;

       set havSrt;
       by company_id;

       * last is year 2000;
       if NOT last.company_id then do;
          earns[year]=earnings;
       end;

       if last.company_id then do;
          earning5=sum(of earns[*]);
          std5=std(of earns[*]);
       end;
       drop earn1-earn5 ;
     run;quit;

     proc sort data=havRev out=want;
       by company_id year;
     run;quit;


    2. DOW solution
    ---------------

     data want;

      retain earn1-earn5 earning5 std5 ;
      array earns[2001:2005] earn1-earn5;

      do until(last.company_id);
       set have;
       by company_id;

         if NOT first.company_id then do;
            earns[year]=earnings;
         end;

         if last.company_id then do;
            earning5 = sum(of earns[*]);
            std5     = std(of earns[*]);
         end;
      end;

      do until(last.company_id);
       set have;
       by company_id;
       if not first.company_id then do;
          earning5 =.;
          std5     =.;
       end;
       output;
      end;

      drop earn1-earn5 ;

     run;quit;



    3. Mark Keintz (merge solution)
    -------------------------------

     data want (drop=_:);
       merge
        have
        have (firstobs=2 keep=earnings rename=(earnings=_e1))
        have (firstobs=3 keep=earnings rename=(earnings=_e2))
        have (firstobs=4 keep=earnings rename=(earnings=_e3))
        have (firstobs=5 keep=earnings rename=(earnings=_e4))
        have (firstobs=6 keep=company_id earnings
                 rename=(earnings=_e5 company_id=_id5));

       if _id5=company_id then do;
          earnings_5yr=sum(of _e:);
          std_dev=std(of _e:);
       end;
     run;


    OUTPUT
    ======

     WORK.WANT total obs=12

     COMPANY_
        ID       YEAR    EARNINGS    EARNING5      STD5

        123      2000       5.5         36       0.53852
        123      2001       6.7          .        .
        123      2002       7.1          .        .
        123      2003       7.2          .        .
        123      2004       6.9          .        .
        123      2005       8.1          .        .
        456      2000      15.5         96       4.97896
        456      2001      16.7          .        .
        456      2002      17.1          .        .
        456      2003      17.2          .        .
        456      2004      16.9          .        .
        456      2005      28.1          .        .
    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;

    data have;
    input company_id Year Earnings;
    cards4;
    123 2000 5.5
    123 2001 6.7
    123 2002 7.1
    123 2003 7.2
    123 2004 6.9
    123 2005 8.1
    456 2000 15.5
    456 2001 16.7
    456 2002 17.1
    456 2003 17.2
    456 2004 16.9
    456 2005 28.1
    ;;;;
    run;quit;
    *
    __      ___ __  ___
    \ \ /\ / / '_ \/ __|
     \ V  V /| |_) \__ \
      \_/\_/ | .__/|___/
             |_|
    ;


    1. Double Sort solution
    2. DOW solution
    3. Mark Keintz (merge solution)

    %utl_submit_wps64('
    libname wrk sas7bdat "%sysfunc(pathname(work))";
     proc sort data=wrk.have out=havSrt;
       by company_id descending year;
     run;quit;

     data havRev;

       retain earn1-earn5;
       array earns[2001:2005] earn1-earn5;

       set havSrt;
       by company_id;

       if not last.company_id then do;
          earns[year]=earnings;
       end;

       if last.company_id then do;
          earning5=sum(of earns[*]);
          std5=std(of earns[*]);
       end;
       drop earn1-earn5 ;
     run;quit;

     proc sort data=havRev out=wrk.wantps;
       by company_id year;
     run;quit;
    run;quit;
    ');


    %utl_submit_wps64('
    libname wrk sas7bdat "%sysfunc(pathname(work))";
    data want;

     retain earn1-earn5 earning5 std5 ;
     array earns[2001:2005] earn1-earn5;

     do until(last.company_id);
      set wrk.have;
      by company_id;

        if NOT first.company_id then do;
           earns[year]=earnings;
        end;

        if last.company_id then do;
           earning5 = sum(of earns[*]);
           std5     = std(of earns[*]);
        end;
     end;

     do until(last.company_id);
      set wrk.have;
      by company_id;
      if not first.company_id then do;
         earning5 =.;
         std5     =.;
      end;
      output;
     end;

     drop earn1-earn5 ;

    run;quit;

    proc print;
    run;quit;

    ');


    %utl_submit_wps64('
    libname wrk sas7bdat "%sysfunc(pathname(work))";
     data want (drop=_:);
       merge
        wrk.have
        wrk.have (firstobs=2 keep=earnings rename=(earnings=_e1))
        wrk.have (firstobs=3 keep=earnings rename=(earnings=_e2))
        wrk.have (firstobs=4 keep=earnings rename=(earnings=_e3))
        wrk.have (firstobs=5 keep=earnings rename=(earnings=_e4))
        wrk.have (firstobs=6 keep=company_id earnings
                 rename=(earnings=_e5 company_id=_id5));

       if _id5=company_id then do;
          earnings_5yr=sum(of _e:);
          std_dev=std(of _e:);
       end;
     run;
    ');
