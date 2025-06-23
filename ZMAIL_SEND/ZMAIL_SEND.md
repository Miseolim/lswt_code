``` abap
*&---------------------------------------------------------------------*
*& Report ZMAIL_SEND
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*

INCLUDE zmail_sendtop                           .    " Global Data

INCLUDE zmail_sends01                           .  " Selection screen
INCLUDE zmail_sendc01                           .  " ALV Events
INCLUDE zmail_sendo01                           .  " PBO-Modules
INCLUDE zmail_sendi01                           .  " PAI-Modules
INCLUDE zmail_sendf01                           .  " FORM-Routines


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
  PERFORM get_base_data.

  CALL SCREEN 100.
