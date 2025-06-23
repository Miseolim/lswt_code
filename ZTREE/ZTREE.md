``` abap
*&---------------------------------------------------------------------*
*& Report ZTREE
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*

INCLUDE ztreetop                                .    " Global Data

INCLUDE ztrees01                                .  " Selection screen
INCLUDE ztreec01                                .  " ALV Events
INCLUDE ztreeo01                                .  " PBO-Modules
INCLUDE ztreei01                                .  " PAI-Modules
INCLUDE ztreef01                                .  " FORM-Routines


**********************************************************************
* START-OF-SELECTION.
**********************************************************************
START-OF-SELECTION.

  PERFORM fill_tree_info.
  PERFORM fill_alv_info.

  IF lines( gt_sflight ) GT 0.
    CALL SCREEN 100.
  ELSE.
    MESSAGE s037 DISPLAY LIKE 'E'.
  ENDIF.
