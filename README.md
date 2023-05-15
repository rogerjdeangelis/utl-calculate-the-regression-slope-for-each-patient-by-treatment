# utl-calculate-the-regression-slope-for-each-patient-by-treatment
Calculate the regression slope for each patient by treatment
    %let pgm=utl-calculate-the-regression-slope-for-each-patient-by-treatment;

    Calculate the regression slope for each patient by treatment

      Three Solutions

           1. sas proc reg  (lease code)
           2. wps proc r
              NICE - shows the flexibility of R - missing in SAS/WPS
              even proc matrix is not as elegant as r - need an external module)
              This is why you need R
              https://stackoverflow.com/users/12158757/thomasiscoding
           3. wps proc reg
           4. wps reg skinny


    github
    https://tinyurl.com/bdcmpa3s
    https://github.com/rogerjdeangelis/utl-calculate-the-regression-slope-for-each-patient-by-treatment

    StackOverflow R
    https://tinyurl.com/487ty7ff
    https://stackoverflow.com/questions/65728742/calculate-the-slope-for-each-individual?noredirect=1&lq=1

    https://stackoverflow.com/users/12158757/thomasiscoding

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    libname sd1 "d:/sd1";

    data have sd1.have;informat
    ID 8.
    TIME 8.
    TRT1 8.
    TRT2 8.
    TRT3 8.
    TRT4 8.
    ;input
    ID TIME TRT1 TRT2 TRT3 TRT4;
    cards4;
    1 0 12.75 114.25 44.79 20.62
    1 12 8.11 112.01 53.61 18.52
    1 36 12.48 95.17 31.16 23.19
    2 0 13.66 121.16 61.47 18.65
    2 7 15.57 115.81 57.63 22.01
    2 23 9.58 95.86 54.07 18.66
    2 68 16.54 133.5 46.56 25.26
    3 0 9.92 75.63 65.17 20.36
    3 23 10.61 91.39 52.87 19.2
    4 0 11.7 102.4 30.86 21.82
    4 32 15.61 110.22 37.32 16.96
    4 45 15.34 94.87 41.03 18.63
    ;;;;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                       | RULES Compute the regression slope by trt and id               */
    /*                                                       |                                                                */
    /*                                                       |                                                                */
    /* Up to 40 obs from last table SD1.HAVE total obs=12 14 |  Model trt# = time                                             */
    /*                                                       |                                                                */
    /* Obs    ID     TRT1     TRT2      TRT3     TRT4  TIME  | ID   TRT4  TIME   TRT4  slope                                  */
    /*                                                       |                                                                */
    /*   1     1    12.75    114.25    44.79    20.62    0   |  1  20.62    0    0.0890                                       */
    /*   2     1     8.11    112.01    53.61    18.52   12   |  1  18.52   12                                                 */
    /*   3     1    12.48     95.17    31.16    23.19   36   |  1  23.19   36                                                 */
    /*                                                       |                                                                */
    /*   4     2    13.66    121.16    61.47    18.65    0   |   Do this by id and trt;                                       */
    /*   5     2    15.57    115.81    57.63    22.01    7   |                                                                */
    /*   6     2     9.58     95.86    54.07    18.66   23   |   proc reg data=sd1.have(where=(id=1))                         */
    /*   7     2    16.54    133.50    46.56    25.26   68   |     outest=want1(keep=id time) ;                               */
    /*   8     3     9.92     75.63    65.17    20.36    0   |   by id ;                                                      */
    /*   9     3    10.61     91.39    52.87    19.20   23   |   model trt4 = time ;                                          */
    /*  10     4    11.70    102.40    30.86    21.82    0   |   run;quit;                                                    */
    /*  11     4    15.61    110.22    37.32    16.96   32   |                                                                */
    /*  12     4    15.34     94.87    41.03    18.63   45   |  Up to 40 obs from WANT1 total obs=1                           */
    /*                                                       |  Obs    ID      TIME                                           */
    /*                                                       |                                                                */
    /*                                                       |   1      1    0.088988                                         */
    /*                                                       |                                                                */
    /**************************************************************************************************************************/

    /*           _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| `_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    */

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* Up to 40 obs from WANT with formatted variables  total obs=4 14MAY2023:17:42:30                                        */
    /*                                                                                                                        */
    /* Obs    ID      TRT1        TRT2        TRT3         TRT4                                                               */
    /*                                                                                                                        */
    /*  1      1    0.019583    -0.55452    -0.45815     0.088988                                                             */
    /*  2      2    0.034979     0.23862    -0.20360     0.081657                                                             */
    /*  3      3    0.030000     0.68522    -0.53478    -0.050435                                                             */
    /*  4      4    0.088692    -0.08955     0.22144    -0.086190                                                             */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*
    / |  ___  __ _ ___   _ __  _ __ ___   ___   _ __ ___  __ _
    | | / __|/ _` / __| | `_ \| `__/ _ \ / __| | `__/ _ \/ _` |
    | | \__ \ (_| \__ \ | |_) | | | (_) | (__  | | |  __/ (_| |
    |_| |___/\__,_|___/ | .__/|_|  \___/ \___| |_|  \___|\__, |
                        |_|                              |___/
    */

    /*--- Minimal code ----*/
    data want;
       retain id 1;
    run;quit;

    %array(_trt,values=1-4);

    %do_over(_trt,phrase=%str(
       proc reg data=have OUTEST=havEst?(keep=id time);
          by id;
          model trt? = time;
       run;quit;
       data want;
         merge  want havEst?(rename=time=trt?);
       run;quit;
       ));

    /*___
    |___ \  __      ___ __  ___   _ __  _ __ ___   ___   _ __
      __) | \ \ /\ / / `_ \/ __| | `_ \| `__/ _ \ / __| | `__|
     / __/   \ V  V /| |_) \__ \ | |_) | | | (_) | (__  | |
    |_____|   \_/\_/ | .__/|___/ | .__/|_|  \___/ \___| |_|
                     |_|         |_|
    */

    %let _pth=%sysfunc(pathname(work));

    %utl_submit_wps64('

    libname wrk "&_pth";

    proc r;
    export data=wrk.have r=have;
    submit;
    library(dplyr);
    want<-have %>%
       group_by(ID) %>%
       summarise(across(starts_with("TRT"),
                        list(slope = ~lm(. ~ TIME)$coef[2])));
    endsubmit;
    import data=wrk.want_r r=want;
    proc print data=wrk.want_r;
    format trt: 9.6;
    run;quit;
    ');

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* The WPS System                                                                                                         */
    /*                                                                                                                        */
    /* Obs    ID    TRT1_SLOPE    TRT2_SLOPE    TRT3_SLOPE    TRT4_SLOPE                                                      */
    /*                                                                                                                        */
    /*  1      1      0.019583     -0.554524     -0.458155      0.088988                                                      */
    /*  2      2      0.034979      0.238617     -0.203600      0.081657                                                      */
    /*  3      3      0.030000      0.685217     -0.534783     -0.050435                                                      */
    /*  4      4      0.088692     -0.089546      0.221442     -0.086190                                                      */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*____
    |___ /  __      ___ __  ___   _ __  _ __ ___   ___   _ __ ___  __ _
      |_ \  \ \ /\ / / `_ \/ __| | `_ \| `__/ _ \ / __| | `__/ _ \/ _` |
     ___) |  \ V  V /| |_) \__ \ | |_) | | | (_) | (__  | | |  __/ (_| |
    |____/    \_/\_/ | .__/|___/ | .__/|_|  \___/ \___| |_|  \___|\__, |
                     |_|         |_|                              |___/
    */

    %utl_wpsbegin;
    parmcards4;

    libname sd1 "d:/sd1";

    data want;
       retain id 1;
    run;quit;

    %array(_trt,values=1-4);

    %do_over(_trt,phrase=%str(
       proc reg data=sd1.have OUTEST=havEst?(keep=id time);
          by id;
          model trt? = time;
       run;quit;
       data want;
         format trt: 5.3;
         merge want havEst?(rename=time=time?);
       run;quit;
       ));
    proc print data=want(rename=(time1-time4=trt1-trt4));
    format trt: 9.6;
    run;quit;
    ;;;;
    %utl_wpsend;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* The WPS System                                                                                                         */
    /*                                                                                                                        */
    /* Obs    ID         trt1         trt2         trt3         trt4                                                          */
    /*                                                                                                                        */
    /*  1      1     0.019583    -0.554524    -0.458155     0.088988                                                          */
    /*  2      2     0.034979     0.238617    -0.203600     0.081657                                                          */
    /*  3      3     0.030000     0.685217    -0.534783    -0.050435                                                          */
    /*  4      4     0.088692    -0.089546     0.221442    -0.086190                                                          */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*  _                               _    _
    | || |    __      ___ __  ___   ___| | _(_)_ __  _ __  _   _
    | || |_   \ \ /\ / / `_ \/ __| / __| |/ / | `_ \| `_ \| | | |
    |__   _|   \ V  V /| |_) \__ \ \__ \   <| | | | | | | | |_| |
       |_|(_)   \_/\_/ | .__/|___/ |___/_|\_\_|_| |_|_| |_|\__, |
                       |_|                                 |___/
    */
    %let _pth=%sysfunc(pathname(work));

    libname wrk "&_pth";

    %utl_submit_wps64('

    libname wrk "&_pth";

    proc transpose data=wrk.have out=havXpo(rename=(_name_=trt col1=val));
     by id time;
     var trt1-trt4;
    run;quit;

    proc sort data=havXpo out=havSrt;
     by id trt time;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  p to 40 obs from HAVSRT total obs=48 15MAY2023:07:12:41                                                               */
    /*  bs    ID    TIME    TRT       VAL                                                                                     */
    /*                                                                                                                        */
    /*   1     1      0     TRT1     12.75                                                                                    */
    /*   2     1     12     TRT1      8.11                                                                                    */
    /*   3     1     36     TRT1     12.48                                                                                    */
    /*   4     1      0     TRT2    114.25                                                                                    */
    /*   5     1     12     TRT2    112.01                                                                                    */
    /*   6     1     36     TRT2     95.17                                                                                    */
    /*   7     1      0     TRT3     44.79                                                                                    */
    /*   8     1     12     TRT3     53.61                                                                                    */
    /*   9     1     36     TRT3     31.16                                                                                    */
    /*  10     1      0     TRT4     20.62                                                                                    */
    /*  11     1     12     TRT4     18.52                                                                                    */
    /*  12     1     36     TRT4     23.19                                                                                    */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    proc reg data=havSrt OUTEST=havEst(keep=id trt time);
       by id trt;
       model val = time;
    run;quit;

    proc transpose data=havEst out=want_wps;
    by id;
    id trt;
    var time;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* Up to 40 obs from HAVEST total obs=16 15MAY2023:07:15:20                                                               */
    /*                                                                                                                        */
    /* Obs    ID    TRT       TIME                                                                                            */
    /*                                                                                                                        */
    /*   1     1    TRT1     0.01958                                                                                          */
    /*   2     1    TRT2    -0.55452                                                                                          */
    /*   3     1    TRT3    -0.45815                                                                                          */
    /*   4     1    TRT4     0.08899                                                                                          */
    /*   5     2    TRT1     0.03498                                                                                          */
    /*   6     2    TRT2     0.23862                                                                                          */
    /*   7     2    TRT3    -0.20360                                                                                          */
    /*   8     2    TRT4     0.08166                                                                                          */
    /*   9     3    TRT1     0.03000                                                                                          */
    /*  10     3    TRT2     0.68522                                                                                          */
    /*  11     3    TRT3    -0.53478                                                                                          */
    /*  12     3    TRT4    -0.05043                                                                                          */
    /*  13     4    TRT1     0.08869                                                                                          */
    /*  14     4    TRT2    -0.08955                                                                                          */
    /*  15     4    TRT3     0.22144                                                                                          */
    /*  16     4    TRT4    -0.08619                                                                                          */
    /*                                                                                                                        */
    /**************************************************************************************************************************/
    proc print data=want_wps;
    format trt: 9.6;
    run;quit;

    ');

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* The WPS System                                                                                                         */
    /*                                                                                                                        */
    /* Obs    ID    _NAME_         TRT1         TRT2         TRT3         TRT4                                                */
    /*                                                                                                                        */
    /*  1      1     TIME      0.019583    -0.554524    -0.458155     0.088988                                                */
    /*  2      2     TIME      0.034979     0.238617    -0.203600     0.081657                                                */
    /*  3      3     TIME      0.030000     0.685217    -0.534783    -0.050435                                                */
    /*  4      4     TIME      0.088692    -0.089546     0.221442    -0.086190                                                */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
