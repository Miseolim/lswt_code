``` abap
*&---------------------------------------------------------------------*
*& Report ZCL1_EXCEL_UPLOAD
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*

INCLUDE zcl1_excel_uploadtop                    .    " Global Data

INCLUDE zcl1_excel_uploads01                    .  " Selection screen
INCLUDE zcl1_excel_uploadc01                    .  " ALV Events
INCLUDE zcl1_excel_uploado01                    .  " PBO-Modules
INCLUDE zcl1_excel_uploadi01                    .  " PAI-Modules
INCLUDE zcl1_excel_uploadf01                    .  " FORM-Routines

SELECTION-SCREEN FUNCTION KEY 1.                   " Selection screen button

**********************************************************************
* INITIALIZATION
**********************************************************************
INITIALIZATION.
  PERFORM set_init_value.

**********************************************************************
* AT SELECTION-SCREEN
**********************************************************************
AT SELECTION-SCREEN.
  PERFORM button_control.

AT SELECTION-SCREEN OUTPUT.
  PERFORM modify_screen.

AT SELECTION-SCREEN ON VALUE-REQUEST FOR p_path.
  PERFORM f4_filename.

**********************************************************************
* START-OF-SELECTION
**********************************************************************
START-OF-SELECTION.

  CASE p_gbn.
    WHEN 'E'.
      PERFORM excel_upload.
      PERFORM make_body.
    WHEN 'R'.
      PERFORM get_base_data.
  ENDCASE.

  IF gt_body IS NOT INITIAL.
    CALL SCREEN 100.
  ELSE.
    MESSAGE s037 DISPLAY LIKE 'E'.
  ENDIF.
