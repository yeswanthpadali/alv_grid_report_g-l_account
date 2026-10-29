# alv_grid_report_g-l_account
alv grid report for g/l account
REPORT ZFI_GL_PERIOD_BAL2 .


*---------------------------------------------------------------------*
* TYPE POOL
*---------------------------------------------------------------------*
TYPE-POOLS: SLIS.

*---------------------------------------------------------------------*
* ALV DATA
*---------------------------------------------------------------------*
DATA: GT_FIELDCAT TYPE SLIS_T_FIELDCAT_ALV,
      GS_FIELDCAT TYPE SLIS_FIELDCAT_ALV.

DATA: GS_LAYOUT TYPE SLIS_LAYOUT_ALV.

DATA: GT_HEADER TYPE SLIS_T_LISTHEADER,
      GS_HEADER TYPE SLIS_LISTHEADER.

*---------------------------------------------------------------------*
* HEADER DATA
*---------------------------------------------------------------------*
DATA: GV_BUKRS TYPE T001-BUKRS,
      GV_BUTXT TYPE T001-BUTXT,
      GV_RYEAR TYPE FAGLFLEXT-RYEAR,
      GV_RLDNR TYPE FAGLFLEXT-RLDNR,
      GV_WAERS TYPE T001-WAERS.

*---------------------------------------------------------------------*
* G/L ACCOUNT + DESCRIPTION
*---------------------------------------------------------------------*
TYPES: BEGIN OF TY_STU,
         SAKNR TYPE SKA1-SAKNR,
         TXT20 TYPE SKAT-TXT20,
       END OF TY_STU.

DATA: GT_STU TYPE TABLE OF TY_STU,
      GS_STU TYPE TY_STU.

*---------------------------------------------------------------------*
* RAW FAGLFLEXT DATA
*---------------------------------------------------------------------*
TYPES: BEGIN OF TY_TOTAL,
         RBUKRS TYPE FAGLFLEXT-RBUKRS,
         RLDNR  TYPE FAGLFLEXT-RLDNR,
         RYEAR  TYPE FAGLFLEXT-RYEAR,
         RACCT  TYPE FAGLFLEXT-RACCT,
         HSLVT  TYPE FAGLFLEXT-HSLVT,
         DRCRK  TYPE FAGLFLEXT-DRCRK,
         HSL01  TYPE FAGLFLEXT-HSL01,
         HSL02  TYPE FAGLFLEXT-HSL02,
         HSL03  TYPE FAGLFLEXT-HSL03,
         HSL04  TYPE FAGLFLEXT-HSL04,
         HSL05  TYPE FAGLFLEXT-HSL05,
         HSL06  TYPE FAGLFLEXT-HSL06,
         HSL07  TYPE FAGLFLEXT-HSL07,
         HSL08  TYPE FAGLFLEXT-HSL08,
         HSL09  TYPE FAGLFLEXT-HSL09,
         HSL10  TYPE FAGLFLEXT-HSL10,
         HSL11  TYPE FAGLFLEXT-HSL11,
         HSL12  TYPE FAGLFLEXT-HSL12,
         HSL13  TYPE FAGLFLEXT-HSL13,
         HSL14  TYPE FAGLFLEXT-HSL14,
         HSL15  TYPE FAGLFLEXT-HSL15,
         HSL16  TYPE FAGLFLEXT-HSL16,
       END OF TY_TOTAL.

DATA: GT_TOTAL TYPE TABLE OF TY_TOTAL,
      GS_TOTAL TYPE TY_TOTAL.

