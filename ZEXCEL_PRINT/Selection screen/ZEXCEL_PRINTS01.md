``` abap
*&---------------------------------------------------------------------*
*& Include          ZEXCEL_PRINTS01
*&---------------------------------------------------------------------*
SELECTION-SCREEN BEGIN OF BLOCK pa1 WITH FRAME TITLE TEXT-t01.
  PARAMETERS : p_bukrs  TYPE t001-bukrs OBLIGATORY MODIF ID buk.
  SELECTION-SCREEN COMMENT 40(25) p_butxt MODIF ID but.

  SELECT-OPTIONS so_budat FOR bkpf-budat.
SELECTION-SCREEN END OF BLOCK pa1.
PARAMETERS p_pdf  AS CHECKBOX.
