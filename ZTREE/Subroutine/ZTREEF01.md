``` abap
*&---------------------------------------------------------------------*
*& Include          ZTREEF01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Form  fill_tree_info
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM fill_tree_info .

  SELECT DISTINCT carrid connid INTO TABLE gt_tr_sflight
    FROM sflight
    WHERE carrid IN so_car
      ORDER BY carrid connid ASCENDING.

ENDFORM.                    " fill_tree_info
*&---------------------------------------------------------------------*
*&      Form  fill_alv_info
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM fill_alv_info .

  SELECT * INTO TABLE gt_sflight
    FROM sflight
    WHERE carrid IN so_car.

ENDFORM.                    " fill_alv_info
*&---------------------------------------------------------------------*
*&      Form  build_node
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM build_node .

  DATA: node      TYPE mtreesnode,
        lv_carrid TYPE sflight-carrid,
        lv_text   TYPE scarr-carrname.

  node-node_key   = 'ROOT'.
  node-text       = 'Airline hierarchy'.
  node-isfolder   = 'X'.
*  node-n_image    = '@06@'.   " 접은 이미지
*  node-exp_image  = '@07@'.   " 펼친 이미지
  node-n_image    = '@04@'.   " 접은 이미지
  node-exp_image  = '@05@'.   " 펼친 이미지
  APPEND node TO node_table.
  CLEAR node.

  LOOP AT gt_tr_sflight INTO gs_tr_sflight .
*--------------------------------------------------------------------*
    ON CHANGE OF gs_tr_sflight-carrid.
      MOVE gs_tr_sflight-carrid TO lv_carrid.

      SELECT SINGLE carrname INTO lv_text
        FROM scarr
        WHERE carrid EQ lv_carrid.

      node-node_key  = gs_tr_sflight-carrid.
      node-relatkey  = 'ROOT'.
      node-isfolder  = 'X'.
*      node-n_image   = '@06@'.
*      node-exp_image = '@07@'.
      node-n_image   = '@04@'. " 접은 이미지
      node-exp_image = '@05@'. " 펼친 이미지
      node-text = lv_text.
      APPEND node TO node_table.
      CLEAR node.
    ENDON.
*--------------------------------------------------------------------*
    node-node_key = gs_tr_sflight-connid.
    node-text = gs_tr_sflight-connid.
    node-relatkey = gs_tr_sflight-carrid.
    node-isfolder = ' '.
    APPEND node TO node_table.
    CLEAR: node, gs_tr_sflight.

  ENDLOOP.
ENDFORM.                    " build_node
*&---------------------------------------------------------------------*
*&      Form  set_layout
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM set_layout .

  gs_layout = VALUE #( zebra      = abap_true
                       cwidth_opt = 'A'
                       sel_mode   = 'D' ).

ENDFORM.                    " set_layout
*&---------------------------------------------------------------------*
*&      Form  register_tree_event
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM register_tree_event .

  event-eventid = cl_gui_simple_tree=>eventid_node_double_click.
  event-appl_event = 'X'.
  APPEND event TO events.

  CALL METHOD go_tree->set_registered_events
    EXPORTING
      events                    = events
    EXCEPTIONS
      cntl_error                = 1
      cntl_system_error         = 2
      illegal_event_combination = 3
      OTHERS                    = 4.

  IF sy-subrc <> 0.
    MESSAGE ID sy-msgid TYPE sy-msgty NUMBER sy-msgno
               WITH sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4.
  ENDIF.

  SET HANDLER lcl_tree_event_handler=>handle_node_double_click FOR go_tree.

ENDFORM.                    " register_tree_event
*&---------------------------------------------------------------------*
*&      Form  search_clicked_node_info
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*      -->P_NODE_KEY  text
*----------------------------------------------------------------------*
FORM search_clicked_node_info  USING pv_node_key.

  CLEAR gt_sflight.

  SELECT * INTO TABLE gt_sflight
    FROM sflight
    WHERE carrid IN so_car
    AND   connid EQ pv_node_key.

ENDFORM.                    " search_clicked_node_info
*&---------------------------------------------------------------------*
*& Form display_screen
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM display_screen .

  IF go_left_cont IS NOT BOUND.

    PERFORM create_object.
    PERFORM register_tree_event.
    PERFORM build_node.

    CALL METHOD go_tree->add_nodes
      EXPORTING
        table_structure_name = 'MTREESNODE'
        node_table           = node_table.

    CALL METHOD go_tree->expand_node
      EXPORTING
        node_key = 'ROOT'.

    PERFORM set_layout.

    CALL METHOD go_alv_grid->set_table_for_first_display
      EXPORTING
        i_structure_name = 'SFLIGHT'
        i_save           = 'A'
        i_default        = 'X'
        is_layout        = gs_layout
      CHANGING
        it_outtab        = gt_sflight.

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

  CREATE OBJECT go_container
    EXPORTING
      repid     = sy-repid
      dynnr     = sy-dynnr
      side      = cl_gui_docking_container=>dock_at_left
      extension = 5000.

  CREATE OBJECT go_base_cont
    EXPORTING
      parent        = go_container
      orientation   = 1 " 0 : Vertical, 1 : Horizontal
      sash_position = 20
      with_border   = 1.

*-- Assign container
  go_left_cont   = go_base_cont->top_left_container.
  go_right_cont  = go_base_cont->bottom_right_container.

  CREATE OBJECT go_tree
    EXPORTING
      parent              = go_left_cont
      node_selection_mode = cl_gui_simple_tree=>node_sel_mode_single.

  CREATE OBJECT go_alv_grid
    EXPORTING
      i_parent = go_right_cont.

ENDFORM.