*---------------------------------------------------------------------*
* AGGREGATED G/L DATA
*---------------------------------------------------------------------*
TYPES: BEGIN OF TY_SUM,
         RACCT TYPE FAGLFLEXT-RACCT,

         HSLVT TYPE FAGLFLEXT-HSLVT,

         HSL01 TYPE FAGLFLEXT-HSL01,
         HSL02 TYPE FAGLFLEXT-HSL02,
         HSL03 TYPE FAGLFLEXT-HSL03,
         HSL04 TYPE FAGLFLEXT-HSL04,
         HSL05 TYPE FAGLFLEXT-HSL05,
         HSL06 TYPE FAGLFLEXT-HSL06,
         HSL07 TYPE FAGLFLEXT-HSL07,
         HSL08 TYPE FAGLFLEXT-HSL08,
         HSL09 TYPE FAGLFLEXT-HSL09,
         HSL10 TYPE FAGLFLEXT-HSL10,
         HSL11 TYPE FAGLFLEXT-HSL11,
         HSL12 TYPE FAGLFLEXT-HSL12,
         HSL13 TYPE FAGLFLEXT-HSL13,
         HSL14 TYPE FAGLFLEXT-HSL14,
         HSL15 TYPE FAGLFLEXT-HSL15,
         HSL16 TYPE FAGLFLEXT-HSL16,

       END OF TY_SUM.

DATA: GT_SUM TYPE TABLE OF TY_SUM,
      GS_SUM TYPE TY_SUM.

*---------------------------------------------------------------------*
* FINAL ALV DATA
*
* Only these fields are displayed:
*
* G/L Account
* Short Description
* P01 - P16
*
*---------------------------------------------------------------------*
TYPES: BEGIN OF TY_ALV,

         SAKNR TYPE SKA1-SAKNR,
         TXT20 TYPE SKAT-TXT20,

         P01 TYPE FAGLFLEXT-HSLVT,
         P02 TYPE FAGLFLEXT-HSLVT,
         P03 TYPE FAGLFLEXT-HSLVT,
         P04 TYPE FAGLFLEXT-HSLVT,
         P05 TYPE FAGLFLEXT-HSLVT,
         P06 TYPE FAGLFLEXT-HSLVT,
         P07 TYPE FAGLFLEXT-HSLVT,
         P08 TYPE FAGLFLEXT-HSLVT,
         P09 TYPE FAGLFLEXT-HSLVT,
         P10 TYPE FAGLFLEXT-HSLVT,
         P11 TYPE FAGLFLEXT-HSLVT,
         P12 TYPE FAGLFLEXT-HSLVT,
         P13 TYPE FAGLFLEXT-HSLVT,
         P14 TYPE FAGLFLEXT-HSLVT,
         P15 TYPE FAGLFLEXT-HSLVT,
         P16 TYPE FAGLFLEXT-HSLVT,

       END OF TY_ALV.

DATA: GT_ALV TYPE TABLE OF TY_ALV,
      GS_ALV TYPE TY_ALV.

*---------------------------------------------------------------------*
* SELECTION SCREEN
*---------------------------------------------------------------------*

DATA: LV_RACCT TYPE SKA1-SAKNR.
*
SELECT-OPTIONS:
  S_RACCT FOR LV_RACCT.

PARAMETERS:
  P_BUKRS TYPE T001-BUKRS default '1000' OBLIGATORY,
  P_RYEAR TYPE FAGLFLEXT-RYEAR default '2026',
  P_RLDNR TYPE FAGLFLEXT-RLDNR default '0l'.

*---------------------------------------------------------------------*
* SELECTION SCREEN VALIDATION
*---------------------------------------------------------------------*

AT SELECTION-SCREEN.

  "Company Code mandatory
  IF P_BUKRS IS INITIAL.

    MESSAGE 'COMPANY CODE IS MANDATORY' TYPE 'E'.

  ENDIF.

  "Fiscal Year mandatory
  IF P_RYEAR IS INITIAL.

    MESSAGE 'FISCAL YEAR IS MANDATORY' TYPE 'E'.

  ENDIF.

  "Ledger mandatory
  IF P_RLDNR IS INITIAL.

    MESSAGE 'LEDGER IS MANDATORY' TYPE 'E'.

  ENDIF.

*---------------------------------------------------------------------*
* START OF SELECTION
*---------------------------------------------------------------------*



START-OF-SELECTION.

*---------------------------------------------------------------------*
* 1. READ COMPANY INFORMATION FROM T001
*
* Required:
* Company Code
* Company Name
* Currency
* Chart of Accounts
* Fiscal Year Variant
*
*---------------------------------------------------------------------*

  SELECT SINGLE BUKRS
                BUTXT
                WAERS
    INTO (GV_BUKRS,
          GV_BUTXT,
          GV_WAERS)
    FROM T001
    WHERE BUKRS = P_BUKRS.

  IF SY-SUBRC <> 0.

    MESSAGE 'COMPANY CODE DOES NOT EXIST IN T001' TYPE 'E'.

  ENDIF.

