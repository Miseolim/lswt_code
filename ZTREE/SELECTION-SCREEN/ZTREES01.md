``` abap
*&---------------------------------------------------------------------*
*& Include          ZTREES01
*&---------------------------------------------------------------------*
SELECTION-SCREEN BEGIN OF BLOCK pa1 WITH FRAME TITLE TEXT-t01.
  SELECT-OPTIONS so_car FOR sflight-carrid.
SELECTION-SCREEN END OF BLOCK pa1.
