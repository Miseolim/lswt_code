``` abap
*&---------------------------------------------------------------------*
*& Report ZHTML_EVENT
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*

INCLUDE zhtml_eventtop                          .    " Global Data

INCLUDE zhtml_eventc01                          .  " ALV Events
INCLUDE zhtml_evento01                          .  " PBO-Modules
INCLUDE zhtml_eventi01                          .  " PAI-Modules
INCLUDE zhtml_eventf01                          .  " FORM-Routines


**********************************************************************
* START-OF-SELECTION
**********************************************************************
START-OF-SELECTION.
  PERFORM get_flight_info.

  CALL SCREEN 100.
