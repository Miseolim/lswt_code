``` abap
*&---------------------------------------------------------------------*
*& Include ZTREETOP                                 - Report ZTREE
*&---------------------------------------------------------------------*
REPORT ztree MESSAGE-ID k5.

**********************************************************************
* TABLES
**********************************************************************
TABLES sflight.

**********************************************************************
* Declaration area for NODE
**********************************************************************
TYPES: node_table_type LIKE STANDARD TABLE OF mtreesnode
                       WITH DEFAULT KEY.
DATA: node_table TYPE node_table_type.

**********************************************************************
* Declaration area for GUI Class instance
**********************************************************************
DATA: go_container  TYPE REF TO cl_gui_docking_container,
      go_base_cont  TYPE REF TO cl_gui_easy_splitter_container,
      go_left_cont  TYPE REF TO cl_gui_container,
      go_right_cont TYPE REF TO cl_gui_container,
      go_alv_grid   TYPE REF TO cl_gui_alv_grid,
      go_tree       TYPE REF TO cl_gui_simple_tree.

**********************************************************************
* Declaration area for Tree event
**********************************************************************
DATA: events TYPE cntl_simple_events,
      event  TYPE cntl_simple_event.

**********************************************************************
* Declaration area for common variable
**********************************************************************
DATA: BEGIN OF gs_tr_sflight,
        carrid TYPE sflight-carrid,
        connid TYPE sflight-connid,
      END OF gs_tr_sflight.

DATA: gt_tr_sflight LIKE TABLE OF gs_tr_sflight.

DATA: BEGIN OF gs_sflight.
        INCLUDE STRUCTURE sflight.
DATA: END OF gs_sflight.

DATA: gt_sflight LIKE TABLE OF gs_sflight,
      gv_okcode  TYPE sy-ucomm,
      gs_layout  TYPE lvc_s_layo,
      gv_cprog   TYPE sy-cprog,
      gv_dynnr   TYPE sy-dynnr.
