``` abap
*&---------------------------------------------------------------------*
*& Include ZSCREEN_F4TOP                            - Report ZSCREEN_F4
*&---------------------------------------------------------------------*
REPORT zscreen_f4 MESSAGE-ID k5.


**********************************************************************
* TABLES
**********************************************************************
TABLES : sgeocity.

**********************************************************************
* Macro
**********************************************************************
DEFINE _init.

  REFRESH &1.
  CLEAR &1.

END-OF-DEFINITION.

**********************************************************************
* Internal table and Work area
**********************************************************************
DATA : BEGIN OF gs_city_value,
         country TYPE sgeocity-country,
         city    TYPE sgeocity-city,
       END OF gs_city_value,
       gt_city_value LIKE TABLE OF gs_city_value.

DATA : BEGIN OF gs_country_value,
         country TYPE sgeocity-country,
       END OF gs_country_value,
       gt_country_value LIKE TABLE OF gs_country_value.
