``` abap
*&---------------------------------------------------------------------*
*& Include          ZCL1_EXCEL_UPLOADS01
*&---------------------------------------------------------------------*
SELECTION-SCREEN BEGIN OF BLOCK pa1 WITH FRAME TITLE TEXT-t01.
  PARAMETERS : p_bukrs  TYPE t001-bukrs OBLIGATORY MODIF ID buk.
  SELECTION-SCREEN COMMENT 40(25) p_butxt MODIF ID but.
  PARAMETERS : p_gjahr  TYPE bkpf-gjahr OBLIGATORY MODIF ID gjr,
               p_gbn    AS LISTBOX VISIBLE LENGTH 15 OBLIGATORY
                          USER-COMMAND gbn DEFAULT 'R' MODIF ID doc.

  PARAMETERS : p_path TYPE rlgrap-filename MODIF ID pat.
SELECTION-SCREEN END OF BLOCK pa1.
