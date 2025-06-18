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

*GUI Texts
*----------------------------------------------------------
* TITLE100 --> [FI] G/L Document posting

*Text elements
*----------------------------------------------------------
* E01 Top of page event error
* E02 삭제할 행을 선택하세요.
* E03 전기할 행을 선택하세요.
* I01 ※ 차변계정은 211000 (발생비용)
* I02 ※ 대변계정은 100000(소액현금)
* T01 Condition
* T02 Information


*Selection texts
*----------------------------------------------------------
* PA_BUDAT D       .
* PA_BUKRS D       .
* PA_MODE         BDC Mode
* PA_WAERS D       .


*Messages
*----------------------------------------------------------
*
* Message class: K5
*001   & & & &

----------------------------------------------------------------------------------
Extracted by Direct Download Enterprise version 1.3.1 - E.G.Mellodew. 1998-2005 UK. Sap Release 758
