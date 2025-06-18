``` abap
*&---------------------------------------------------------------------*
*& Include ZALV_F4_EVENTTOP                         - Report ZALV_F4_EVENT
*&---------------------------------------------------------------------*
REPORT zalv_f4_event MESSAGE-ID k5.

**********************************************************************
* TABLES
**********************************************************************
TABLES zictfi_ska1.

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
DATA : gt_body TYPE TABLE OF zictfi_ska1,
       gs_body TYPE zictfi_ska1.

*-- F4 Search help
DATA : BEGIN OF gs_sh_saknr,
         saknr TYPE ska1-saknr,
         txt50 TYPE skat-txt50,
       END OF gs_sh_saknr,
       gt_sh_saknr LIKE TABLE OF gs_sh_saknr.

DATA : BEGIN OF gs_sh_koart,
         koart  TYPE bseg-koart,
         ddtext TYPE dd07t-ddtext,
       END OF gs_sh_koart,
       gt_sh_koart LIKE TABLE OF gs_sh_koart.

*-- For ALV
DATA : gt_fcat    TYPE lvc_t_fcat,
       gs_fcat    TYPE lvc_s_fcat,
       gs_layout  TYPE lvc_s_layo,
       gs_variant TYPE disvariant,
       gs_stable  TYPE lvc_s_stbl.

**********************************************************************
* Common variable
**********************************************************************
DATA : gv_okcode TYPE sy-ucomm.
