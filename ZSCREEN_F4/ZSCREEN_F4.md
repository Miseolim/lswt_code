``` abap
*&---------------------------------------------------------------------*
*& Report ZSCREEN_F4
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*

INCLUDE zscreen_f4top                           .    " Global Data

INCLUDE zscreen_f4s01                           .  " Selection screen
INCLUDE zscreen_f4f01                           .  " FORM-Routines


**********************************************************************
* INITIALIZATION
**********************************************************************
INITIALIZATION.
  PERFORM set_init_value.

**********************************************************************
* AT SELECTION-SCREEN
**********************************************************************
AT SELECTION-SCREEN ON VALUE-REQUEST FOR pa_ctry.
  PERFORM f4_country.

AT SELECTION-SCREEN ON VALUE-REQUEST FOR so_city-low.
  PERFORM f4_city USING 'LOW'.

AT SELECTION-SCREEN ON VALUE-REQUEST FOR so_city-high.
  PERFORM f4_city USING 'HIGH'.

**********************************************************************
* START-OF-SELECTION
**********************************************************************
START-OF-SELECTION.
