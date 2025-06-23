``` abap
*&---------------------------------------------------------------------*
*& Report ZEXCEL_PRINT
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*

INCLUDE zexcel_printtop                         .    " Global Data

INCLUDE zexcel_prints01                         .  " Selection scren
INCLUDE zexcel_printc01                         .  " ALV Events
INCLUDE zexcel_printo01                         .  " PBO-Modules
INCLUDE zexcel_printi01                         .  " PAI-Modules
INCLUDE zexcel_printf01                         .  " FORM-Routines


**********************************************************************
* INITIALIZATION
**********************************************************************
INITIALIZATION.
  PERFORM set_init_value.

**********************************************************************
* AT SELECTION-SCREEN
**********************************************************************
AT SELECTION-SCREEN OUTPUT.
  PERFORM modify_screen.

**********************************************************************
* START-OF-SELECTION
**********************************************************************
START-OF-SELECTION.
  PERFORM get_document_data.

  CALL SCREEN 100.
