``` abap
*&---------------------------------------------------------------------*
*& Include          ZALV_TREES01
*&---------------------------------------------------------------------*
SELECTION-SCREEN BEGIN OF BLOCK pa1 WITH FRAME TITLE TEXT-t01.
  SELECT-OPTIONS so_bp  FOR zbut000-partner.
SELECTION-SCREEN END OF BLOCK pa1.
PARAMETERS pa_check AS CHECKBOX.
