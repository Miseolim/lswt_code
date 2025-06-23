``` abap
*&---------------------------------------------------------------------*
*& Include          ZHTML_EVENTF01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*& Form get_flight_info
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM get_flight_info .

  CLEAR : gt_sflight, gs_sflight.
  SELECT * INTO CORRESPONDING FIELDS OF TABLE gt_sflight
    FROM sflight.

  IF gt_sflight IS INITIAL.
    MESSAGE s037 DISPLAY LIKE 'E'.
    STOP.
  ENDIF.

ENDFORM.
*&---------------------------------------------------------------------*
*&      Module  USER_COMMAND_0100  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*& Form display_screen
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM display_screen .

  IF go_html_cont IS NOT BOUND.

    PERFORM create_object.

    event-eventid = cl_gui_html_viewer=>m_id_sapevent.
    event-appl_event = 'X'.
    APPEND event TO events.

    SET HANDLER lcl_event_handler=>sap_event FOR go_html_viewer.

    PERFORM handle_html_event.

    gs_layout = VALUE #( zebra      = abap_true
                         cwidth_opt = abap_true
                         sel_mode   = 'D' ).

    CALL METHOD go_alv_grid->set_table_for_first_display
      EXPORTING
        i_structure_name              = 'SFLIGHT'
        i_save                        = 'A'
        i_default                     = 'X'
        is_layout                     = gs_layout
      CHANGING
        it_outtab                     = gt_sflight
      EXCEPTIONS
        invalid_parameter_combination = 1
        program_error                 = 2
        too_many_lines                = 3
        OTHERS                        = 4.

  ELSE.
    CALL METHOD go_alv_grid->refresh_table_display.
  ENDIF.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form create_object
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM create_object .

  CREATE OBJECT go_html_cont
    EXPORTING
      repid     = sy-repid
      dynnr     = sy-dynnr
      side      = go_html_cont->dock_at_left
      extension = 300.

  CREATE OBJECT go_alv_cont
    EXPORTING
      repid     = sy-repid
      dynnr     = sy-dynnr
      side      = go_alv_cont->dock_at_right
      extension = 3000.

  CREATE OBJECT go_html_viewer
    EXPORTING
      parent = go_html_cont.

  CREATE OBJECT go_alv_grid
    EXPORTING
      i_parent = go_alv_cont.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form handle_sap_event
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM handle_sap_event USING pv_action.

  CLEAR gt_sflight.
  SELECT * INTO CORRESPONDING FIELDS OF TABLE gt_sflight
    FROM sflight
    WHERE connid EQ pv_action.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form handle_html_event
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM handle_html_event .

  CALL METHOD go_html_viewer->set_registered_events
    EXPORTING
      events = events.

  CALL METHOD go_html_viewer->load_html_document
    EXPORTING
      document_id          = 'ZSPFLI' " SMW0에 Upload한 HTML문서 ID
    IMPORTING
      assigned_url         = gv_url
    EXCEPTIONS
      document_not_found   = 1
      dp_error_general     = 2
      dp_invalid_parameter = 3.

  CALL METHOD go_html_viewer->show_data
    EXPORTING
      url                    = gv_url
    EXCEPTIONS
      cntl_error             = 1
      cnht_error_not_allowed = 2
      cnht_error_parameter   = 3
      dp_error_general       = 4
      OTHERS                 = 5.

  CALL METHOD cl_gui_cfw=>flush.

ENDFORM.
