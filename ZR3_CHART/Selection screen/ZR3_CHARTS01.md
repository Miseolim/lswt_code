``` abap
*&---------------------------------------------------------------------*
*& Include          ZR3_CHARTS01
*&---------------------------------------------------------------------*
SELECTION-SCREEN BEGIN OF BLOCK pa1 WITH FRAME TITLE TEXT-t01.
  PARAMETERS : pa_carr  TYPE scarr-carrid OBLIGATORY.
SELECTION-SCREEN END OF BLOCK pa1.

SELECTION-SCREEN BEGIN OF BLOCK pa2 WITH FRAME TITLE TEXT-t02.
  PARAMETERS : p_column RADIOBUTTON GROUP rb1,
               p_line   RADIOBUTTON GROUP rb1.
SELECTION-SCREEN END OF BLOCK pa2.
