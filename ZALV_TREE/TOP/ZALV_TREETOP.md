``` abap
*&---------------------------------------------------------------------*
*& Include ZALV_TREETOP                             - Report ZALV_TREE
*&---------------------------------------------------------------------*
REPORT zalv_tree MESSAGE-ID k5.

CLASS cl_gui_column_tree DEFINITION LOAD.
CLASS cl_gui_cfw DEFINITION LOAD.

**********************************************************************
* ICON
**********************************************************************
INCLUDE <icon>.

**********************************************************************
* TABLES
**********************************************************************
TABLES : zbut000.

**********************************************************************
* Class instance
**********************************************************************
DATA: go_tree        TYPE REF TO cl_gui_alv_tree,
      go_container   TYPE REF TO cl_gui_docking_container,
      go_change_menu TYPE REF TO cl_ctmenu.

**********************************************************************
* Internal table and Work area
**********************************************************************
DATA : gt_partner TYPE TABLE OF zbut000,
       gs_partner LIKE LINE OF gt_partner,
       gt_outtab  TYPE TABLE OF zbut000.

DATA: gs_hierhdr         TYPE treev_hhdr,
      gs_variant         TYPE disvariant,
      gt_list_commentary TYPE slis_t_listheader.

DATA: gt_events TYPE cntl_simple_events,
      gs_event  TYPE cntl_simple_event.

DATA : gt_fcat TYPE lvc_t_fcat,
       gs_fcat TYPE lvc_s_fcat.
**********************************************************************
* Common variable
**********************************************************************
DATA : ok_code TYPE sy-ucomm,
       save_ok TYPE sy-ucomm.

DATA : gv_logo TYPE sdydo_value.
