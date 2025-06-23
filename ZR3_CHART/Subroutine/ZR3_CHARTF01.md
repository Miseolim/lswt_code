``` abap
*&---------------------------------------------------------------------*
*& Include          ZR3_CHARTF01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*& Form set_init_value
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM set_init_value .

  pa_carr = 'LH'.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form get_base_data
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM get_base_data .

*-- Get data
  gv_carrid = pa_carr.

  CLEAR gt_body.
  SELECT carrid planetype
         SUM( seatsmax ) AS seatsmax
         SUM( seatsocc ) AS seatsocc
    INTO CORRESPONDING FIELDS OF TABLE gt_body
    FROM sflight
    WHERE carrid EQ pa_carr
    GROUP BY carrid planetype
    ORDER BY carrid planetype.

  IF gt_body IS INITIAL.
    MESSAGE s037 DISPLAY LIKE 'E'.
    STOP.
  ENDIF.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form display_screen
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM display_screen .

  IF go_container IS NOT BOUND.

    CLEAR : gt_fcat, gs_fcat.
    PERFORM set_field_catalog USING : 'X' 'CARRID'    'SCARR'   'C' ' ',
                                      ' ' 'PLANETYPE' 'SFLIGHT' 'C' ' ',
                                      ' ' 'SEATSMAX'  'SFLIGHT' ' ' 'X',
                                      ' ' 'SEATSOCC'  'SFLIGHT' ' ' ' '.

    PERFORM set_layout.
    PERFORM create_object.

    CALL METHOD go_alv_grid->set_table_for_first_display
      EXPORTING
        is_variant      = gs_variant
        i_save          = 'A'
        i_default       = 'X'
        is_layout       = gs_layout
      CHANGING
        it_outtab       = gt_body
        it_fieldcatalog = gt_fcat.

  ENDIF.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form set_field_catalog
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> P_
*&      --> P_
*&      --> P_
*&      --> P_
*&      --> P_
*&---------------------------------------------------------------------*
FORM set_field_catalog  USING pv_key pv_field pv_table pv_just pv_emph.

  gs_fcat = VALUE #(
                     key        = pv_key
                     fieldname  = pv_field
                     ref_table  = pv_table
                     just       = pv_just
                     emphasize  = pv_emph
                    ).

  APPEND gs_fcat TO gt_fcat.
  CLEAR gs_fcat.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form set_layout
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM set_layout .

  gs_layout = VALUE #( zebra      = abap_true
                       cwidth_opt = 'A'
                       sel_mode   = 'D' ).

  gs_variant  = VALUE #( report = sy-repid
                         handle = 'ALV1' ).

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
**********************************************************************
* Create Container
**********************************************************************
  CREATE OBJECT go_container
    EXPORTING
      container_name = 'MAIN_CONT'.

  CREATE OBJECT go_split_cont
    EXPORTING
      parent  = go_container
      rows    = 1
      columns = 2.

  CALL METHOD go_split_cont->get_container
    EXPORTING
      row       = 1
      column    = 1
    RECEIVING
      container = go_left_cont.

  CALL METHOD go_split_cont->get_container
    EXPORTING
      row       = 1
      column    = 2
    RECEIVING
      container = go_right_cont.

  CALL METHOD go_split_cont->set_column_width
    EXPORTING
      id    = 1
      width = 30.

**********************************************************************
* ALV & Chart
**********************************************************************
  CREATE OBJECT go_alv_grid
    EXPORTING
      i_parent = go_left_cont.

  CREATE OBJECT go_chart
    EXPORTING
      parent = go_right_cont.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form display_chart
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM display_chart .

  PERFORM set_chart_data.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form set_chart_data
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM set_chart_data .

  CLEAR : go_ixml,
          go_ixml_sf,
          go_ixml_docu,
          go_ixml_ostream,
          go_ixml_encoding,
          go_chartdata,
          go_categories,
          go_category,
          go_series,
          go_point,
          go_value.

  CLEAR : gv_xstring.

  go_ixml = cl_ixml=>create(  ).
  go_ixml_sf = go_ixml->create_stream_factory( ).

  go_ixml_docu = go_ixml->create_document( ).

  go_ixml_encoding = go_ixml->create_encoding(
                       byte_order    = if_ixml_encoding=>co_little_endian
                       character_set = 'utf-8' ).

  go_ixml_docu->set_encoding( encoding = go_ixml_encoding ).

*-- Now build a DOM, representing an XML document with chart data
  go_chartdata = go_ixml_docu->create_simple_element(
                                        name   = 'ChartData'
                                        parent = go_ixml_docu ).

*-- Categories (parent)
  go_categories = go_ixml_docu->create_simple_element(
                                        name   = 'Categories'
                                        parent = go_chartdata ).

  PERFORM set_category_value.
  PERFORM set_chart_value.
  PERFORM design_mode.

  go_chart->set_data( xdata = gv_xstring ).
  go_chart->render( ).


ENDFORM.
*&---------------------------------------------------------------------*
*& Form set_category_value
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM set_category_value .

  DATA : lv_value TYPE string.

*-- Categories (children)
  go_category = go_ixml_docu->create_simple_element(
                                        name   = 'Category'
                                        parent = go_categories ).
  go_category->if_ixml_node~set_value( 'Capacity' ). " SeatsMax


  go_category = go_ixml_docu->create_simple_element(
                                      name   = 'Category'
                                      parent = go_categories ).
  go_category->if_ixml_node~set_value( 'Occupied' ). " SeatsOcc

ENDFORM.
*&---------------------------------------------------------------------*
*& Form set_chart_value
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM set_chart_value .

  DATA : lv_value TYPE string.

  LOOP AT gt_body INTO gs_body.

    lv_value = gs_body-planetype.

*-- Build series (we need only 1)
    go_series = go_ixml_docu->create_simple_element(
                                                      name = 'Series'
                                                      parent = go_chartdata ).
    go_series->set_attribute( name  = 'label'
                              value = lv_value ). "'Series1' ).

*-- Seatsmax category
    lv_value = gs_body-seatsmax.

    go_point = go_ixml_docu->create_simple_element(
                                                    name = 'Point'
                                                    parent = go_series ).
    go_point->set_attribute( name  = 'label'
                             value = lv_value ).

    go_value = go_ixml_docu->create_simple_element(
                                                    name = 'Value'
                                                    parent = go_point ).

    go_value->if_ixml_node~set_value( lv_value ).

*-- Seatsocc category
    lv_value = gs_body-seatsocc.

    go_point = go_ixml_docu->create_simple_element(
                                                    name = 'Point'
                                                    parent = go_series ).
    go_point->set_attribute( name  = 'label'
                             value = lv_value ).

    go_value = go_ixml_docu->create_simple_element(
                                                    name = 'Value'
                                                    parent = go_point ).

    go_value->if_ixml_node~set_value( lv_value ).

  ENDLOOP.

*-- create ostream (into string variable) and render document into stream
  go_ixml_ostream = go_ixml_sf->create_ostream_xstring( gv_xstring ).
  go_ixml_docu->render( go_ixml_ostream ). "here f_xstring is filled

ENDFORM.
*&---------------------------------------------------------------------*
*& Form data_change
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM data_change .

  CLEAR gt_body.
  SELECT carrid planetype
         SUM( seatsmax ) AS seatsmax
         SUM( seatsocc ) AS seatsocc
    INTO CORRESPONDING FIELDS OF TABLE gt_body
    FROM sflight
    WHERE carrid EQ gv_carrid
    GROUP BY carrid planetype
    ORDER BY carrid planetype.

  IF gt_body IS INITIAL.
    MESSAGE s037 DISPLAY LIKE 'E'.
  ENDIF.

  PERFORM refresh_table.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form refresh_table
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM refresh_table .

  gs_stable = VALUE #( row  = abap_true
                       col  = abap_true ).

  CALL METHOD go_alv_grid->refresh_table_display
    EXPORTING
      is_stable = gs_stable.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form design_mode
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM design_mode .
**********************************************************************
* Chart type : Only Columns and Lines
**********************************************************************
  DATA: l_win_chart   TYPE REF TO cl_gui_chart_engine_win,
        g_design_mode.

  CATCH SYSTEM-EXCEPTIONS move_cast_error = 1.
    l_win_chart ?= go_chart->get_control( ).
  ENDCATCH.

  IF sy-subrc IS INITIAL.

    l_win_chart->set_design_mode( flag = g_design_mode event = 'X' ).
    l_win_chart->restrict_property_events( events = 'ChartType' ).

    CASE abap_true.
      WHEN p_column.
        l_win_chart->restrict_chart_types( charttypes = 'Columns' ).
      WHEN p_line.
        l_win_chart->restrict_chart_types( charttypes = 'Lines' ).
    ENDCASE.

  ENDIF.

ENDFORM.
