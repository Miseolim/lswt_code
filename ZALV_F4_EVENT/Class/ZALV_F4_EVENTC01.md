``` abap
*&---------------------------------------------------------------------*
*& Include          ZALV_F4_EVENTC01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*& Class LCL_EVENT_HANDLER
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
CLASS lcl_event_handler DEFINITION FINAL.

  PUBLIC SECTION.
    CLASS-METHODS : top_of_page    FOR EVENT top_of_page  OF cl_gui_alv_grid
                                   IMPORTING e_dyndoc_id,
                    modify_value   FOR EVENT data_changed_finished
                                   OF cl_gui_alv_grid
                                   IMPORTING e_modified et_good_cells,
                handle_search_help FOR EVENT onf4         OF cl_gui_alv_grid
                                   IMPORTING e_fieldname
                                             e_fieldvalue
                                             es_row_no
                                             er_event_data
                                             et_bad_cells
                                             e_display.

ENDCLASS.
*&---------------------------------------------------------------------*
*& Class (Implementation) LCL_EVENT_HANDLER
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
CLASS lcl_event_handler IMPLEMENTATION.

  METHOD top_of_page.
    PERFORM event_top_of_page.
  ENDMETHOD.                    "top_of_page

  METHOD modify_value.
    PERFORM handle_modify_value USING e_modified et_good_cells.
  ENDMETHOD.

  METHOD handle_search_help.
    PERFORM onf4 USING e_fieldname
                       e_fieldvalue
                       es_row_no
                       er_event_data
                       et_bad_cells
                       e_display.
  ENDMETHOD.                    "handle_search_help


ENDCLASS.

----------------------------------------------------------------------------------
Extracted by Direct Download Enterprise version 1.3.1 - E.G.Mellodew. 1998-2005 UK. Sap Release 758
