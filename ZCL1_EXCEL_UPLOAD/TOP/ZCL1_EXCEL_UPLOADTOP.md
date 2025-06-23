``` abap
*&---------------------------------------------------------------------*
*& Include ZCL1_EXCEL_UPLOADTOP                     - Report ZCL1_EXCEL_UPLOAD
*&---------------------------------------------------------------------*
REPORT zcl1_excel_upload MESSAGE-ID k5.

**********************************************************************
* TYPE-POOLS
**********************************************************************
TYPE-POOLS vrm.

**********************************************************************
* TABLES
**********************************************************************
TABLES : bbkpf, bbseg, sscrfields.

**********************************************************************
* Macro
**********************************************************************
DEFINE _init.

  REFRESH &1.
  CLEAR &1.

END-OF-DEFINITION.

DEFINE _ranges.

  &1 = VALUE #( sign    = &2
                option  = &3
                low     = &4
                high    = &5 ).
  APPEND &1.

END-OF-DEFINITION.

DEFINE _date_interval.

  CALL FUNCTION 'RP_CALC_DATE_IN_INTERVAL'
    EXPORTING
      date      = &1
      days      = &2
      months    = &3
      signum    = &4
      years     = &5
    IMPORTING
      calc_date = &6.

END-OF-DEFINITION.

DEFINE _last_day.

  CALL FUNCTION 'LAST_DAY_OF_MONTHS'
    EXPORTING
      day_in            = &1
    IMPORTING
      last_day_of_month = &1.

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

DEFINE _alpha_input.

  CALL FUNCTION 'CONVERSION_EXIT_ALPHA_INPUT'
    EXPORTING
      input  = &1
    IMPORTING
      output = &1.

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

DEFINE _idoc_to_sap.

  CALL FUNCTION 'CURRENCY_AMOUNT_IDOC_TO_SAP'
    EXPORTING
      currency    = &1
      idoc_amount = &2
    IMPORTING
      sap_amount  = &3.

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
DATA : BEGIN OF gs_excel,
        bukrs TYPE bkpf-bukrs,
        gjahr TYPE bkpf-gjahr,
        belnr TYPE bkpf-belnr,
        blart TYPE bkpf-blart,
        budat TYPE bkpf-budat,
        bldat TYPE bkpf-bldat,
        bktxt TYPE bkpf-bktxt,
        waers TYPE bkpf-waers,
        dmbtr(20),
       END OF gs_excel.

DATA : gt_body  TYPE TABLE OF ztbkpf,
       gt_excel LIKE TABLE OF gs_excel,
       gs_body  TYPE ztbkpf.

*-- For ALV
DATA : gt_fcat   TYPE lvc_t_fcat,
       gs_fcat   TYPE lvc_s_fcat,
       gs_layout TYPE lvc_s_layo,
       gs_stable TYPE lvc_s_stbl.

*-- For select box in selection screen
DATA : gs_vrm_name  TYPE vrm_id,
       gs_vrm_posi  TYPE vrm_values,
       gs_vrm_value LIKE LINE OF gs_vrm_posi.

**********************************************************************
* Commmon variable
**********************************************************************
DATA : gv_okcode TYPE sy-ucomm,
       gv_tcode  TYPE sy-tcode,
       gv_file   LIKE rlgrap-filename.

*-- For file path
DATA : w_pickedfolder  TYPE string,
       w_initialfolder TYPE string,
       w_fullinfo      TYPE string,
       w_pfolder       TYPE rlgrap-filename. "MEMORY ID mfolder.

*-- For 1000 screen button
DATA : w_functxt TYPE smp_dyntxt,
       it_files  TYPE filetable,
       st_files  LIKE LINE OF it_files,
       w_rc      TYPE i,
       w_dir     TYPE string.
