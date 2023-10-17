# utl-select-all-unique-pairs-of-ingredients-in-salads-r-wps-r-python-sql
    %let pgm=utl-select-all-unique-pairs-of-ingredients-in-salads-r-wps-r-python-sql;

    Select all unique pairs of ingredients in salads-r-wps-r-python-sql

    SOLUTIONS

        1 wps sql
        2 wps r sql
        3 wps python sql
        4 wps r base  ( rewrite sa posted solution)


    The  count has to be 1, within a salad there are no duplicated ingredients.
    I have removed the count column and restructure the posted solutions.

    github
    https://tinyurl.com/fmaz896j
    https://github.com/rogerjdeangelis/utl-select-all-unique-pairs-of-ingredients-in-salads-r-wps-r-python-sql

    /stackoverflow
    https://tinyurl.com/72jenu4t
    https://stackoverflow.com/questions/77304003/how-do-i-count-the-number-of-times-each-item-co-occurs-with-every-other-item-in

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
    informat salad $8. INGREDIENTS $11.;
    input  SALAD     INGREDIENTS;
    cards4;
    Green      Lettuce
    Green      Tomato
    Green      Onion
    Green      Olives
    Caprese    Tomato
    Caprese    Onion
    Caprese    Mozzarella
    Fruit      Melon
    Fruit      Orange
    Fruit      Tomato
    ;;;;
    run;quit;

    /****************************************************************************************************************************/
    /*                         |                                             |                                                  */
    /*         INPUT           |                    PROCESS                  |               OUTPUT                             */
    /*                         |                                             |                                                  */
    /*  SALAD     INGREDIENTS  |                                             |                                                  */
    /*                         |                        |                    |     INGREDIENT_ INGREDIENT_                      */
    /* Caprese    Mozzarella   |    Select  the first   | Mozzarella  Onion  |          A           B                           */
    /* Caprese    Onion        |    Mozzarella and join | Mozzarella  Tomato |                                                  */
    /* Caprese    Tomato       |    with all others     |                    |     Lettuce       Olives                         */
    /*                         |                        |                    |     Lettuce       Onion                          */
    /*                         |    Select the second   | Onion       Tomato |     Lettuce       Tomato                         */
    /*                         |    Onion and join with |                    |     Melon         Orange                         */
    /*                         |    all others ie tomato|                    |     Melon         Tomato                         */
    /*                         |                        |                    |     Mozzarella    Onion                          */
    /* Fruit      Melon        |                                             |     Mozzarella    Tomato                         */
    /* Fruit      Orange       |                                             |     Olives        Onion                          */
    /* Fruit      Tomato       |                                             |     Olives        Tomato                         */
    /*                         |                                             |     Onion         Tomato                         */
    /* Green      Lettuce      |                                             |     Orange        Tomato                         */
    /* Green      Olives       |                                             |                                                  */
    /* Green      Onion        |                                             |                                                  */
    /* Green      Tomato       |                                             |                                                  */
    /*                         |                                             |                                                  */
    /************************************************************************|***************************************************/

    /*                                  _
    / | __      ___ __  ___   ___  __ _| |
    | | \ \ /\ / / `_ \/ __| / __|/ _` | |
    | |  \ V  V /| |_) \__ \ \__ \ (_| | |
    |_|   \_/\_/ | .__/|___/ |___/\__, |_|
                 |_|                 |_|
    */

    proc datasets lib=sd1 nolist nodetails;delete want; run;quit;

    %utl_submit_wps64x('

    libname sd1 "d:/sd1";

    options validvarname=any;

    proc sql;
      create
         table sd1.want as
      select
        distinct
         l.ingredients  as  ingredient_a
        ,r.ingredients  as  ingredient_b
      from
         sd1.have as l left join sd1.have as r
      on
             l.ingredients < r.ingredients
         and l.salad       = r.salad
      where
         not missing(r.ingredients)
      order
         by l.ingredients
    ;quit;

    ');

    proc print data=sd1.want;;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  SD1.WANT total obs=11                                                                                                 */
    /*                                                                                                                        */
    /*         INGREDIENT_    INGREDIENT_                                                                                     */
    /*  Obs         A              B                                                                                          */
    /*                                                                                                                        */
    /*    1    Lettuce          Olives                                                                                        */
    /*    2    Lettuce          Onion                                                                                         */
    /*    3    Lettuce          Tomato                                                                                        */
    /*    4    Melon            Orange                                                                                        */
    /*    5    Melon            Tomato                                                                                        */
    /*    6    Mozzarella       Onion                                                                                         */
    /*    7    Mozzarella       Tomato                                                                                        */
    /*    8    Olives           Onion                                                                                         */
    /*    9    Olives           Tomato                                                                                        */
    /*   10    Onion            Tomato                                                                                        */
    /*   11    Orange           Tomato                                                                                        */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*___                                          _
    |___ \  __      ___ __  ___   _ __   ___  __ _| |
      __) | \ \ /\ / / `_ \/ __| | `__| / __|/ _` | |
     / __/   \ V  V /| |_) \__ \ | |    \__ \ (_| | |
    |_____|   \_/\_/ | .__/|___/ |_|    |___/\__, |_|
                     |_|                        |_|
    */
    proc datasets lib=sd1 nolist nodetails;delete want; run;quit;

    libname sd1 "d:/sd1";
    %utl_submit_wps64x('
    libname sd1 "d:/sd1";
    proc r;
    export data=sd1.have r=have;
    submit;
    library(sqldf);
    want <- sqldf("
      select
        distinct
         l.ingredients  as  ingredient_a
        ,r.ingredients  as  ingredient_b
      from
         have as l left join have as r
      on
             l.ingredients < r.ingredients
         and l.salad       = r.salad
      where
         r.ingredients     is not NULL
      order
         by l.ingredients
    ");
    want;
    endsubmit;
    import data=sd1.want r=want;
    run;quit;
    ');

    proc print data=sd1.want width=min;
    run;quit;


    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  The WPS proc R                                                                                                        */
    /*                                                                                                                        */
    /*     ingredient_a ingredient_b                                                                                          */
    /*                                                                                                                        */
    /*  1       Lettuce       Tomato                                                                                          */
    /*  2       Lettuce        Onion                                                                                          */
    /*  3       Lettuce       Olives                                                                                          */
    /*  4         Melon       Orange                                                                                          */
    /*  5         Melon       Tomato                                                                                          */
    /*  6    Mozzarella       Tomato                                                                                          */
    /*  7    Mozzarella        Onion                                                                                          */
    /*  8        Olives       Tomato                                                                                          */
    /*  9        Olives        Onion                                                                                          */
    /*  10        Onion       Tomato                                                                                          */
    /*  11       Orange       Tomato                                                                                          */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*____                                    _   _                             _
    |___ /  __      ___ __  ___   _ __  _   _| |_| |__   ___  _ __    ___  __ _| |
      |_ \  \ \ /\ / / `_ \/ __| | `_ \| | | | __| `_ \ / _ \| `_ \  / __|/ _` | |
     ___) |  \ V  V /| |_) \__ \ | |_) | |_| | |_| | | | (_) | | | | \__ \ (_| | |
    |____/    \_/\_/ | .__/|___/ | .__/ \__, |\__|_| |_|\___/|_| |_| |___/\__, |_|
                     |_|         |_|    |___/                                |_|
    */

    proc datasets lib=sd1 nolist nodetails;delete want; run;quit;

    %utl_submit_wps64x("
    options validvarname=any lrecl=32756;
    libname sd1 'd:/sd1';
    proc python;
    export data=sd1.have python=have;
    submit;
    print(have);
    from os import path;
    import pandas as pd;
    import numpy as np;
    from pandasql import sqldf;
    mysql = lambda q: sqldf(q, globals());
    from pandasql import PandaSQL;
    pdsql = PandaSQL(persist=True);
    sqlite3conn = next(pdsql.conn.gen).connection.connection;
    sqlite3conn.enable_load_extension(True);
    sqlite3conn.load_extension('c:/temp/libsqlitefunctions.dll');
    mysql = lambda q: sqldf(q, globals());
    want = pdsql('''
      select
        distinct
         l.ingredients  as  ingredient_a
        ,r.ingredients  as  ingredient_b
      from
         have as l left join have as r
      on
             l.ingredients < r.ingredients
         and l.salad       = r.salad
      where
         r.ingredients     is not NULL
      order
         by l.ingredients
    ''');
    print(want);
    endsubmit;
    import data=sd1.want python=want;
    run;quit;
    ");

    proc print data=sd1.want;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  WPS PYTHON Procedure                                                                                                  */
    /*                                                                                                                        */
    /*        SALAD  INGREDIENTS                                                                                              */
    /*  0  Green     Lettuce                                                                                                  */
    /*  1  Green     Tomato                                                                                                   */
    /*  2  Green     Onion                                                                                                    */
    /*  3  Green     Olives                                                                                                   */
    /*  4  Caprese   Tomato                                                                                                   */
    /*  5  Caprese   Onion                                                                                                    */
    /*  6  Caprese   Mozzarella                                                                                               */
    /*  7  Fruit     Melon                                                                                                    */
    /*  8  Fruit     Orange                                                                                                   */
    /*  9  Fruit     Tomato                                                                                                   */
    /*                                                                                                                        */
    /*  WPS BASE                                                                                                              */
    /*                                                                                                                        */
    /*     ingredient_a ingredient_b  count                                                                                   */
    /*  0   Lettuce      Tomato           1                                                                                   */
    /*  1   Lettuce      Onion            1                                                                                   */
    /*  2   Lettuce      Olives           1                                                                                   */
    /*  3   Melon        Orange           1                                                                                   */
    /*  4   Melon        Tomato           1                                                                                   */
    /*  5   Mozzarella   Tomato           1                                                                                   */
    /*  6   Mozzarella   Onion            1                                                                                   */
    /*  7   Olives       Tomato           1                                                                                   */
    /*  8   Olives       Onion            1                                                                                   */
    /*  9   Onion        Tomato           1                                                                                   */
    /*  10  Orange       Tomato           1                                                                                   */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*  _                                 _
    | || |   __      ___ __  ___   _ __  | |__   __ _ ___  ___
    | || |_  \ \ /\ / / `_ \/ __| | `__| | `_ \ / _` / __|/ _ \
    |__   _|  \ V  V /| |_) \__ \ | |    | |_) | (_| \__ \  __/
       |_|     \_/\_/ | .__/|___/ |_|    |_.__/ \__,_|___/\___|
                      |_|
    */

    proc datasets lib=sd1 nolist nodetails;delete want; run;quit;

    libname sd1 "d:/sd1";

    %utl_submit_wps64x('
    libname sd1 "d:/sd1";
    proc r;
    export data=sd1.have r=have;
    submit;
    have;
    library(dplyr);
    library(tidyr);
    want <- have %>%
      left_join(., ., by = join_by(SALAD, INGREDIENTS < INGREDIENTS), suffix = c("_A", "_B")) %>%
      filter(complete.cases(.)) %>% distinct(.[,c(2,3)]) ;
    want;
    endsubmit;
    import data=sd1.have r=have;
    run;quit;
    ');

    proc print data=sd1.want;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*   The WPS R Base                                                                                                       */
    /*                                                                                                                        */
    /*        SALAD INGREDIENTS                                                                                               */
    /*   1    Green     Lettuce                                                                                               */
    /*   2    Green      Tomato                                                                                               */
    /*   3    Green       Onion                                                                                               */
    /*   4    Green      Olives                                                                                               */
    /*   5  Caprese      Tomato                                                                                               */
    /*   6  Caprese       Onion                                                                                               */
    /*   7  Caprese  Mozzarella                                                                                               */
    /*   8    Fruit       Melon                                                                                               */
    /*   9    Fruit      Orange                                                                                               */
    /*   10   Fruit      Tomato                                                                                               */
    /*                                                                                                                        */
    /*   WPS                                                                                                                  */
    /*                                                                                                                        */
    /*      INGREDIENTS_A INGREDIENTS_B                                                                                       */
    /*   1        Lettuce        Tomato                                                                                       */
    /*   2        Lettuce         Onion                                                                                       */
    /*   3        Lettuce        Olives                                                                                       */
    /*   4          Onion        Tomato                                                                                       */
    /*   5         Olives        Tomato                                                                                       */
    /*   6         Olives         Onion                                                                                       */
    /*   7     Mozzarella        Tomato                                                                                       */
    /*   8     Mozzarella         Onion                                                                                       */
    /*   9          Melon        Orange                                                                                       */
    /*   10         Melon        Tomato                                                                                       */
    /*   11        Orange        Tomato                                                                                       */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
Select all unique pairs of ingredients in salads-r-wps-r-python
