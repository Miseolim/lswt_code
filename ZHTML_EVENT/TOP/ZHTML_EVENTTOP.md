``` abap
*&---------------------------------------------------------------------*
*& Include ZHTML_EVENTTOP                           - Report ZHTML_EVENT
*&---------------------------------------------------------------------*
REPORT ZHTML_EVENT MESSAGE-ID k5.


**********************************************************************
* Class instance
**********************************************************************
DATA: go_html_cont   TYPE REF TO cl_gui_docking_container,
      go_alv_cont    TYPE REF TO cl_gui_docking_container,
      go_html_viewer TYPE REF TO cl_gui_html_viewer,
      go_alv_grid    TYPE REF TO cl_gui_alv_grid.

**********************************************************************
* Internal table and Work area
**********************************************************************
DATA : gt_sflight TYPE TABLE OF sflight,
       gs_sflight TYPE sflight.

*-- For ALV
DATA : gt_fcat    TYPE lvc_t_fcat,
       gs_fcat    TYPE lvc_s_fcat,
       gs_layout  TYPE lvc_s_layo.

**********************************************************************
* Common variable
**********************************************************************
DATA: event  TYPE cntl_simple_event,
      events TYPE cntl_simple_events.

DATA : gv_okcode  TYPE sy-ucomm,
       gv_url(200).
