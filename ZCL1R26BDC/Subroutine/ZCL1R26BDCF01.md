``` abap
*&---------------------------------------------------------------------*
*& Include          ZCL1R26BDCF01
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

  pa_bukrs = '0001'.
  pa_budat = sy-datum.
  pa_waers = 'KRW'.

*-- Set Company name
  SELECT SINGLE butxt INTO p_butxt
    FROM t001
    WHERE bukrs EQ pa_bukrs.

*-- Get G/L Account name
  SELECT saknr txt50
    INTO CORRESPONDING FIELDS OF TABLE gt_skat
    FROM skat
    WHERE ktopl EQ 'INT'
      AND spras EQ sy-langu.

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
    PERFORM set_field_catalog USING : 'X' 'STATUS'  ' '    'C' ' ',
                                      'X' 'BELNR'   'BKPF' 'C' ' ',
                                      ' ' 'BLART'   'BKPF' 'C' ' ',
                                      ' ' 'BKTXT'   'BKPF' ' ' 'X',
                                      ' ' 'SKONT'   ' '    'C' ' ',
                                      ' ' 'STEXT'   ' '    'C' 'X',
                                      ' ' 'HKONT'   ' '    'C' ' ',
                                      ' ' 'HTEXT'   ' '    'C' 'X',
                                      ' ' 'WRBTR'   'BSEG' ' ' ' ',
                                      ' ' 'WAERS'   'BKPF' 'C' ' '.
    PERFORM set_layout.
    PERFORM create_object.
    PERFORM exclude_button  USING gt_ui_functions.

    SET HANDLER : lcl_event_handler=>hotspot_click FOR go_alv_grid,
                  lcl_event_handler=>top_of_page   FOR go_alv_grid,
                  lcl_event_handler=>edit_toolbar  FOR go_alv_grid,
                  lcl_event_handler=>user_command  FOR go_alv_grid,
                  lcl_event_handler=>modify_value  FOR go_alv_grid.

    CALL METHOD go_alv_grid->set_table_for_first_display
      EXPORTING
        i_save               = 'A'
        i_default            = 'X'
        is_layout            = gs_layout
        is_variant           = gs_variant
        it_toolbar_excluding = gt_ui_functions
      CHANGING
        it_outtab            = gt_body
        it_fieldcatalog      = gt_fcat.

    PERFORM register_event.

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

  gs_fcat = VALUE #( key        = pv_key
                     fieldname  = pv_field
                     ref_table  = pv_table
                     just       = pv_just
                     emphasize  = pv_emph ).

  CASE pv_field.
    WHEN 'BELNR'.
      gs_fcat = VALUE #( BASE gs_fcat hotspot = abap_true ).
    WHEN 'STATUS'.
      gs_fcat = VALUE #( BASE gs_fcat coltext = 'Status' ).
    WHEN 'WRBTR'.
      gs_fcat = VALUE #( BASE gs_fcat cfieldname  = 'WAERS' ).
    WHEN 'SKONT'.
      gs_fcat = VALUE #( BASE gs_fcat coltext = 'Debit Account' ).
    WHEN 'HKONT'.
      gs_fcat = VALUE #( BASE gs_fcat coltext = 'Credit Account' ).
    WHEN 'STEXT' OR 'HTEXT'.
      gs_fcat = VALUE #( BASE gs_fcat coltext = 'Account name' ).
  ENDCASE.

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
                       sel_mode   = 'D'
                       stylefname = 'CELL_TAB' ).

  gs_variant  = VALUE #( report = sy-repid
                         handle = 'ALV1' ).

ENDFORM.
*&---------------------------------------------------------------------*
*& Form modify_screen
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM modify_screen .

  LOOP AT SCREEN.

    CASE screen-group1.
      WHEN 'BUK'.
        screen-input = 0.
      WHEN 'BUT'.
        screen-intensified = 1.
    ENDCASE.

    MODIFY SCREEN.

  ENDLOOP.

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

  CREATE OBJECT go_top_container
    EXPORTING
      repid     = sy-cprog
      dynnr     = sy-dynnr
      side      = go_top_container->dock_at_top
      extension = 46.

  CREATE OBJECT go_container
    EXPORTING
      repid     = sy-cprog
      dynnr     = sy-dynnr
      side      = go_container->dock_at_left
      extension = 3000.

  CREATE OBJECT go_alv_grid
    EXPORTING
      i_parent = go_container.

*-- Create TOP-Document
  CREATE OBJECT go_dyndoc_id
    EXPORTING
      style = 'ALV_GRID'.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form register_event
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM register_event .

  CALL METHOD go_dyndoc_id->initialize_document
    EXPORTING
      background_color = cl_dd_area=>col_textarea.

  CALL METHOD go_alv_grid->list_processing_events
    EXPORTING
      i_event_name = 'TOP_OF_PAGE'
      i_dyndoc_id  = go_dyndoc_id.

*-- Set display mode when program execute
  CALL METHOD go_alv_grid->set_ready_for_input
    EXPORTING
      i_ready_for_input = 0.

  CALL METHOD go_alv_grid->register_edit_event
    EXPORTING
      i_event_id = cl_gui_alv_grid=>mc_evt_modified.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form handle_hotstpo_click
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> E_ROW_ID
*&      --> E_COLUMN_ID
*&---------------------------------------------------------------------*
FORM handle_hotstpo_click  USING pv_row_id pv_column_id.

  CLEAR gs_body.
  gs_body = VALUE #( gt_body[ pv_row_id ] ).

  IF gs_body-belnr IS INITIAL.
    EXIT.
  ENDIF.

*-- Goto read document
  SET PARAMETER ID : 'BUK' FIELD pa_bukrs,
                     'GJR' FIELD pa_budat(4),
                     'BLN' FIELD gs_body-belnr.

  CALL TRANSACTION 'FB03' AND SKIP FIRST SCREEN.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form event_top_of_page
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM event_top_of_page .

  DATA : lr_dd_table TYPE REF TO cl_dd_table_element,
         col_field   TYPE REF TO cl_dd_area,
         col_value   TYPE REF TO cl_dd_area.

  DATA : lv_text TYPE sdydo_text_element.

  CALL METHOD go_dyndoc_id->add_table
    EXPORTING
      no_of_columns = 2
      border        = '0'
    IMPORTING
      table         = lr_dd_table.

*-- Set column
  CALL METHOD lr_dd_table->add_column
    IMPORTING
      column = col_field.

  CALL METHOD lr_dd_table->add_column
    IMPORTING
      column = col_value.

  lv_text = pa_bukrs && ' [' && p_butxt && ']'.
  PERFORM add_row USING lr_dd_table col_field col_value 'Company' lv_text.

*-- 회계연도
  lv_text = pa_budat(4) && '년'.
  PERFORM add_row USING lr_dd_table col_field col_value 'Fiscal year' lv_text.

  lv_text = pa_budat(4) && '.' && pa_budat+4(2) && '.' && pa_budat+6(2).
  PERFORM add_row USING lr_dd_table col_field col_value 'Posting date' lv_text.

  PERFORM set_top_of_page.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form add_row
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> LR_DD_TABLE
*&      --> COL_FIELD
*&      --> COL_VALUE
*&      --> P_
*&      --> LV_TEXT
*&---------------------------------------------------------------------*
FORM add_row  USING pr_dd_table  TYPE REF TO cl_dd_table_element
                    pv_col_field TYPE REF TO cl_dd_area
                    pv_col_value TYPE REF TO cl_dd_area
                    pv_field
                    pv_text.

  DATA : lv_text TYPE sdydo_text_element.

*-- Field.
  lv_text = pv_field.

  CALL METHOD pv_col_field->add_text
    EXPORTING
      text         = lv_text
      sap_emphasis = cl_dd_document=>strong
      sap_color    = cl_dd_document=>list_heading_inv.

  CALL METHOD pv_col_field->add_gap
    EXPORTING
      width = 3.

*-- Value.
  lv_text = pv_text.

  CALL METHOD pv_col_value->add_text
    EXPORTING
      text         = lv_text
      sap_emphasis = cl_dd_document=>heading
      sap_color    = cl_dd_document=>list_negative_inv.

  CALL METHOD pv_col_value->add_gap
    EXPORTING
      width = 3.

  CALL METHOD pr_dd_table->new_row.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form set_top_of_page
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM set_top_of_page .

* Creating html control
  IF go_html_cntrl IS INITIAL.
    CREATE OBJECT go_html_cntrl
      EXPORTING
        parent = go_top_container.
  ENDIF.

  CALL METHOD go_dyndoc_id->merge_document.
  go_dyndoc_id->html_control = go_html_cntrl.

* Display document
  CALL METHOD go_dyndoc_id->display_document
    EXPORTING
      reuse_control      = 'X'
      parent             = go_top_container
    EXCEPTIONS
      html_display_error = 1.

  IF sy-subrc NE 0.
    MESSAGE s001 WITH TEXT-e01 DISPLAY LIKE 'E'.
  ENDIF.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form add_toolbar
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> E_OBJECT
*&      --> E_INTERACTIVE
*&---------------------------------------------------------------------*
FORM add_toolbar  USING po_object TYPE REF TO cl_alv_event_toolbar_set
                        pv_interactive.

  DATA : lv_disabled.

  IF gv_mode EQ 'D'.
    lv_disabled = abap_true.
  ENDIF.

  CLEAR pv_interactive.

  _add_toolbar : '3' ''     ''                         ''                '' '',
                 ' ' 'TOGL' icon_toggle_display_change 'Read <-> Change' '' '',
                 '3' ''     ''                         ''                '' '',
                 ' ' 'IROW' icon_insert_row   'Create Rows'  ''    lv_disabled,
                 ' ' 'DROW' icon_delete_row   'Delete Rows'  ''    lv_disabled.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form handle_user_command
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> E_UCOMM
*&---------------------------------------------------------------------*
FORM handle_user_command  USING pv_ucomm.

  CASE pv_ucomm.
    WHEN 'TOGL'.
      PERFORM toggle_change.
    WHEN 'IROW'.
      PERFORM insert_row.
    WHEN 'DROW'.
      PERFORM delete_row.
  ENDCASE.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form toggle_change
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM toggle_change .

  CASE gv_mode.
    WHEN 'E'.
      gv_mode = 'D'.
      CALL METHOD go_alv_grid->set_ready_for_input
        EXPORTING
          i_ready_for_input = 0.
    WHEN 'D'.
      gv_mode = 'E'.
      CALL METHOD go_alv_grid->set_ready_for_input
        EXPORTING
          i_ready_for_input = 1.
  ENDCASE.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form insert_row
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM insert_row .

  DATA : ls_styl TYPE lvc_s_styl.

  CLEAR : gs_body, ls_styl.
  ls_styl-fieldname = 'STATUS'.
  ls_styl-style = cl_gui_alv_grid=>mc_style_disabled.
  INSERT ls_styl INTO TABLE gs_body-cell_tab.

  CLEAR ls_styl.
  ls_styl-fieldname = 'BELNR'.
  ls_styl-style = cl_gui_alv_grid=>mc_style_disabled.
  INSERT ls_styl INTO TABLE gs_body-cell_tab.

  CLEAR ls_styl.
  ls_styl-fieldname = 'STEXT'.
  ls_styl-style = cl_gui_alv_grid=>mc_style_disabled.
  INSERT ls_styl INTO TABLE gs_body-cell_tab.

  CLEAR ls_styl.
  ls_styl-fieldname = 'HTEXT'.
  ls_styl-style = cl_gui_alv_grid=>mc_style_disabled.
  INSERT ls_styl INTO TABLE gs_body-cell_tab.

  CLEAR ls_styl.
  ls_styl-style = cl_gui_alv_grid=>mc_style_enabled.
  INSERT ls_styl INTO TABLE gs_body-cell_tab.

*-- Set default value
  gs_body = VALUE #( BASE gs_body waers  = pa_waers ).

  APPEND gs_body TO gt_body.

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

  gs_stable = VALUE #( col = abap_true
                       row = abap_true ).

  CALL METHOD go_alv_grid->refresh_table_display
    EXPORTING
      is_stable = gs_stable.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form delete_row
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM delete_row .

  DATA : lt_roid TYPE lvc_t_roid,
         ls_roid TYPE lvc_s_roid.

  CALL METHOD go_alv_grid->get_selected_rows
    IMPORTING
      et_row_no = lt_roid.

  IF lt_roid IS INITIAL.
    MESSAGE s001 WITH TEXT-e02 DISPLAY LIKE 'E'.
    EXIT.
  ENDIF.

  SORT lt_roid BY row_id DESCENDING.
  LOOP AT lt_roid INTO ls_roid.

    DELETE gt_body INDEX ls_roid-row_id.

  ENDLOOP.

  PERFORM refresh_table.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form exclude_button
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> GT_UI_FUNCTIONS
*&---------------------------------------------------------------------*
FORM exclude_button  USING pt_ui_functions TYPE ui_functions.

  DATA : ls_ui_functions TYPE ui_func.

  CLEAR : pt_ui_functions.

  ls_ui_functions = cl_gui_alv_grid=>mc_fc_loc_undo.
  APPEND ls_ui_functions TO pt_ui_functions.
  ls_ui_functions = cl_gui_alv_grid=>mc_fc_loc_copy.
  APPEND ls_ui_functions TO pt_ui_functions.
  ls_ui_functions = cl_gui_alv_grid=>mc_fc_loc_copy_row.
  APPEND ls_ui_functions TO pt_ui_functions.
  ls_ui_functions = cl_gui_alv_grid=>mc_fc_loc_cut.
  APPEND ls_ui_functions TO pt_ui_functions.
  ls_ui_functions = cl_gui_alv_grid=>mc_fc_loc_delete_row.
  APPEND ls_ui_functions TO pt_ui_functions.
  ls_ui_functions = cl_gui_alv_grid=>mc_fc_loc_insert_row.
  APPEND ls_ui_functions TO pt_ui_functions.
  ls_ui_functions = cl_gui_alv_grid=>mc_fc_loc_append_row.
  APPEND ls_ui_functions TO pt_ui_functions.
  ls_ui_functions = cl_gui_alv_grid=>mc_fc_loc_paste.
  APPEND ls_ui_functions TO pt_ui_functions.
  ls_ui_functions = cl_gui_alv_grid=>mc_fc_loc_paste_new_row.
  APPEND ls_ui_functions TO pt_ui_functions.
  ls_ui_functions = cl_gui_alv_grid=>mc_fc_refresh.
  APPEND ls_ui_functions TO pt_ui_functions.
  ls_ui_functions = cl_gui_alv_grid=>mc_fc_auf.
  APPEND ls_ui_functions TO pt_ui_functions.
  ls_ui_functions = cl_gui_alv_grid=>mc_fc_average.
  APPEND ls_ui_functions TO pt_ui_functions.
  ls_ui_functions = cl_gui_alv_grid=>mc_fc_print.
  APPEND ls_ui_functions TO pt_ui_functions.
  ls_ui_functions = cl_gui_alv_grid=>mc_fc_graph.
  APPEND ls_ui_functions TO pt_ui_functions.
  ls_ui_functions = cl_gui_alv_grid=>mc_fc_check.
  APPEND ls_ui_functions TO pt_ui_functions.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form handle_modify_value
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> E_MODIFIED
*&      --> ET_GOOD_CELLS
*&---------------------------------------------------------------------*
FORM handle_modify_value  USING pv_modified
                                pt_good_cells TYPE lvc_t_modi.

  DATA : ls_modi_cell TYPE lvc_s_modi.

  CHECK pv_modified IS NOT INITIAL.

  ls_modi_cell = VALUE #( pt_good_cells[ 1 ] ).

  CASE ls_modi_cell-fieldname.
    WHEN 'SKONT'.
      CLEAR gs_skat.
      gs_skat = VALUE #( gt_skat[ saknr = ls_modi_cell-value ] OPTIONAL ).
      gs_body-stext = gs_skat-txt50.
    WHEN 'HKONT'.
      gs_skat = VALUE #( gt_skat[ saknr = ls_modi_cell-value ] OPTIONAL ).
      gs_body-htext = gs_skat-txt50.
    WHEN 'WAERS'.
      PERFORM refresh_table.
    WHEN OTHERS.
      EXIT.
  ENDCASE.

  MODIFY gt_body FROM gs_body INDEX ls_modi_cell-row_id
                              TRANSPORTING stext htext.

  PERFORM refresh_table.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form execute_posting
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM execute_posting .

  DATA : lt_roid       TYPE lvc_t_roid,
         ls_roid       TYPE lvc_s_roid,
         lv_answer,
         lv_amount(30).

*-- Get selected rows
  CALL METHOD go_alv_grid->get_selected_rows
    IMPORTING
      et_row_no = lt_roid.

  IF lt_roid IS INITIAL.
    MESSAGE s001 WITH TEXT-e03 DISPLAY LIKE 'E'.
    EXIT.
  ENDIF.

  _popup_confirm lv_answer.

  IF lv_answer NE '1'.
    EXIT.
  ENDIF.

*-- 선택한 행만큼 전표생성
  LOOP AT lt_roid INTO ls_roid.

    CLEAR gs_body.
    gs_body = VALUE #( gt_body[ ls_roid-row_id ] OPTIONAL ).

    WRITE gs_body-wrbtr CURRENCY pa_waers TO lv_amount. " Conversion amount
    CONDENSE lv_amount NO-GAPS.                         " 공백제거

    CLEAR : gt_bdcdata, gs_bdcdata.
    PERFORM dynpro  USING : 'X' 'SAPMF05A' '0100',
                            ' ' 'BDC_OKCODE' '/00',
                            ' ' 'BKPF-BLDAT' pa_budat,
                            ' ' 'BKPF-BLART' gs_body-blart,
                            ' ' 'BKPF-BUKRS' pa_bukrs,
                            ' ' 'BKPF-BUDAT' pa_budat,
                            ' ' 'BKPF-WAERS' pa_waers,
                            ' ' 'BKPF-BKTXT' gs_body-bktxt,
                            ' ' 'RF05A-NEWBS' '40',
                            ' ' 'RF05A-NEWKO' gs_body-skont,
                            'X' 'SAPMF05A' '0300',
                            ' ' 'BDC_OKCODE' '/00',
                            ' ' 'BSEG-WRBTR' lv_amount,
                            ' ' 'BSEG-SGTXT' 'Item 1',
                            ' ' 'RF05A-NEWBS' '50',
                            ' ' 'RF05A-NEWKO' gs_body-hkont,
                            'X' 'SAPLKACB' '0002',
                            ' ' 'BDC_OKCODE' '=ENTE',
                            'X' 'SAPMF05A' '0300',
                            ' ' 'BDC_OKCODE' '=BU',
                            ' ' 'BSEG-WRBTR' lv_amount,
                            ' ' 'BSEG-SGTXT' 'Item 2',
                            'X' 'SAPLKACB' '0002',
                            ' ' 'BDC_OKCODE' '=ENTE'.

    gs_params-dismode   = pa_mode.
    gs_params-updmode   = 'A'.        " Asynchronous
    gs_params-racommit  = abap_true.
    gs_params-nobinpt   = space.

    _init gt_messtab.
    CALL TRANSACTION 'FB01' USING gt_bdcdata
                            OPTIONS FROM gs_params
                            MESSAGES INTO gt_messtab.

    READ TABLE gt_messtab WITH KEY msgid  = 'F5'
                                   msgnr  = '312'.
    IF sy-subrc EQ 0.
*-- 전표생성 성공
      gs_body-status = icon_led_green.
      gs_body-belnr  = gt_messtab-msgv1.  " 생성된 전표번호

      MODIFY gt_body FROM gs_body INDEX ls_roid-row_id TRANSPORTING status belnr.
    ELSE.
*-- 전표생성 실패 (오류 메시지 추출)
      READ TABLE gt_messtab INDEX lines( gt_messtab[] ).
      _msg_build gt_messtab-msgid gt_messtab-msgnr gt_messtab-msgv1
                 gt_messtab-msgv2 gt_messtab-msgv3 gt_messtab-msgv4
                 gs_body-msg.

      gs_body-status = icon_led_red.
      MODIFY gt_body FROM gs_body INDEX ls_roid-row_id TRANSPORTING status msg.
    ENDIF.

  ENDLOOP.

  PERFORM refresh_table.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form dynpro
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> P_
*&      --> P_
*&      --> P_
*&---------------------------------------------------------------------*
FORM dynpro  USING pv_dynpro pv_fnam pv_fval.

  CASE pv_dynpro.
    WHEN 'X'.
      gs_bdcdata = VALUE #( dynbegin  = pv_dynpro
                            program   = pv_fnam
                            dynpro    = pv_fval ).
    WHEN OTHERS.
      gs_bdcdata = VALUE #( fnam  = pv_fnam
                            fval  = pv_fval ).
  ENDCASE.

  APPEND gs_bdcdata TO gt_bdcdata.
  CLEAR gs_bdcdata.

ENDFORM.

----------------------------------------------------------------------------------
Extracted by Direct Download Enterprise version 1.3.1 - E.G.Mellodew. 1998-2005 UK. Sap Release 758
