``` abap
*&---------------------------------------------------------------------*
*& Include ZR3_CHARTTOP                             - Report ZR3_CHART
*&---------------------------------------------------------------------*
REPORT zr3_chart MESSAGE-ID k5.

**********************************************************************
* TABLES
**********************************************************************
TABLES : scarr, spfli, sflight, sbook.

**********************************************************************
* Class instance
**********************************************************************
DATA : go_container  TYPE REF TO cl_gui_custom_container,
       go_split_cont TYPE REF TO cl_gui_splitter_container,
       go_left_cont  TYPE REF TO cl_gui_container,
       go_right_cont TYPE REF TO cl_gui_container,
       go_alv_grid   TYPE REF TO cl_gui_alv_grid.

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
* Internal table and Work area
**********************************************************************
DATA : BEGIN OF gs_body,
         carrid    TYPE sflight-carrid,
         planetype TYPE sflight-planetype,
         seatsmax  TYPE sflight-seatsmax,
         seatsocc  TYPE sflight-seatsocc,
       END OF gs_body,
       gt_body LIKE TABLE OF gs_body.

DATA : BEGIN OF gs_plane,
         planetype TYPE sflight-planetype,
       END OF gs_plane,
       gt_plane LIKE TABLE OF gs_plane.

*-- For ALV
DATA : gt_fcat   TYPE lvc_t_fcat,
       gs_fcat   TYPE lvc_s_fcat,
       gs_layout TYPE lvc_s_layo.

DATA : gs_variant TYPE disvariant,
       gs_stable  TYPE lvc_s_stbl.

*-- For Chart
DATA : gv_length  TYPE i,
       gv_xstring TYPE xstring.

**********************************************************************
* Common variable
**********************************************************************
DATA : gv_okcode TYPE sy-ucomm,
       gv_carrid TYPE scarr-carrid.