*---------------------------------------------------------------------*
* Store selected values for header
*---------------------------------------------------------------------*

  GV_RYEAR = P_RYEAR.
  GV_RLDNR = P_RLDNR.

*---------------------------------------------------------------------*
* 2. READ ALL G/L ACCOUNTS FROM SKB1
*
* SKB1 is used as the driving table.
*
* This is important because accounts without postings must also
* appear in the report.
*
*---------------------------------------------------------------------*

  SELECT SAKNR
    INTO TABLE GT_STU
    FROM SKB1
    WHERE BUKRS = P_BUKRS
      AND SAKNR IN S_RACCT.

*---------------------------------------------------------------------*
* 3. READ SHORT DESCRIPTION FROM SKAT
*
* Update each G/L account with its short text.
*
*---------------------------------------------------------------------*

  LOOP AT GT_STU INTO GS_STU.

    SELECT SINGLE TXT20
      INTO GS_STU-TXT20
      FROM SKAT
      WHERE SAKNR = GS_STU-SAKNR
        AND KTOPL = ( SELECT  KTOPL
                        FROM T001
                        WHERE BUKRS = P_BUKRS )
        AND SPRAS = SY-LANGU.

    MODIFY GT_STU FROM GS_STU.

  ENDLOOP.

*---------------------------------------------------------------------*
* 4. READ FAGLFLEXT
*
* Company Code
* Ledger
* Fiscal Year
* G/L Account
* Opening Balance
* Period Balances
* Debit/Credit Indicator
*
*---------------------------------------------------------------------*

  SELECT RBUKRS
         RLDNR
         RYEAR
         RACCT
         HSLVT
         DRCRK
         HSL01
         HSL02
         HSL03
         HSL04
         HSL05
         HSL06
         HSL07
         HSL08
         HSL09
         HSL10
         HSL11
         HSL12
         HSL13
         HSL14
         HSL15
         HSL16

    INTO TABLE GT_TOTAL

    FROM FAGLFLEXT

    WHERE RBUKRS = P_BUKRS
      AND RLDNR  = P_RLDNR
      AND RYEAR  = P_RYEAR
      AND RACCT IN S_RACCT.

*---------------------------------------------------------------------*
* 5. APPLY DEBIT / CREDIT SIGN
*
* SAP standard:
*
* S = Debit  -> Positive
* H = Credit -> Negative
*
*---------------------------------------------------------------------*

 DATA: LV_INDEX TYPE SY-TABIX.

READ TABLE GT_SUM
     WITH KEY RACCT = GS_TOTAL-RACCT
     TRANSPORTING NO FIELDS.

IF SY-SUBRC = 0.

  LV_INDEX = SY-TABIX.

  READ TABLE GT_SUM INTO GS_SUM INDEX LV_INDEX.

  GS_SUM-HSLVT = GS_SUM-HSLVT + GS_TOTAL-HSLVT.
  GS_SUM-HSL01 = GS_SUM-HSL01 + GS_TOTAL-HSL01.
  GS_SUM-HSL02 = GS_SUM-HSL02 + GS_TOTAL-HSL02.
  GS_SUM-HSL03 = GS_SUM-HSL03 + GS_TOTAL-HSL03.
  GS_SUM-HSL04 = GS_SUM-HSL04 + GS_TOTAL-HSL04.
  GS_SUM-HSL05 = GS_SUM-HSL05 + GS_TOTAL-HSL05.
  GS_SUM-HSL06 = GS_SUM-HSL06 + GS_TOTAL-HSL06.
  GS_SUM-HSL07 = GS_SUM-HSL07 + GS_TOTAL-HSL07.
  GS_SUM-HSL08 = GS_SUM-HSL08 + GS_TOTAL-HSL08.
  GS_SUM-HSL09 = GS_SUM-HSL09 + GS_TOTAL-HSL09.
  GS_SUM-HSL10 = GS_SUM-HSL10 + GS_TOTAL-HSL10.
  GS_SUM-HSL11 = GS_SUM-HSL11 + GS_TOTAL-HSL11.
  GS_SUM-HSL12 = GS_SUM-HSL12 + GS_TOTAL-HSL12.
  GS_SUM-HSL13 = GS_SUM-HSL13 + GS_TOTAL-HSL13.
  GS_SUM-HSL14 = GS_SUM-HSL14 + GS_TOTAL-HSL14.
  GS_SUM-HSL15 = GS_SUM-HSL15 + GS_TOTAL-HSL15.
  GS_SUM-HSL16 = GS_SUM-HSL16 + GS_TOTAL-HSL16.

  MODIFY GT_SUM FROM GS_SUM INDEX LV_INDEX.

