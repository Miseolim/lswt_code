``` abap
*&---------------------------------------------------------------------*
*& Include          SAPMZ_CL1_SEL_SCREENS01
*&---------------------------------------------------------------------*
SELECTION-SCREEN BEGIN OF SCREEN 9000 AS SUBSCREEN.
  SELECTION-SCREEN BEGIN OF BLOCK bl1 WITH FRAME TITLE TEXT-t01.
    PARAMETERS     : pa_carr TYPE sflight-carrid.
    SELECT-OPTIONS : so_conn  FOR sflight-connid.
  SELECTION-SCREEN END OF BLOCK bl1.

  SELECTION-SCREEN PUSHBUTTON 1(20) TEXT-b01 USER-COMMAND search.
SELECTION-SCREEN END OF SCREEN 9000.
