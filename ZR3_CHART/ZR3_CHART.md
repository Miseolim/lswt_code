``` abap
*&---------------------------------------------------------------------*
*& Report ZR3_CHART
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*

INCLUDE zr3_charttop                            .    " Global Data

INCLUDE zr3_charts01                            .  " Selection screen
INCLUDE zr3_chartc01                            .  " ALV Events
INCLUDE zr3_charto01                            .  " PBO-Modules
INCLUDE zr3_charti01                            .  " PAI-Modules
INCLUDE zr3_chartf01                            .  " FORM-Routines


**********************************************************************
* INITIALIZATION
**********************************************************************
INITIALIZATION.
  PERFORM set_init_value.

**********************************************************************
* START-OF-SELECTION
**********************************************************************
START-OF-SELECTION.
  PERFORM get_base_data.

  CALL SCREEN 100.
