``` abap
*&---------------------------------------------------------------------*
*& Include          ZALV_F4_EVENTS01
*&---------------------------------------------------------------------*
SELECTION-SCREEN BEGIN OF BLOCK pa1 WITH FRAME TITLE TEXT-t01.
  SELECT-OPTIONS so_saknr FOR zictfi_ska1-saknr.
SELECTION-SCREEN END OF BLOCK pa1.
