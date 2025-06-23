``` abap
*&---------------------------------------------------------------------*
*& Report ZFILE_ATTACH
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*

INCLUDE zfile_attachtop                         .    " Global Data

INCLUDE zfile_attachs01                         .  " Selection screen
INCLUDE zfile_attachc01                         .  " ALV Events
INCLUDE zfile_attacho01                         .  " PBO-Modules
INCLUDE zfile_attachi01                         .  " PAI-Modules
INCLUDE zfile_attachf01                         .  " FORM-Routines


**********************************************************************
* START-OF-SELECTION
**********************************************************************
START-OF-SELECTION.
  PERFORM get_data.
  PERFORM check_attache.

  CALL SCREEN 100.
