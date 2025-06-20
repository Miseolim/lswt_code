``` abap
*&---------------------------------------------------------------------*
*& Include          ZALV_TREEO01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*& Module STATUS_0100 OUTPUT
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
MODULE status_0100 OUTPUT.
 SET PF-STATUS 'MENU100'.
 SET TITLEBAR 'TITLE100'.
ENDMODULE.
*&---------------------------------------------------------------------*
*& Module INIT_PROCESS_CONTROL OUTPUT
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
MODULE init_process_control OUTPUT.

  IF go_tree IS NOT BOUND.

    PERFORM init_tree.
    PERFORM define_hierarchy_header CHANGING gs_hierhdr.
    PERFORM build_comment USING gt_list_commentary gv_logo.
    PERFORM define_field_catalog.
    PERFORM create_hierarchy.
    PERFORM fill_column_tree.

  ENDIF.

ENDMODULE.