ELSE.

  CLEAR GS_SUM.

  GS_SUM-RACCT = GS_TOTAL-RACCT.
  GS_SUM-HSLVT = GS_TOTAL-HSLVT.
  GS_SUM-HSL01 = GS_TOTAL-HSL01.
  GS_SUM-HSL02 = GS_TOTAL-HSL02.
  GS_SUM-HSL03 = GS_TOTAL-HSL03.
  GS_SUM-HSL04 = GS_TOTAL-HSL04.
  GS_SUM-HSL05 = GS_TOTAL-HSL05.
  GS_SUM-HSL06 = GS_TOTAL-HSL06.
  GS_SUM-HSL07 = GS_TOTAL-HSL07.
  GS_SUM-HSL08 = GS_TOTAL-HSL08.
  GS_SUM-HSL09 = GS_TOTAL-HSL09.
  GS_SUM-HSL10 = GS_TOTAL-HSL10.
  GS_SUM-HSL11 = GS_TOTAL-HSL11.
  GS_SUM-HSL12 = GS_TOTAL-HSL12.
  GS_SUM-HSL13 = GS_TOTAL-HSL13.
  GS_SUM-HSL14 = GS_TOTAL-HSL14.
  GS_SUM-HSL15 = GS_TOTAL-HSL15.
  GS_SUM-HSL16 = GS_TOTAL-HSL16.

  APPEND GS_SUM TO GT_SUM.

