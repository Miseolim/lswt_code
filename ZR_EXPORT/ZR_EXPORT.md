``` abap
*&---------------------------------------------------------------------*
*& Report ZREXPORT
*&---------------------------------------------------------------------*
*& Description : ABAP Memory sender
*&---------------------------------------------------------------------*
INCLUDE zr_exporttop.

START-OF-SELECTION.

*-- Get base data
  CLEAR gt_scarr.
  SELECT * INTO CORRESPONDING FIELDS OF TABLE gt_scarr
    FROM scarr
    WHERE carrid IN s_carrid.

  IF sy-subrc EQ 0.
*-- Export to ABAP Memory (Local memory)
    CLEAR gs_scarr.
    READ TABLE gt_scarr INTO gs_scarr INDEX 2.

    EXPORT
      gs_scarr FROM gs_scarr
      gt_scarr FROM gt_scarr
      TO MEMORY ID 'B00'.

*-- Goto target program
    SUBMIT zr_import AND RETURN.

  ENDIF.
