``` abap
*&---------------------------------------------------------------------*
*& Include          ZSCREEN_F4S01
*&---------------------------------------------------------------------*
SELECTION-SCREEN BEGIN OF BLOCK pa1 WITH FRAME TITLE TEXT-t01.
  PARAMETERS : pa_ctry  TYPE sgeocity-country.
  SELECT-OPTIONS so_city  FOR sgeocity-city.
SELECTION-SCREEN END OF BLOCK pa1.
