``` abap
*&---------------------------------------------------------------------*
*& Report ZALV_LISTBOX
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*

INCLUDE zalv_listboxtop                         .    " Global Data

INCLUDE zalv_listboxs01                         .  " Selection screen
INCLUDE zalv_listboxc01                         .  " ALV Events
INCLUDE zalv_listboxo01                         .  " PBO-Modules
INCLUDE zalv_listboxi01                         .  " PAI-Modules
INCLUDE zalv_listboxf01                         .  " FORM-Routines


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
  PERFORM screen_change.

  CALL SCREEN 100.
