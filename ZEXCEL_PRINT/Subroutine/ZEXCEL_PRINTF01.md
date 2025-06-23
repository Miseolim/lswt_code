``` abap
*&---------------------------------------------------------------------*
*& Include          ZEXCEL_PRINTF01
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

  p_bukrs  = '0001'.

  SELECT SINGLE butxt INTO p_butxt
    FROM t001
    WHERE bukrs EQ p_bukrs.

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
        MODIFY SCREEN.
      WHEN 'BUT'.
        screen-intensified = 1.
        MODIFY SCREEN.
    ENDCASE.

  ENDLOOP.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form get_document_data
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM get_document_data .

*-- Get document header & line item
  CLEAR : gt_body, gs_body.
  SELECT a~bukrs a~belnr buzei bktxt waers budat bldat
         shkzg hkont txt50 wrbtr sgtxt
    INTO CORRESPONDING FIELDS OF TABLE gt_body
    FROM bkpf AS a INNER JOIN bseg AS b
      ON a~bukrs EQ b~bukrs
     AND a~belnr EQ b~belnr
     AND a~gjahr EQ b~gjahr
                   INNER JOIN skat AS c
      ON b~hkont EQ c~saknr
    WHERE a~bukrs EQ p_bukrs
      AND budat   IN so_budat
      AND ktopl   EQ 'INT'
      AND spras   EQ 'E'.

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

  DATA : ls_variant TYPE disvariant,
         lv_cnt     TYPE i.

  ls_variant = VALUE #( report  = sy-repid
                        handle  = 'ZID' ).

  IF go_container IS NOT BOUND.

    CLEAR : gt_fcat, gs_fcat.
    PERFORM set_field_catalog USING : 'X' 'BUKRS' 'BKPF' 'C' ' ',
                                      'X' 'BELNR' 'BKPF' 'C' ' ',
                                      'X' 'BUZEI' 'BSEG' 'C' ' ',
                                      ' ' 'BUDAT' 'BKPF' 'C' ' ',
                                      ' ' 'BLDAT' 'BKPF' 'C' ' ',
                                      ' ' 'BKTXT' 'BKPF' ' ' 'X',
                                      ' ' 'SHKZG' 'BSEG' 'C' ' ',
                                      ' ' 'HKONT' 'SKA1' 'C' ' ',
                                      ' ' 'TXT50' 'SKAT' ' ' 'X',
                                      ' ' 'WRBTR' 'BSEG' ' ' ' ',
                                      ' ' 'WAERS' 'BKPF' 'C' ' ',
                                      ' ' 'SGTXT' 'BSEG' ' ' 'X'.

    gs_layout = VALUE #( zebra      = abap_true
                         cwidth_opt = abap_true
                         sel_mode   = 'D' ).

    PERFORM create_object.

    SET HANDLER : lcl_event_handler=>hotspot_click FOR go_alv_grid,
                  lcl_event_handler=>top_of_page   FOR go_alv_grid.

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
*&      --> P_
*&---------------------------------------------------------------------*
FORM set_field_catalog  USING pv_key pv_field pv_table pv_just pv_emph.

  gs_fcat = VALUE #( key        = pv_key
                     fieldname  = pv_field
                     ref_table  = pv_table
                     just       = pv_just
                     emphasize  = pv_emph ).

  CASE pv_field.
    WHEN 'WRBTR'.
      gs_fcat-cfieldname = 'WAERS'.
    WHEN 'BELNR'.
      gs_fcat-hotspot = abap_true.
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
*& Form handle_hotspot_click
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> E_ROW_ID
*&      --> E_COLUMN_ID
*&---------------------------------------------------------------------*
FORM handle_hotspot_click  USING pv_row_id pv_column_id.

  gs_body = VALUE #( gt_body[ pv_row_id ] OPTIONAL ).

  SET PARAMETER ID : 'BUK' FIELD gs_body-bukrs,
                     'BLN' FIELD gs_body-belnr,
                     'GJR' FIELD gs_body-budat(4).

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

*-- 회사코드
  SELECT SINGLE butxt INTO lv_text
    FROM t001
    WHERE bukrs EQ p_bukrs.

  PERFORM add_row USING lr_dd_table col_field col_value '회사코드' lv_text.

*-- 회계연도
  lv_text = '2021' && '년'.
  PERFORM add_row USING lr_dd_table col_field col_value '회계연도' lv_text.

  lv_text = 'Data print to Excel'.
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
    MESSAGE s001 WITH TEXT-e01 DISPLAY LIKE 'E'.
  ENDIF.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form excel_job
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM excel_job .

  DATA : lt_roid   TYPE lvc_t_roid,
         ls_roid   TYPE lvc_s_roid,
         lv_line   TYPE i,
         lv_answer.

*-- 선택행 정보를 가져온다.
  CALL METHOD go_alv_grid->get_selected_rows
    IMPORTING
      et_row_no = lt_roid.

*-- 선택행 없으면 오류메시지
  IF lt_roid IS INITIAL.
    MESSAGE s001 WITH TEXT-e02 DISPLAY LIKE 'E'.
    EXIT.
  ENDIF.

*-- 1건만 선택 가능하도록 체크
  lv_line = lines( lt_roid ).

  IF lv_line GT 1.
    MESSAGE s001 WITH TEXT-e03 DISPLAY LIKE 'E'.
    EXIT.
  ENDIF.

*-- Dialog box
  _popup_to_confirm TEXT-q01 lv_answer.

  IF lv_answer NE '1'.
    EXIT.
  ENDIF.

*-- 선택행의 전표정보 발췌
  ls_roid = VALUE #( lt_roid[ 1 ] OPTIONAL ).
  gs_body = VALUE #( gt_body[ ls_roid-row_id ] OPTIONAL ).

  IF objfile IS INITIAL.
    TRY .
        CREATE OBJECT objfile.
    ENDTRY.
  ENDIF.

*-- Open file save window -------------------------------------------*
  IF pfolder IS NOT INITIAL.
    initialfolder = pfolder.
  ELSE.
    objfile->get_temp_directory( CHANGING temp_dir = initialfolder
                                 EXCEPTIONS cntl_error = 1
                                           error_no_gui = 2
                                           not_supported_by_gui = 3 ).
  ENDIF.

  objfile->directory_browse( EXPORTING initial_folder = initialfolder
                             CHANGING selected_folder = pickedfolder
                             EXCEPTIONS cntl_error = 1
                                        error_no_gui = 2
                                        not_supported_by_gui = 3 ).

  IF sy-subrc = 0.
    pfolder = pickedfolder.
  ELSE.
    MESSAGE 'An error has occured picking a folder' TYPE 'I' DISPLAY LIKE 'W'.
    EXIT.
  ENDIF.
*--------------------------------------------------------------------*
  IF pfolder IS INITIAL.
    EXIT.
  ENDIF.

  CALL METHOD objfile->free.
  FREE objfile.

  PERFORM download_excel.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form download_excel
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM download_excel .

  DATA : lv_rc  TYPE i.

*-- File name
  CLEAR gv_temp_filename.
  CONCATENATE pfolder '\' gs_body-belnr '.XLS'
              INTO gv_temp_filename.

  gv_form = 'ZEXCEL_PRINT'.
  PERFORM download_template   USING gv_form gv_temp_filename.
  PERFORM open_excel_template USING gv_form.
  PERFORM fill_excel_line.

*-- 기본적으로 Sheet 1을 보여주도록 셋팅
  CALL METHOD OF excel 'SHEETS' = sheet EXPORTING #1 = 1.
  CALL METHOD OF sheet 'SELECT' NO FLUSH.

*-- 모두 출력후 맨윗칸으로 커서 이동
  CALL METHOD OF excel 'Cells' = cell
    EXPORTING
      #1 = 1
      #2 = 1.

  CALL METHOD OF cell 'Select' .

  SET PROPERTY OF excel 'VISIBLE' = 1 . "엑셀 데이타를 다 뿌리고나서 보여줌

*-- Convert to PDF
  IF p_pdf EQ abap_true.
    PERFORM convert_to_pdf.
  ENDIF.

  MESSAGE s001 WITH TEXT-s01.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form download_template
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> GV_FORM
*&      --> GV_TEMP_FILENAME
*&---------------------------------------------------------------------*
FORM download_template  USING p_zform p_filename.

  DATA : wwwdata_item LIKE wwwdatatab,
         rc           TYPE i.

  gv_file = p_filename.

  CALL FUNCTION 'WS_FILE_DELETE'
    EXPORTING
      file   = gv_file
    IMPORTING
      return = rc.

  IF rc = 0 OR rc = 1.

  ELSE.
    MESSAGE e001  WITH '임시파일 초기화 실패.'
                       '이전에 Excel에서 자료를 OPEN하였는지 확인.'.
  ENDIF.

  SELECT SINGLE * FROM wwwdata
    INTO CORRESPONDING FIELDS OF wwwdata_item
   WHERE objid = p_zform.

  CALL FUNCTION 'DOWNLOAD_WEB_OBJECT'
    EXPORTING
      key         = wwwdata_item
      destination = gv_file.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form open_excel_template
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> GV_FORM
*&---------------------------------------------------------------------*
FORM open_excel_template  USING p_zform.

  IF excel IS INITIAL.
    CREATE OBJECT excel 'EXCEL.APPLICATION'.
  ENDIF.

  IF sy-subrc NE 0.
    MESSAGE i001 WITH sy-msgli.
  ENDIF.

  CALL METHOD OF excel 'WORKBOOKS' = workbook .
  SET PROPERTY OF excel 'VISIBLE' = 0 .

  CALL METHOD OF workbook 'OPEN' EXPORTING #1 = gv_file.

*-- Sheet에대한 설정을 할때 사용된다.






  GET PROPERTY OF : workbook    'Application' = application,
                    application 'ActiveSheet' = activesheet.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form fill_etc_excel_line
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM fill_excel_line .

  DATA : lv_belnr      TYPE bkpf-belnr,
         lv_line       TYPE i,
         lv_amount(20),
         lv_date(10).

**********************************************************************
* Write Document header
**********************************************************************
*-- 전기일자
  CLEAR lv_date.
  lv_date = gs_body-budat(4) && '.' && gs_body-budat+4(2) && '.' &&
            gs_body-budat+6(2).
  PERFORM fill_cells USING 8 3 lv_date.

*-- 증빙일자
  CLEAR lv_date.
  lv_date = gs_body-bldat(4) && '.' && gs_body-bldat+4(2) && '.' &&
            gs_body-bldat+6(2).
  PERFORM fill_cells USING 9 3 lv_date.

  PERFORM fill_cells USING 10  3 gs_body-bktxt.  " 전표 헤더텍스트
  PERFORM fill_cells USING 8  15 gs_body-bukrs.  " 회사코드
  PERFORM fill_cells USING 9  15 gs_body-waers.  " 통화키
  PERFORM fill_cells USING 10 15 gs_body-belnr.  " 전표번호

**********************************************************************
* Write Line item
**********************************************************************
  lv_belnr  = gs_body-belnr.
  lv_line   = 14.

  LOOP AT gt_body INTO gs_body WHERE belnr EQ lv_belnr.

    WRITE gs_body-wrbtr CURRENCY gs_body-waers TO lv_amount.

    PERFORM fill_cells USING lv_line  2 gs_body-buzei.  " 개별항목 번호
    PERFORM fill_cells USING lv_line  3 gs_body-shkzg.  " 차/대 지시자
    PERFORM fill_cells USING lv_line  4 gs_body-hkont.  " G/L Account
    PERFORM fill_cells USING lv_line  5 gs_body-txt50.  " G/L Account text
    PERFORM fill_cells USING lv_line  8 lv_amount.      " 전표통화금액
    PERFORM fill_cells USING lv_line 10 gs_body-sgtxt.  " Item text

    lv_line += 1.

  ENDLOOP.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form fill_cells
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> P_8
*&      --> P_3
*&      --> LV_DATE
*&---------------------------------------------------------------------*
FORM fill_cells  USING i j val.

  CALL METHOD OF excel 'CELLS' = cell
    EXPORTING
      #1 = i  " 행
      #2 = j. " 열

  SET PROPERTY OF cell 'VALUE' = val.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form convert_to_pdf
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM convert_to_pdf .

  DATA lv_rc TYPE i.

  CLEAR gv_temp_filename_pdf.
  CONCATENATE pfolder '\' gs_body-belnr '.PDF'
              INTO gv_temp_filename_pdf.

  GET PROPERTY OF excel 'Workbooks' = workbook
    EXPORTING #1 = 1.

  CALL METHOD OF workbook 'ExportAsFixedFormat'
    EXPORTING
      #1 = '0'
      #2 = gv_temp_filename_pdf.

  CALL METHOD OF workbook 'Close'
    EXPORTING
      #1 = 0.

  CALL METHOD OF excel 'Quit'.

  CALL METHOD cl_gui_frontend_services=>file_delete
    EXPORTING
      filename = CONV #( gv_temp_filename )
    CHANGING
      rc       = lv_rc.

  CALL METHOD OF workbook 'ExportAsFixedFormat'
    EXPORTING
      #1 = 0
      #2 = gv_temp_filename_pdf.

ENDFORM.
