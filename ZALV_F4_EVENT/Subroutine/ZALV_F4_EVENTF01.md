``` abap
*&---------------------------------------------------------------------*
*& Include          ZALV_F4_EVENTF01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*& Form get_base_data
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM get_base_data .

*-- Get source data
  CLEAR : gt_body.
  SELECT * INTO CORRESPONDING FIELDS OF TABLE gt_body
    FROM zictfi_ska1
    WHERE saknr IN so_saknr
      AND saknr NE space.

  IF gt_body IS INITIAL.
    MESSAGE s037 DISPLAY LIKE 'E'.
    STOP.
  ENDIF.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form get_search_help_data
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM get_search_help_data .

  CLEAR : gt_sh_saknr, gt_sh_koart.
  LOOP AT gt_body INTO gs_body.

*-- G/L Account
    gs_sh_saknr = VALUE #( saknr = gs_body-saknr
                           txt50 = gs_body-txt50 ).
    APPEND gs_sh_saknr TO gt_sh_saknr.

*-- Account type
    gs_sh_koart = VALUE #( koart  = gs_body-koart
                           ddtext = gs_body-koarttxt ).
    COLLECT gs_sh_koart INTO gt_sh_koart.

    CLEAR : gs_sh_saknr, gs_sh_koart.

  ENDLOOP.

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

  DATA : lv_cnt TYPE i.

  IF go_container IS NOT BOUND.

    CLEAR : gt_fcat, gs_fcat.
    PERFORM set_field_catalog USING : 'X' 'KTOPL' 'SKA1' 'C' ' ',
                                      'X' 'SAKNR' ''     'C' ' ',
                                      ' ' 'TXT50' 'SKAT' ' ' 'X',
                                      ' ' 'KOART' ''     'C' ' ',
                                      ' ' 'KOARTTXT' 'ZICTFI_SKA1' ' ' 'X'.
    gs_layout-zebra       = abap_true.
    gs_layout-cwidth_opt  = abap_true.
    gs_layout-sel_mode    = 'D'.

    gs_variant = VALUE #( report  = sy-repid
                          handle  = 'ZZ' ).

    PERFORM create_object.

    SET HANDLER : lcl_event_handler=>top_of_page        FOR go_alv_grid,
                  lcl_event_handler=>modify_value       FOR go_alv_grid,
                  lcl_event_handler=>handle_search_help FOR go_alv_grid.

    CALL METHOD go_alv_grid->set_table_for_first_display
      EXPORTING
        i_save          = 'A'
        i_default       = 'X'
        is_layout       = gs_layout
        is_variant      = gs_variant
      CHANGING
        it_outtab       = gt_body
        it_fieldcatalog = gt_fcat.

    PERFORM register_event.
    PERFORM register_f4_fields.

    lv_cnt = lines( gt_body ).
    MESSAGE s001 WITH lv_cnt '건이 조회되었습니다.'.

  ELSE.
    PERFORM refresh_table.
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
*&      --> P_
*&---------------------------------------------------------------------*
FORM set_field_catalog  USING pv_key pv_field pv_table pv_just pv_emph.

  gs_fcat = VALUE #( key        = pv_key
                     fieldname  = pv_field
                     ref_table  = pv_table
                     just       = pv_just
                     emphasize  = pv_emph ).

  CASE pv_field.
    WHEN 'SAKNR'.
      gs_fcat-coltext    = 'G/L Account'.
      gs_fcat-f4availabl = abap_true.
      gs_fcat-edit       = abap_true.
    WHEN 'KOART'.
      gs_fcat-coltext    = 'Account type'.
      gs_fcat-f4availabl = abap_true.
      gs_fcat-edit       = abap_true.
  ENDCASE.

  APPEND gs_fcat TO gt_fcat.
  CLEAR gs_fcat.

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

* Create TOP-Document
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

  lv_text = '1000'.
  PERFORM add_row USING lr_dd_table col_field col_value 'Chart of Account' lv_text.

*-- 회계연도
  lv_text = sy-datum(4) && '년'.
  PERFORM add_row USING lr_dd_table col_field col_value 'Fiscal year' lv_text.

  lv_text = 'F4 Search help test'.
  PERFORM add_row USING lr_dd_table col_field col_value 'Sample' lv_text.

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
*& Form onf4
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> E_FIELDNAME
*&      --> E_FIELDVALUE
*&      --> ES_ROW_NO
*&      --> ER_EVENT_DATA
*&      --> ET_BAD_CELLS
*&      --> E_DISPLAY
*&---------------------------------------------------------------------*
FORM onf4  USING p_fieldname   TYPE  lvc_fname
                 p_fieldvalue  TYPE  lvc_value
                 ps_row_no     TYPE  lvc_s_roid
                 pi_event_data TYPE REF TO cl_alv_event_data
                 pt_bad_cells  TYPE  lvc_t_modi
                 p_display     TYPE  char01.

  FIELD-SYMBOLS <fs_tab> TYPE STANDARD TABLE.

  DATA: dynprog          LIKE sy-repid,
        dynnr            LIKE sy-dynnr,
        window_title(30) TYPE c,
        l_row            TYPE p.

  DATA : ls_body  LIKE gs_body,
         lv_field TYPE dfies-fieldname,
         lv_text  TYPE help_info-dynprofld,
         lv_flag.

  READ TABLE gt_body INTO ls_body INDEX ps_row_no-row_id.

  DATA : lt_return LIKE TABLE OF ddshretval WITH HEADER LINE.
  CLEAR : dynprog, dynnr, window_title.", ls_itab.

  dynprog = sy-repid.
  dynnr   = sy-dynnr.

  CASE p_fieldname.
    WHEN 'SAKNR'.
      window_title = 'G/L Account'.
      lv_field     = 'SAKNR'.
      lv_text      = 'TXT50'.
      ASSIGN gt_sh_saknr TO <fs_tab>.
    WHEN 'KOART'.
      window_title = 'Account type'.
      lv_field     = 'KOART'.
      lv_text      = 'DDTEXT'.
      ASSIGN gt_sh_koart TO <fs_tab>.
  ENDCASE.

  CALL FUNCTION 'F4IF_INT_TABLE_VALUE_REQUEST'
    EXPORTING
      retfield        = lv_field " ALV 에 박히는 값
      dynpprog        = dynprog
      dynpnr          = dynnr
      dynprofield     = lv_text
      window_title    = window_title
      value_org       = 'S'
    TABLES
      value_tab       = <fs_tab>
      return_tab      = lt_return
    EXCEPTIONS
      parameter_error = 1
      no_values_found = 2
      OTHERS          = 3.

  pi_event_data->m_event_handled = 'X'.

  FIELD-SYMBOLS:  <fs> TYPE lvc_t_modi.

  DATA: ls_modi TYPE lvc_s_modi.

  ASSIGN pi_event_data->m_data->* TO <fs>.

  READ TABLE lt_return INDEX 1.
  IF sy-subrc = 0.
    ls_modi-row_id    = ps_row_no-row_id.
    ls_modi-fieldname = p_fieldname.
    ls_modi-value     = lt_return-fieldval.
    APPEND ls_modi TO <fs>.
  ENDIF.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form register_f4_fields
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM register_f4_fields .

  DATA: lt_f4 TYPE lvc_t_f4 WITH HEADER LINE.
  DATA: lt_f4_data TYPE lvc_s_f4.

  lt_f4_data-fieldname = 'SAKNR'.
  lt_f4_data-register = 'X' .
  lt_f4_data-getbefore = 'X' .
  lt_f4_data-chngeafter  ='X'.
  INSERT lt_f4_data INTO TABLE lt_f4.

  lt_f4_data-fieldname = 'KOART'.
  lt_f4_data-register = 'X' .
  lt_f4_data-getbefore = 'X' .
  lt_f4_data-chngeafter  ='X'.
  INSERT lt_f4_data INTO TABLE lt_f4.

  CALL METHOD go_alv_grid->register_f4_for_fields
    EXPORTING
      it_f4 = lt_f4[].

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

  DATA : ls_modi_cell TYPE lvc_s_modi,
         lv_field(10).

  CHECK pv_modified IS NOT INITIAL.

  ls_modi_cell = VALUE #( pt_good_cells[ 1 ] OPTIONAL ).

  CASE ls_modi_cell-fieldname.
    WHEN 'SAKNR'.
      gs_sh_saknr = VALUE #( gt_sh_saknr[ saknr = ls_modi_cell-value ] OPTIONAL ).
      gs_body-txt50 = gs_sh_saknr-txt50.
      lv_field = 'TXT50'.
    WHEN 'KOART'.
      gs_sh_koart = VALUE #( gt_sh_koart[ koart = ls_modi_cell-value ] OPTIONAL ).
      gs_body-koarttxt  = gs_sh_koart-ddtext.
      lv_field = 'KOARTTXT'.
  ENDCASE.

  MODIFY gt_body FROM gs_body INDEX ls_modi_cell-row_id
                              TRANSPORTING (lv_field).

  PERFORM refresh_table.



ENDFORM.