ENDIF.
*---------------------------------------------------------------------*
* 6. AGGREGATE FAGLFLEXT BY G/L ACCOUNT
*
* There can be multiple FAGLFLEXT records for one G/L account.
*
* Therefore first aggregate them.
*
*---------------------------------------------------------------------*
CLEAR GS_SUM.
  LOOP AT GT_TOTAL INTO GS_TOTAL.



    READ TABLE GT_SUM INTO GS_SUM
         WITH KEY RACCT = GS_TOTAL-RACCT.

    IF SY-SUBRC = 0.

      "Existing account - add amounts

      GS_SUM-HSLVT =
        GS_SUM-HSLVT + GS_TOTAL-HSLVT.

      GS_SUM-HSL01 =
        GS_SUM-HSL01 + GS_TOTAL-HSL01.

      GS_SUM-HSL02 =
        GS_SUM-HSL02 + GS_TOTAL-HSL02.

      GS_SUM-HSL03 =
        GS_SUM-HSL03 + GS_TOTAL-HSL03.

      GS_SUM-HSL04 =
        GS_SUM-HSL04 + GS_TOTAL-HSL04.

      GS_SUM-HSL05 =
        GS_SUM-HSL05 + GS_TOTAL-HSL05.

      GS_SUM-HSL06 =
        GS_SUM-HSL06 + GS_TOTAL-HSL06.

      GS_SUM-HSL07 =
        GS_SUM-HSL07 + GS_TOTAL-HSL07.

      GS_SUM-HSL08 =
        GS_SUM-HSL08 + GS_TOTAL-HSL08.

      GS_SUM-HSL09 =
        GS_SUM-HSL09 + GS_TOTAL-HSL09.

      GS_SUM-HSL10 =
        GS_SUM-HSL10 + GS_TOTAL-HSL10.

      GS_SUM-HSL11 =
        GS_SUM-HSL11 + GS_TOTAL-HSL11.

      GS_SUM-HSL12 =
        GS_SUM-HSL12 + GS_TOTAL-HSL12.

      GS_SUM-HSL13 =
        GS_SUM-HSL13 + GS_TOTAL-HSL13.

      GS_SUM-HSL14 =
        GS_SUM-HSL14 + GS_TOTAL-HSL14.

      GS_SUM-HSL15 =
        GS_SUM-HSL15 + GS_TOTAL-HSL15.

      GS_SUM-HSL16 =
        GS_SUM-HSL16 + GS_TOTAL-HSL16.

      MODIFY GT_SUM FROM GS_SUM index sy-tabix.

    ELSE.
      clear gs_sum.

      "First record for this G/L account

      GS_SUM-RACCT = GS_TOTAL-RACCT.

      GS_SUM-HSLVT = GS_TOTAL-HSLVT.

      GS_SUM-HSL01 = GS_TOTAL-HSL01.
      GS_SUM-HSL02 = GS_TOTAL-HSL02.
      GS_SUM-HSL03 = GS_TOTAL-HSL03.
      GS_SUM-HSL04 = GS_TOTAL-HSL04.
      GS_SUM-HSL05 = GS_TOTAL-HSL05.
      GS_SUM-HSL06 = GS_TOTAL-HSL06.
      GS_SUM-HSL07 = GS_TOTAL-HSL07.
      GS_SUM-HSL08 = GS_TOTAL-HSL08.
      GS_SUM-HSL09 = GS_TOTAL-HSL09.
      GS_SUM-HSL10 = GS_TOTAL-HSL10.
      GS_SUM-HSL11 = GS_TOTAL-HSL11.
      GS_SUM-HSL12 = GS_TOTAL-HSL12.
      GS_SUM-HSL13 = GS_TOTAL-HSL13.
      GS_SUM-HSL14 = GS_TOTAL-HSL14.
      GS_SUM-HSL15 = GS_TOTAL-HSL15.
      GS_SUM-HSL16 = GS_TOTAL-HSL16.

      APPEND GS_SUM TO GT_SUM.

    ENDIF.

  ENDLOOP.

*---------------------------------------------------------------------*
* 7. BUILD FINAL ALV DATA
*
* GT_STU is the driving table.
*
* Therefore:
* - Accounts with postings are displayed
* - Accounts without postings are also displayed
*
*---------------------------------------------------------------------*

  LOOP AT GT_STU INTO GS_STU.

    CLEAR GS_ALV.

*---------------------------------------------------------------------*
* G/L Account and Description
*---------------------------------------------------------------------*

    GS_ALV-SAKNR = GS_STU-SAKNR.
    GS_ALV-TXT20 = GS_STU-TXT20.

*---------------------------------------------------------------------*
* Find aggregated amount for this G/L account
*---------------------------------------------------------------------*

    READ TABLE GT_SUM INTO GS_SUM
         WITH KEY RACCT = GS_STU-SAKNR.

    IF SY-SUBRC = 0.
*---------------------------------------------------------------------*
* CUMULATIVE BALANCE
*
* P01 = Opening + P01 movement
*---------------------------------------------------------------------*

      GS_ALV-P01 =
          GS_SUM-HSLVT
        + GS_SUM-HSL01.

*---------------------------------------------------------------------*
* P02 = P01 + P02 movement
*---------------------------------------------------------------------*

      GS_ALV-P02 =
          GS_ALV-P01
        + GS_SUM-HSL02.

*---------------------------------------------------------------------*
* P03
*---------------------------------------------------------------------*

      GS_ALV-P03 =
          GS_ALV-P02
        + GS_SUM-HSL03.

*---------------------------------------------------------------------*
* P04
*---------------------------------------------------------------------*

      GS_ALV-P04 =
          GS_ALV-P03
        + GS_SUM-HSL04.

*---------------------------------------------------------------------*
* P05
*---------------------------------------------------------------------*

      GS_ALV-P05 =
          GS_ALV-P04
        + GS_SUM-HSL05.

*---------------------------------------------------------------------*
* P06
*---------------------------------------------------------------------*

      GS_ALV-P06 =
          GS_ALV-P05
        + GS_SUM-HSL06.

