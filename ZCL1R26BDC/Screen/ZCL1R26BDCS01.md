``` abap
*&---------------------------------------------------------------------*
*& Include          ZCL1R26BDCS01
*&---------------------------------------------------------------------*
SELECTION-SCREEN BEGIN OF BLOCK pa1 WITH FRAME TITLE TEXT-t01.
  PARAMETERS : pa_bukrs TYPE bkpf-bukrs MODIF ID buk.
  SELECTION-SCREEN COMMENT 40(25) p_butxt MODIF ID but.

  PARAMETERS : pa_budat TYPE bkpf-budat,
               pa_waers TYPE bkpf-waers OBLIGATORY,
               pa_mode  TYPE ctu_params-dismode AS LISTBOX VISIBLE LENGTH 30
                             DEFAULT 'N'.
SELECTION-SCREEN END OF BLOCK pa1.

SELECTION-SCREEN BEGIN OF BLOCK pa2 WITH FRAME TITLE TEXT-t02.
  SELECTION-SCREEN BEGIN OF LINE.
    SELECTION-SCREEN COMMENT (50) TEXT-i01.
  SELECTION-SCREEN END OF LINE.

  SELECTION-SCREEN BEGIN OF LINE.
    SELECTION-SCREEN COMMENT (50) TEXT-i02.
  SELECTION-SCREEN END OF LINE.
SELECTION-SCREEN END OF BLOCK pa2.

----------------------------------------------------------------------------------
Extracted by Direct Download Enterprise version 1.3.1 - E.G.Mellodew. 1998-2005 UK. Sap Release 758
