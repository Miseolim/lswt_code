``` abap
*&---------------------------------------------------------------------*
*& Include ZEXCEL_PRINTTOP                          - Report ZEXCEL_PRINT
*&---------------------------------------------------------------------*
REPORT zexcel_print MESSAGE-ID k5.

**********************************************************************
* TYPE-POOLS
**********************************************************************
TYPE-POOLS ole2 .

**********************************************************************
* Macro
**********************************************************************
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

**********************************************************************
* TABLES
**********************************************************************
TABLES : bkpf, bseg, skat.

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
         bukrs TYPE bkpf-bukrs,
         belnr TYPE bkpf-belnr,
         buzei TYPE bseg-buzei,
         bktxt TYPE bkpf-bktxt,
         waers TYPE bkpf-waers,
         budat TYPE bkpf-budat,
         bldat TYPE bkpf-bldat,
         shkzg TYPE bseg-shkzg,
         hkont TYPE bseg-hkont,
         txt50 TYPE skat-txt50,
         wrbtr TYPE bseg-wrbtr,
         sgtxt TYPE bseg-sgtxt,
       END OF gs_body.

DATA : gt_body LIKE TABLE OF gs_body.

*-- For ALV
DATA : gt_fcat   TYPE lvc_t_fcat,
       gs_fcat   TYPE lvc_s_fcat,
       gs_layout TYPE lvc_s_layo,
       gs_stable TYPE lvc_s_stbl.

**********************************************************************
* Common variable
**********************************************************************
DATA : gv_okcode  TYPE sy-ucomm.

*-- For file browser
DATA : objfile       TYPE REF TO cl_gui_frontend_services,
       pickedfolder  TYPE string,
       initialfolder TYPE string,
       fullinfo      TYPE string,
       pfolder       TYPE rlgrap-filename. "MEMORY ID mfolder.

*-- For Excel
DATA: gv_tot_page   LIKE sy-pagno,          " Total page
      gv_percent(3) TYPE n,                 " Reading percent
      gv_file       LIKE rlgrap-filename .  " File name

DATA : gv_temp_filename     LIKE rlgrap-filename,
       gv_temp_filename_pdf LIKE rlgrap-filename,
       gv_form(40).

DATA: excel       TYPE ole2_object,
      workbook    TYPE ole2_object,
      books       TYPE ole2_object,
      book        TYPE ole2_object,
      sheets      TYPE ole2_object,
      sheet       TYPE ole2_object,
      activesheet TYPE ole2_object,
      application TYPE ole2_object,
      pagesetup   TYPE ole2_object,
      cells       TYPE ole2_object,
      cell        TYPE ole2_object,
      row         TYPE ole2_object,
      buffer      TYPE ole2_object,
      font        TYPE ole2_object,
      range       TYPE ole2_object,  " Range
      borders     TYPE ole2_object.

DATA: cell1 TYPE ole2_object,
      cell2 TYPE ole2_object.
