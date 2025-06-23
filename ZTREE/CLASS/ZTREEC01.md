``` abap
*&---------------------------------------------------------------------*
*& Include          ZTREEC01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&       Class LCL_TREE_EVENT_HANDLER
*&---------------------------------------------------------------------*
*        Text
*----------------------------------------------------------------------*
CLASS lcl_tree_event_handler DEFINITION.

  PUBLIC SECTION.
    CLASS-METHODS: handle_node_double_click FOR EVENT node_double_click
                                            OF cl_gui_simple_tree
                                            IMPORTING node_key.

ENDCLASS.               "LCL_TREE_EVENT_HANDLER
*&---------------------------------------------------------------------*
*&       Class (Implementation)  lcl_tree_event_handler
*&---------------------------------------------------------------------*
*        Text
*----------------------------------------------------------------------*
CLASS lcl_tree_event_handler IMPLEMENTATION.

  METHOD handle_node_double_click.

    PERFORM search_clicked_node_info USING node_key.

    IF lines( gt_sflight ) GT 0.
      CALL METHOD go_alv_grid->refresh_table_display.
    ELSE.
      MESSAGE i001 WITH TEXT-e01.
    ENDIF.

  ENDMETHOD.                    "handle_node_double_click

ENDCLASS.               "lcl_tree_event_handler
