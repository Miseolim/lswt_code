``` abap
*&---------------------------------------------------------------------*
*& Include ZC102PPR0008TOP                          - Report ZC102PPR0008
*&---------------------------------------------------------------------*
REPORT zc102ppr0008_lsw MESSAGE-ID zc102msg.

**********************************************************************
*TABLES
**********************************************************************
TABLES: zc102mmt0006.


**********************************************************************
* Declaration area for NODE
**********************************************************************
TYPES: node_table_type LIKE STANDARD TABLE OF mtreesnode
                       WITH DEFAULT KEY.
DATA: node_table TYPE node_table_type.

**********************************************************************
*call instance
**********************************************************************
DATA: go_container       TYPE REF TO cl_gui_docking_container,
      go_base_cont       TYPE REF TO cl_gui_easy_splitter_container,
      go_split_right     TYPE REF TO cl_gui_splitter_container,

      go_plant_top       TYPE REF TO cl_gui_container,
      go_plant_bottom    TYPE REF TO cl_gui_container,

      go_page_cont       TYPE REF TO cl_gui_docking_container,  " TOP OF PAGE 컨테이너
      go_html_cntrl      TYPE REF TO cl_gui_html_viewer,
      go_dyndoc_id       TYPE REF TO cl_dd_document,

*--plant 별 각각 다르게 담기 위함--*
      go_left_cont       TYPE REF TO cl_gui_container,
      go_plant_container TYPE REF TO cl_gui_container,

      go_plant_grid      TYPE REF TO cl_gui_alv_grid,
      go_chart_grid      TYPE REF TO cl_gui_alv_grid,
      go_tree            TYPE REF TO cl_gui_simple_tree.

*-- For Chart
DATA : go_chart    TYPE REF TO cl_gui_chart_engine.

DATA : go_ixml          TYPE REF TO if_ixml,
       go_ixml_sf       TYPE REF TO if_ixml_stream_factory,
       go_ixml_docu     TYPE REF TO if_ixml_document,
       go_ixml_ostream  TYPE REF TO if_ixml_ostream,
       go_ixml_encoding TYPE REF TO if_ixml_encoding.

DATA : go_chartdata  TYPE REF TO if_ixml_element,
       go_categories TYPE REF TO if_ixml_element,
       go_category   TYPE REF TO if_ixml_element,
       go_series     TYPE REF TO if_ixml_element,
       go_point      TYPE REF TO if_ixml_element,
       go_value      TYPE REF TO if_ixml_element.


**********************************************************************
* Declaration area for Tree event
**********************************************************************
DATA: events TYPE cntl_simple_events,
      event  TYPE cntl_simple_event.


**********************************************************************
*itab and work area
**********************************************************************

*--숙성창고 가져오기--*
DATA: gt_ripen TYPE TABLE OF zc102mmt0014_1,
      gs_ripen TYPE zc102mmt0014_1.

*--숙성창고 alv용 itab plant 1,2 ,3 ,4 순서대로--*
DATA: BEGIN OF gs_ripen1,
        stlno    TYPE zc102mmt0014_1-stlno,
        stltype  TYPE zc102mmt0014_1-stltype,
        werks    TYPE zc102mmt0014_1-werks,
        matnr    TYPE zc102mmt0014_1-matnr,
        datbi    TYPE zc102mmt0014_1-datbi,
        menge    TYPE zc102mmt0014_1-menge,
        meins    TYPE zc102mmt0014_1-meins,
        tempe    TYPE zc102mmt0014_1-tempe,
        t_unit   TYPE zc102mmt0014_1-t_unit,
        humid    TYPE zc102mmt0014_1-humid,
        h_unit   TYPE zc102mmt0014_1-h_unit,
        batno    TYPE zc102mmt0014_1-batno,
        dpose    TYPE zc102mmt0014_1-dpose,
        temptime TYPE zc102mmt0014_1-temptime,
        vbeln_so TYPE zc102sdt0006-vbeln_so,
      END OF gs_ripen1,
      gt_ripen_tree LIKE TABLE OF gs_ripen1,
      gt_ripen1     LIKE TABLE OF gs_ripen1.

*********차트용**********

*--월 가져오기--*
DATA: BEGIN OF gs_dispo_month,
        werks TYPE zc102_pp_ripen_disrate-werks, "플랜트
        perct TYPE zc102_pp_ripen_disrate-perct,
        td01  TYPE zc102_pp_ripen_disrate-td01,
        td02  TYPE zc102_pp_ripen_disrate-td02,
        td03  TYPE zc102_pp_ripen_disrate-td03,
        td04  TYPE zc102_pp_ripen_disrate-td04,
        td05  TYPE zc102_pp_ripen_disrate-td05,
        td06  TYPE zc102_pp_ripen_disrate-td06,
        td07  TYPE zc102_pp_ripen_disrate-td07,
        td08  TYPE zc102_pp_ripen_disrate-td08,
        td09  TYPE zc102_pp_ripen_disrate-td09,
        td10  TYPE zc102_pp_ripen_disrate-td10,
        td11  TYPE zc102_pp_ripen_disrate-td11,
        td12  TYPE zc102_pp_ripen_disrate-td12,
      END OF gs_dispo_month,
      gt_dispo_month LIKE TABLE OF gs_dispo_month.

*--폐기율 정보 가져올 곳--*
DATA: BEGIN OF gs_dispo_rate,
        stlno      TYPE zc102_rip_01-stlno,
        stltype    TYPE zc102_rip_01-stltype,
        werks      TYPE zc102_rip_01-werks,
        matnr      TYPE zc102_rip_01-matnr,
        datbi      TYPE zc102_rip_01-datbi,
        menge      TYPE zc102_rip_01-menge,
        meins      TYPE zc102_rip_01-meins,
        tempe      TYPE zc102_rip_01-tempe,
        t_unit     TYPE zc102_rip_01-tunit,
        humid      TYPE zc102_rip_01-humid,
        h_unit     TYPE zc102_rip_01-hunit,
        batno      TYPE zc102_rip_01-batno,
        dpose      TYPE zc102_rip_01-dpose,
        temptime   TYPE zc102_rip_01-temptime,
        prac_dispo TYPE zc102_rip_01-prac_dispo,
        m_dispo    TYPE zc102_rip_01-m_dispo,
      END OF gs_dispo_rate,
      gt_dispo_rate LIKE TABLE OF gs_dispo_rate.

*--for ALV
DATA: gt_plant_fcat   TYPE lvc_t_fcat,
      gs_plant_fcat   TYPE lvc_s_fcat,

      gt_chart_fcat   TYPE lvc_t_fcat,
      gs_chart_fcat   TYPE lvc_s_fcat,

      gs_plant_layout TYPE lvc_s_layo,

      gs_variant      TYPE disvariant.

*--toolbar
DATA: gt_ui_functions TYPE ui_functions,
      gs_button       TYPE stb_button.

**********************************************************************
*variant
**********************************************************************
DATA: gv_okcode     TYPE sy-ucomm,
      gv_mode       VALUE 'P',
      gv_enabled,
      gv_plant_name TYPE zc102mmt0014_1-werks.

*-- For Chart
DATA : gv_length  TYPE i,
       gv_xstring TYPE xstring.
