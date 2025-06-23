``` abap
*&---------------------------------------------------------------------*
*& Include          ZCL1_EXCEL_UPLOADF01
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

  p_bukrs = '0001'.
  p_gjahr = '2024'.

  SELECT SINGLE butxt INTO p_butxt
    FROM t001
    WHERE bukrs EQ p_bukrs.

*-- List box
  CLEAR gs_vrm_value.
  gs_vrm_value-key  = 'E'.
  gs_vrm_value-text = TEXT-l03. " Excel upload
  APPEND gs_vrm_value TO gs_vrm_posi.

  CLEAR gs_vrm_value.
  gs_vrm_value-key  = 'R'.
  gs_vrm_value-text = TEXT-l05. " Display data
  APPEND gs_vrm_value TO gs_vrm_posi.

  gs_vrm_name = 'P_GBN'.
  CALL FUNCTION 'VRM_SET_VALUES'
    EXPORTING
      id     = gs_vrm_name
      values = gs_vrm_posi[].

*-- For fuction key
  MOVE : 'Template'   TO w_functxt-quickinfo,
         icon_xls     TO w_functxt-icon_id,
         'Excel form' TO w_functxt-icon_text,
         w_functxt    TO sscrfields-functxt_01.

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
      WHEN 'BUK' OR 'GJR'.
        screen-input = 0.
        MODIFY SCREEN.
      WHEN 'BUT'.
        screen-intensified = 1.
        MODIFY SCREEN.
      WHEN 'PAT'.
        IF p_gbn EQ 'E'.
          screen-active = 1.
        ELSE.
          screen-active = 0.
        ENDIF.
        MODIFY SCREEN.
    ENDCASE.

  ENDLOOP.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form button_control
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM button_control .

  CASE sscrfields-ucomm.
    WHEN 'FC01'.
      PERFORM form_download.
  ENDCASE.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form form_download
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM form_download .
**********************************************************************
* Create excel upload application form
**********************************************************************
  DATA : lv_filename LIKE rlgrap-filename,
         lv_msg(100).

*-- Call windows browser
  CLEAR w_pfolder.
  PERFORM get_browser_info.

  IF w_pfolder IS INITIAL.
    EXIT.
  ENDIF.

  CLEAR lv_filename.
  CONCATENATE w_pfolder '\' 'Data_Upload' '.XLS' INTO lv_filename.

  PERFORM download_template USING lv_filename.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form f4_filename
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM f4_filename .
**********************************************************************
* Upload 할 파일을 찾는다
**********************************************************************
  DATA : lt_files  TYPE filetable,
         ls_files  LIKE LINE OF lt_files,
         lv_filter TYPE string,
         lv_path   TYPE string,
         lv_rc     TYPE i.

  CONCATENATE cl_gui_frontend_services=>filetype_excel
              'Excel 통합 문서(*.XLSX)|*.XLSX|'
              INTO lv_filter.

  CALL METHOD cl_gui_frontend_services=>file_open_dialog
    EXPORTING
      window_title            = 'File open'
      file_filter             = lv_filter
      initial_directory       = lv_path
    CHANGING
      file_table              = lt_files
      rc                      = lv_rc
    EXCEPTIONS
      file_open_dialog_failed = 1
      cntl_error              = 2
      error_no_gui            = 3
      not_supported_by_gui    = 4
      OTHERS                  = 5.

  CHECK sy-subrc EQ 0.
  ls_files = VALUE #( lt_files[ 1 ] OPTIONAL ).

  CLEAR p_path.
  p_path = ls_files-filename.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form get_browser_info
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM get_browser_info .

  IF w_pfolder IS NOT INITIAL.
    w_initialfolder = w_pfolder.
  ELSE.
    CALL METHOD cl_gui_frontend_services=>get_temp_directory
      CHANGING
        temp_dir = w_initialfolder.
  ENDIF.

  CALL METHOD cl_gui_frontend_services=>directory_browse
    EXPORTING
      window_title    = 'Download path'
      initial_folder  = w_initialfolder
    CHANGING
      selected_folder = w_pickedfolder.

  IF sy-subrc = 0.
    w_pfolder = w_pickedfolder.
  ELSE.
    MESSAGE i001 WITH TEXT-e05 DISPLAY LIKE 'E'.
    EXIT.
  ENDIF.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form download_template
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> LV_FILENAME
*&---------------------------------------------------------------------*
FORM download_template  USING pv_filename.

  DATA : wwwdata_item LIKE wwwdatatab,
         rc           TYPE i.

  gv_file = pv_filename.

  CALL FUNCTION 'WS_FILE_DELETE'
    EXPORTING
      file   = gv_file
    IMPORTING
      return = rc.

  SELECT SINGLE * FROM wwwdata
    INTO CORRESPONDING FIELDS OF wwwdata_item
   WHERE objid = 'ZEXCEL_UPLOAD'. " Form name

  CALL FUNCTION 'DOWNLOAD_WEB_OBJECT'
    EXPORTING
      key         = wwwdata_item
      destination = gv_file.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form excel_upload
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM excel_upload .

  TYPES: truxs_t_text_data(4096)   TYPE c OCCURS 0.

  DATA: lt_raw_data  TYPE truxs_t_text_data,
        lt_excel     LIKE TABLE OF alsmex_tabline WITH HEADER LINE,
        lv_index     LIKE sy-tabix,
        lv_waers     TYPE bkpf-waers,
        lv_dmbtr(20).

  FIELD-SYMBOLS:  <field>.

  IF p_path IS INITIAL.
    MESSAGE s001 WITH 'Upload 할 파일을 선택하세요' DISPLAY LIKE 'E'.
    STOP.
  ENDIF.

  CLEAR : gt_excel, gs_excel, lv_index.

  CALL FUNCTION 'ALSM_EXCEL_TO_INTERNAL_TABLE'
    EXPORTING
      filename                = p_path
      i_begin_col             = 1
      i_begin_row             = 2
      i_end_col               = 100
      i_end_row               = 50000
    TABLES
      intern                  = lt_excel
    EXCEPTIONS
      inconsistent_parameters = 1
      upload_ole              = 2
      OTHERS                  = 3.

  IF sy-subrc = 1.
    MESSAGE s001(k5) WITH TEXT-e01.
    STOP.
  ELSEIF sy-subrc <> 0.
    MESSAGE s001(k5) WITH TEXT-e02.
    STOP.
  ENDIF.

  CHECK NOT ( lt_excel[] IS INITIAL ).

  SORT lt_excel BY row col.

  LOOP AT lt_excel.

    lv_index = lt_excel-col.
    ASSIGN COMPONENT lv_index OF STRUCTURE gs_excel TO <field>.
    <field> = lt_excel-value.

    AT END OF row.
      APPEND gs_excel TO gt_excel.
      CLEAR gs_excel.
    ENDAT.

  ENDLOOP.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form make_body
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM make_body .

  CLEAR : gt_body, gs_body.
  LOOP AT gt_excel INTO gs_excel.

    gs_body = CORRESPONDING #( gs_excel ).

*-- Amount conversion
    _idoc_to_sap gs_body-waers gs_excel-dmbtr gs_body-dmbtr.

    APPEND gs_body TO gt_body.
    CLEAR gs_body.

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

  DATA : ls_variant TYPE disvariant,
         lv_cnt     TYPE i.

  ls_variant = VALUE #( report  = sy-repid
                        handle  = 'ZID' ).

  IF go_container IS NOT BOUND.

    CLEAR : gt_fcat, gs_fcat.
    PERFORM set_field_catalog USING : 'X' 'BUKRS' 'C' ' ',
                                      'X' 'BELNR' 'C' ' ',
                                      'X' 'GJAHR' 'C' ' ',
                                      ' ' 'BLART' 'C' ' ',
                                      ' ' 'BUDAT' 'C' ' ',
                                      ' ' 'BLDAT' 'C' ' ',
                                      ' ' 'BKTXT' ' ' 'X',
                                      ' ' 'WAERS' 'C' ' ',
                                      ' ' 'DMBTR' ' ' ' '.

    gs_layout = VALUE #( zebra      = abap_true
                         cwidth_opt = abap_true
                         sel_mode   = 'D' ).

    PERFORM create_object.

    SET HANDLER lcl_event_handler=>top_of_page  FOR go_alv_grid.

    CALL METHOD go_alv_grid->set_table_for_first_display
      EXPORTING
        i_save          = 'A'
        i_default       = 'X'
        is_layout       = gs_layout
        is_variant      = ls_variant
      CHANGING
        it_outtab       = gt_body
        it_fieldcatalog = gt_fcat.

    PERFORM register_event.

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
*&---------------------------------------------------------------------*
FORM set_field_catalog  USING pv_key pv_field pv_just pv_emphasize.

  gs_fcat = VALUE #( key        = pv_key
                     fieldname  = pv_field
                     ref_table  = 'BKPF'
                     just       = pv_just
                     emphasize  = pv_emphasize ).
  CASE pv_field.
    WHEN 'DMBTR'.
      gs_fcat-ref_table   = 'BSEG'.
      gs_fcat-cfieldname  = 'WAERS'.
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
      extension = 45.

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

*-- 회사코드
  SELECT SINGLE butxt INTO lv_text
    FROM t001
    WHERE bukrs EQ p_bukrs.

  PERFORM add_row USING lr_dd_table col_field col_value '회사코드' lv_text.

*-- 회계연도
  lv_text = p_gjahr && '년'.
  PERFORM add_row USING lr_dd_table col_field col_value '회계연도' lv_text.

  CASE p_gbn.
    WHEN 'E'.
      lv_text = 'Excel Upload'.
    WHEN 'R'.
      lv_text = 'Display data'.
  ENDCASE.

  PERFORM add_row USING lr_dd_table col_field col_value '작업구분' lv_text.

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
    MESSAGE s001 WITH TEXT-e03 DISPLAY LIKE 'E'.
  ENDIF.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form data_save
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM data_save .

  DATA : lv_answer.

  _popup_to_confirm TEXT-q01 lv_answer.

  IF lv_answer NE '1'.
    EXIT.
  ENDIF.

*-- Data save
  MODIFY ztbkpf FROM TABLE gt_body.
  COMMIT WORK.

  MESSAGE s102.

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

  CLEAR gt_body.
  SELECT * INTO CORRESPONDING FIELDS OF TABLE gt_body
    FROM ztbkpf
    WHERE bukrs EQ p_bukrs
      AND gjahr EQ p_gjahr.

  IF gt_body IS INITIAL.
    MESSAGE s037 DISPLAY LIKE 'E'.
    STOP.
  ENDIF.

ENDFORM.
