``` abap
*&---------------------------------------------------------------------*
*& Report ZCL1R26BDC
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*

INCLUDE zcl1r26bdctop.                            "    Global declaration

INCLUDE zcl1r26bdcs01.                            "  Selection screen
INCLUDE zcl1r26bdcc01.                            "  ALV Events
INCLUDE zcl1r26bdco01.                            "  PBO Module
INCLUDE zcl1r26bdci01.                            "  PAI Module
INCLUDE zcl1r26bdcf01.                            "  FORM Routines


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

  CALL SCREEN 100.

