``` abap
*&---------------------------------------------------------------------*
*& Include          ZHTML_EVENTC01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*& Class LCL_EVENT_HANDLER
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
CLASS lcl_event_handler DEFINITION FINAL.

  PUBLIC SECTION.
    CLASS-METHODS : sap_event FOR EVENT sapevent OF cl_gui_html_viewer
                              IMPORTING action
                                        frame
                                        getdata
                                        postdata
                                        query_table.

ENDCLASS.
*&---------------------------------------------------------------------*
*& Class (Implementation) LCL_EVENT_HANDLER
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
CLASS lcl_event_handler IMPLEMENTATION.

  METHOD sap_event.

    PERFORM handle_sap_event USING action.

  ENDMETHOD.

ENDCLASS.
