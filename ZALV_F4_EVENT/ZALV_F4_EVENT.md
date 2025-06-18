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

*GUI Texts
*----------------------------------------------------------
* TITLE100 --> [C1] Search help event in ALV
* TITLE100 --> [C1] Search help event in ALV

*Text elements
*----------------------------------------------------------
* E01 Top of page event error
* T01 Condition


*Selection texts
*----------------------------------------------------------
* SO_SAKNR D       .


*Messages
*----------------------------------------------------------
*
* Message class: K5
*001   & & & &
*037   No data found

----------------------------------------------------------------------------------
Extracted by Direct Download Enterprise version 1.3.1 - E.G.Mellodew. 1998-2005 UK. Sap Release 758
