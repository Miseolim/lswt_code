``` abap
*&---------------------------------------------------------------------*
*& Include          ZFILE_ATTACHC01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*& Class lcl_event_handler
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
CLASS lcl_event_handler DEFINITION FINAL.
  PUBLIC SECTION.
    CLASS-METHODS : handle_hotspot FOR EVENT hotspot_click OF cl_gui_alv_grid
                                   IMPORTING e_row_id e_column_id.

ENDCLASS.
*&---------------------------------------------------------------------*
*& Class (Implementation) lcl_event_handler
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
CLASS lcl_event_handler IMPLEMENTATION.

  METHOD handle_hotspot.

    PERFORM handle_hotspot USING e_row_id e_column_id.

  ENDMETHOD.

ENDCLASS.