*---------------------------------------------------------------------*
* P07
*---------------------------------------------------------------------*

      GS_ALV-P07 =
          GS_ALV-P06
        + GS_SUM-HSL07.

*---------------------------------------------------------------------*
* P08
*---------------------------------------------------------------------*

      GS_ALV-P08 =
          GS_ALV-P07
        + GS_SUM-HSL08.

*---------------------------------------------------------------------*
* P09
*---------------------------------------------------------------------*

      GS_ALV-P09 =
          GS_ALV-P08
        + GS_SUM-HSL09.

*---------------------------------------------------------------------*
* P10
*---------------------------------------------------------------------*

      GS_ALV-P10 =
          GS_ALV-P09
        + GS_SUM-HSL10.

*---------------------------------------------------------------------*
* P11
*---------------------------------------------------------------------*

      GS_ALV-P11 =
          GS_ALV-P10
        + GS_SUM-HSL11.

*---------------------------------------------------------------------*
* P12
*---------------------------------------------------------------------*

      GS_ALV-P12 =
          GS_ALV-P11
        + GS_SUM-HSL12.

*---------------------------------------------------------------------*
* P13
*---------------------------------------------------------------------*

      GS_ALV-P13 =
          GS_ALV-P12
        + GS_SUM-HSL13.

*---------------------------------------------------------------------*
* P14
*---------------------------------------------------------------------*

      GS_ALV-P14 =
          GS_ALV-P13
        + GS_SUM-HSL14.

*---------------------------------------------------------------------*
* P15
*---------------------------------------------------------------------*

      GS_ALV-P15 =
          GS_ALV-P14
        + GS_SUM-HSL15.

*---------------------------------------------------------------------*
* P16
*---------------------------------------------------------------------*

      GS_ALV-P16 =
          GS_ALV-P15
        + GS_SUM-HSL16.

    ENDIF.

*---------------------------------------------------------------------*
* Add account to final ALV
*---------------------------------------------------------------------*

    APPEND GS_ALV TO GT_ALV.

  ENDLOOP.


*---------------------------------------------------------------------*
* Remove completely zero-balance G/L accounts
*---------------------------------------------------------------------*

  DELETE GT_ALV WHERE P01 = 0
                  AND P02 = 0
                  AND P03 = 0
                  AND P04 = 0
                  AND P05 = 0
                  AND P06 = 0
                  AND P07 = 0
                  AND P08 = 0
                  AND P09 = 0
                  AND P10 = 0
                  AND P11 = 0
                  AND P12 = 0
                  AND P13 = 0
                  AND P14 = 0
                  AND P15 = 0
                  AND P16 = 0.

*---------------------------------------------------------------------*
* 8. BUILD FIELD CATALOG
*---------------------------------------------------------------------*

  PERFORM BUILD_FIELDCAT.

*---------------------------------------------------------------------*
* 9. ALV LAYOUT
*---------------------------------------------------------------------*

  GS_LAYOUT-ZEBRA = 'X'.

  GS_LAYOUT-COLWIDTH_OPTIMIZE = 'X'.

*---------------------------------------------------------------------*
* 10. DISPLAY ALV
*---------------------------------------------------------------------*

  CALL FUNCTION 'REUSE_ALV_GRID_DISPLAY'

    EXPORTING

      I_CALLBACK_PROGRAM     = SY-REPID

      IS_LAYOUT = GS_LAYOUT

      IT_FIELDCAT = GT_FIELDCAT

      I_CALLBACK_TOP_OF_PAGE = 'TOP_OF_PAGE'

    TABLES

      T_OUTTAB = GT_ALV

    EXCEPTIONS

      PROGRAM_ERROR = 1

      OTHERS = 2.

*---------------------------------------------------------------------*
* ALV ERROR
*---------------------------------------------------------------------*

  IF SY-SUBRC <> 0.

    MESSAGE 'ERROR WHILE DISPLAYING ALV' TYPE 'E'.

  ENDIF.

*---------------------------------------------------------------------*
* FORM BUILD_FIELDCAT
*---------------------------------------------------------------------*

FORM BUILD_FIELDCAT.

  CLEAR GT_FIELDCAT.

