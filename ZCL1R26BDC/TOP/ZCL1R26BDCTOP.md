``` abap
                 
*&---------------------------------------------------------------------*
*& Include          ZCL1R26BDCTOP
*&---------------------------------------------------------------------*
REPORT zcl1r26bdc MESSAGE-ID k5.


**********************************************************************
* Macro
**********************************************************************
DEFINE _add_toolbar.

  CLEAR gs_toolbar.
  MOVE : &1 TO gs_toolbar-butn_type,
         &2 TO gs_toolbar-function,
         &3 TO gs_toolbar-icon,
         &4 TO gs_toolbar-quickinfo,
         &5 TO gs_toolbar-text,
         &6 TO gs_toolbar-disabled.
  APPEND gs_toolbar TO po_object->mt_toolbar.

END-OF-DEFINITION.

DEFINE _popup_confirm.

  CALL FUNCTION 'POPUP_TO_CONFIRM'
    EXPORTING
      titlebar              = 'Data save'
      text_question         = 'Are you sure?'
      icon_button_1         = 'ICON_OKAY'
      icon_button_2         = 'ICON_CANCEL'
      default_button        = '1'
      display_cancel_button = 'X'
    IMPORTING
      answer                = &1
    EXCEPTIONS
      text_not_found        = 1
      OTHERS                = 2.

END-OF-DEFINITION.

DEFINE _msg_build.

  CALL FUNCTION 'MESSAGE_TEXT_BUILD'
    EXPORTING
      msgid               = &1
      msgnr               = &2
      msgv1               = &3
      msgv2               = &4
      msgv3               = &5
      msgv4               = &6
    IMPORTING
      message_text_output = &7.

END-OF-DEFINITION.

DEFINE _init.

  REFRESH &1.
  CLEAR &1.

END-OF-DEFINITION.

**********************************************************************
* TABLES
**********************************************************************
TABLES : bkpf, bseg.

**********************************************************************
* Class instance
**********************************************************************
DATA : go_container     TYPE REF TO cl_gui_docking_container,
       go_alv_grid      TYPE REF TO cl_gui_alv_grid,
*-- For Top-of-page -------------------------------------------------*
       go_top_container TYPE REF TO cl_gui_docking_container,
       go_dyndoc_id     TYPE REF TO cl_dd_document,
       go_html_cntrl    TYPE REF TO cl_gui_html_viewer.

**********************************************************************
* Internal table and Work area
**********************************************************************
DATA : BEGIN OF gs_body,
         status   TYPE icon-id,
         belnr    TYPE bkpf-belnr,
         blart    TYPE bkpf-blart,
         bktxt    TYPE bkpf-bktxt,
         skont    TYPE bseg-hkont,  " 차변계정
         stext    TYPE skat-txt50,
         hkont    TYPE bseg-hkont,  " 대변계정
         htext    TYPE skat-txt50,
         wrbtr    TYPE bseg-wrbtr,
         waers    TYPE bkpf-waers,
         msg(100),                 " Result message
         cell_tab TYPE lvc_t_styl,
       END OF gs_body,
       gt_body LIKE TABLE OF gs_body.

DATA : gt_skat TYPE TABLE OF skat,
       gs_skat TYPE skat.

*-- For ALV
DATA : gt_fcat   TYPE lvc_t_fcat,
       gs_fcat   TYPE lvc_s_fcat,
       gs_layout TYPE lvc_s_layo.

DATA : gs_variant TYPE disvariant.

*-- ALV Toolbar
DATA : gs_toolbar      TYPE stb_button,   " For ALV Toolbar button
       gt_ui_functions TYPE ui_functions, " Exclude ALV Standard button
       gs_stable       TYPE lvc_s_stbl.   " Stable when ALV refresh

*-- For BDC
DATA : gt_bdcdata TYPE TABLE OF bdcdata,
       gs_bdcdata TYPE bdcdata,
       gs_params  TYPE ctu_params.

DATA : gt_messtab TYPE TABLE OF bdcmsgcoll WITH HEADER LINE.

**********************************************************************
* Common variable
**********************************************************************
DATA : gv_okcode TYPE sy-ucomm,
       gv_mode   VALUE 'D'.

----------------------------------------------------------------------------------
Extracted by Direct Download Enterprise version 1.3.1 - E.G.Mellodew. 1998-2005 UK. Sap Release 758
