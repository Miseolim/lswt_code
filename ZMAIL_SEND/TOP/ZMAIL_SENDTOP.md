``` abap
*&---------------------------------------------------------------------*
*& Include ZMAIL_SENDTOP                            - Report ZMAIL_SEND
*&---------------------------------------------------------------------*
REPORT ZMAIL_SEND MESSAGE-ID k5.


**********************************************************************
* TABLES
**********************************************************************
TABLES : ztbkpf.

**********************************************************************
* Macro
**********************************************************************
DEFINE _init.

  REFRESH &1.
  CLEAR &1.

END-OF-DEFINITION.

DEFINE _popup_to_confirm.

  CALL FUNCTION 'POPUP_TO_CONFIRM'
    EXPORTING
      text_question         = &1
      text_button_1         = 'Yes'(001)
      text_button_2         = 'No'(002)
      display_cancel_button = 'X'
    IMPORTING
      answer                = &2
    EXCEPTIONS
      text_not_found        = 1
      OTHERS                = 2.

END-OF-DEFINITION.

DEFINE _range.

  &1 = VALUE #( sign    = &2
                option  = &3
                low     = &4
                high    = &5 ).

  APPEND &1.
  CLEAR &1.

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
DATA : gt_body TYPE TABLE OF ztbkpf,
       gs_body TYPE ztbkpf.

*---For mail send
DATA : doc_chng  LIKE sodocchgi1,
       tab_lines LIKE sy-tabix,
       objtxt    LIKE solisti1   OCCURS 0 WITH HEADER LINE,
       objpack   LIKE sopcklsti1 OCCURS 0 WITH HEADER LINE,
       objbin    LIKE solisti1   OCCURS 0 WITH HEADER LINE,
       reclist   LIKE somlreci1  OCCURS 0 WITH HEADER LINE.

*-- For ALV
DATA : gt_fcat    TYPE lvc_t_fcat,
       gs_fcat    TYPE lvc_s_fcat,
       gs_layout  TYPE lvc_s_layo,
       gs_stable  TYPE lvc_s_stbl,
       gs_variant TYPE disvariant.

**********************************************************************
* Common variable
**********************************************************************
DATA : gv_okcode            TYPE sy-ucomm,
       gv_temp_filename_pdf LIKE rlgrap-filename.
