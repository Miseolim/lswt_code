``` abap
*&---------------------------------------------------------------------*
*& Report ZC102PPR0008
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*

INCLUDE zc102ppr0008top_lsw                     .    " Global Data


INCLUDE zc102ppr0008s01_lsw                     .  " selection scree
INCLUDE zc102ppr0008c01_lsw                     .  " ALV Events
INCLUDE zc102ppr0008o01_lsw                     .  " PBO-Modules
INCLUDE zc102ppr0008i01_lsw                     .  " PAI-Modules
INCLUDE zc102ppr0008f01_lsw                     .  " FORM-Routines

**********************************************************************
*INITIALIZATION
**********************************************************************
INITIALIZATION.
  PERFORM set_value.

**********************************************************************
*AT SELECTION-SCREEN
**********************************************************************
AT SELECTION-SCREEN ON VALUE-REQUEST FOR so_werks-low.
  PERFORM f4_btype.

AT SELECTION-SCREEN ON VALUE-REQUEST FOR so_werks-high.
  PERFORM f4_btype .

**********************************************************************
*START-OF-SELECTION
**********************************************************************
START-OF-SELECTION.
  PERFORM get_ripening. "숙성창고 가져오기

  CALL SCREEN 100.
