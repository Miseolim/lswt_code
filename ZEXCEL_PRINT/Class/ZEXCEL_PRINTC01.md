``` abap
*&---------------------------------------------------------------------*
*& Include          ZEXCEL_PRINTC01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*& Class LCL_EVENT_HANDLER
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
CLASS lcl_event_handler DEFINITION FINAL.

  PUBLIC SECTION.
    CLASS-METHODS : hotspot_click FOR EVENT hotspot_click OF cl_gui_alv_grid
                                  IMPORTING e_row_id e_column_id,
                    top_of_page   FOR EVENT top_of_page OF cl_gui_alv_grid
                                  IMPORTING e_dyndoc_id.

ENDCLASS.
*&---------------------------------------------------------------------*
*& Class (Implementation) LCL_EVENT_HANDLER
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
CLASS lcl_event_handler IMPLEMENTATION.

  METHOD hotspot_click.
    PERFORM handle_hotspot_click USING e_row_id e_column_id.
  ENDMETHOD.

  METHOD top_of_page.
    PERFORM event_top_of_page.
  ENDMETHOD.

ENDCLASS.
