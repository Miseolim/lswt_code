``` abap
*&---------------------------------------------------------------------*
*& Report ZR_IMPORT
*&---------------------------------------------------------------------*
*& Description : ABAP Memory receiver
*&---------------------------------------------------------------------*

INCLUDE zr_importtop                            .    " Global Data

START-OF-SELECTION.

*-- Receive data from Abap memory
  IMPORT
    gs_scarr TO gs_scarr
    gt_scarr TO gt_scarr
    FROM MEMORY ID 'B00'.

  IF gt_scarr IS INITIAL.
    MESSAGE s037(k5) DISPLAY LIKE 'E'.
    STOP.
  ENDIF.

  LOOP AT gt_scarr INTO gs_scarr.

    WRITE :/ gs_scarr-carrid, gs_scarr-carrname, gs_scarr-url.

  ENDLOOP.

  CLEAR : gt_scarr, gs_scarr.

  EXPORT
      space
*      gs_scarr FROM gs_scarr
*      gt_scarr FROM gt_scarr
      TO MEMORY ID 'B00'.

  IMPORT
    gs_scarr TO gs_scarr
    gt_scarr TO gt_scarr
    FROM MEMORY ID 'B00'.

  IF gt_scarr IS INITIAL.

  ENDIF.
