``` abap
*&---------------------------------------------------------------------*
*& Include          ZFILE_ATTACHS01
*&---------------------------------------------------------------------*
SELECTION-SCREEN BEGIN OF BLOCK bl1 WITH FRAME TITLE TEXT-t01.
  SELECT-OPTIONS : so_ebeln FOR ekko-ebeln.
SELECTION-SCREEN END OF BLOCK bl1.