*---------------------------------------------------------------------*
* G/L ACCOUNT
*---------------------------------------------------------------------*

  PERFORM ADD_FIELDCAT
          USING 'SAKNR'
                'G/L Account'.

*---------------------------------------------------------------------*
* SHORT DESCRIPTION
*---------------------------------------------------------------------*


  PERFORM ADD_FIELDCAT
          USING 'TXT20'
                'Short Description'.

*---------------------------------------------------------------------*
* POSTING PERIODS
*---------------------------------------------------------------------*

  PERFORM ADD_FIELDCAT USING 'P01' 'P01'.

  PERFORM ADD_FIELDCAT USING 'P02' 'P02'.

  PERFORM ADD_FIELDCAT USING 'P03' 'P03'.

  PERFORM ADD_FIELDCAT USING 'P04' 'P04'.

  PERFORM ADD_FIELDCAT USING 'P05' 'P05'.

  PERFORM ADD_FIELDCAT USING 'P06' 'P06'.

  PERFORM ADD_FIELDCAT USING 'P07' 'P07'.

  PERFORM ADD_FIELDCAT USING 'P08' 'P08'.

  PERFORM ADD_FIELDCAT USING 'P09' 'P09'.

  PERFORM ADD_FIELDCAT USING 'P10' 'P10'.

  PERFORM ADD_FIELDCAT USING 'P11' 'P11'.

  PERFORM ADD_FIELDCAT USING 'P12' 'P12'.

  PERFORM ADD_FIELDCAT USING 'P13' 'P13'.

  PERFORM ADD_FIELDCAT USING 'P14' 'P14'.

  PERFORM ADD_FIELDCAT USING 'P15' 'P15'.

  PERFORM ADD_FIELDCAT USING 'P16' 'P16'.

ENDFORM.

*---------------------------------------------------------------------*
* FORM ADD_FIELDCAT
*---------------------------------------------------------------------*

FORM ADD_FIELDCAT USING P_FIELDNAME
                        P_SELTEXT.

  CLEAR GS_FIELDCAT.

  GS_FIELDCAT-FIELDNAME = P_FIELDNAME.

  GS_FIELDCAT-SELTEXT_M = P_SELTEXT.


  APPEND GS_FIELDCAT TO GT_FIELDCAT.

ENDFORM.

*---------------------------------------------------------------------*
* FORM TOP_OF_PAGE
*
* Header displayed above ALV
*---------------------------------------------------------------------*



  FORM TOP_OF_PAGE.

  CLEAR GT_HEADER.
  CLEAR GS_HEADER.


  "Company Code
  CLEAR GS_HEADER.
  GS_HEADER-TYP  = 'S'.
  GS_HEADER-KEY  = 'Company Code :'.
  GS_HEADER-INFO = GV_BUKRS.
  APPEND GS_HEADER TO GT_HEADER.

  "Company Name
  CLEAR GS_HEADER.
  GS_HEADER-TYP  = 'S'.
  GS_HEADER-KEY  = 'Company Name :'.
  GS_HEADER-INFO = GV_BUTXT.
  APPEND GS_HEADER TO GT_HEADER.

  "Fiscal Year
  CLEAR GS_HEADER.
  GS_HEADER-TYP  = 'S'.
  GS_HEADER-KEY  = 'Fiscal Year :'.
  GS_HEADER-INFO = GV_RYEAR.
  APPEND GS_HEADER TO GT_HEADER.

  "Ledger
  CLEAR GS_HEADER.
  GS_HEADER-TYP  = 'S'.
  GS_HEADER-KEY  = 'Ledger :'.
  GS_HEADER-INFO = GV_RLDNR.
  APPEND GS_HEADER TO GT_HEADER.

  "Currency
  CLEAR GS_HEADER.
  GS_HEADER-TYP  = 'S'.
  GS_HEADER-KEY  = 'Currency :'.
  GS_HEADER-INFO = GV_WAERS.
  APPEND GS_HEADER TO GT_HEADER.

  CALL FUNCTION 'REUSE_ALV_COMMENTARY_WRITE'
    EXPORTING
      IT_LIST_COMMENTARY = GT_HEADER.

ENDFORM.
