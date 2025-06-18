``` abap
*&---------------------------------------------------------------------*
*& Report ZALV_F4_EVENT
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*

INCLUDE zalv_f4_eventtop                        .    " Global Data

INCLUDE zalv_f4_events01                        .  " Selection screen
INCLUDE zalv_f4_eventc01                        .  " ALV Events
INCLUDE zalv_f4_evento01                        .  " PBO-Modules
INCLUDE zalv_f4_eventi01                        .  " PAI-Modules
INCLUDE zalv_f4_eventf01                        .  " FORM-Routines


**********************************************************************
* START-OF-SELECTION
**********************************************************************
START-OF-SELECTION.
  PERFORM get_base_data.
  PERFORM get_search_help_data.

  CALL SCREEN 100.
