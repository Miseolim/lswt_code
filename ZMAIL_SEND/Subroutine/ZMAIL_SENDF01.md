``` abap
*&---------------------------------------------------------------------*
*& Include          ZMAIL_SENDF01
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

  SELECT SINGLE butxt INTO pa_butxt
    FROM t001
    WHERE bukrs EQ pa_bukrs.

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
*& Form get_base_data
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM get_base_data .

  CLEAR : gt_body, gs_body.
  SELECT * INTO CORRESPONDING FIELDS OF TABLE gt_body
    FROM ztbkpf
    WHERE bukrs EQ pa_bukrs
      AND budat IN so_budat.

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

  DATA : lv_cnt TYPE i.

  gs_variant = VALUE #( report  = sy-repid
                        handle  = 'ZZ' ).

  IF go_container IS NOT BOUND.

    CLEAR : gt_fcat, gs_fcat.
    PERFORM set_field_catalog USING : 'X' 'GJAHR' 'ZTBKPF' 'C' ' ',
                                      'X' 'BELNR' 'ZTBKPF' 'C' ' ',
                                      ' ' 'BLART' 'ZTBKPF' 'C' ' ',
                                      ' ' 'BUDAT' 'ZTBKPF' 'C' ' ',
                                      ' ' 'BKTXT' 'ZTBKPF' ' ' 'X',
                                      ' ' 'WAERS' 'ZTBKPF' 'C' ' ',
                                      ' ' 'DMBTR' 'ZTBKPF' ' ' ' '.

    gs_layout = VALUE #( zebra      = abap_true
                         cwidth_opt = abap_true
                         sel_mode   = 'D' ).

    PERFORM create_object.

    SET HANDLER : lcl_event_handler=>top_of_page   FOR go_alv_grid.

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
    WHEN 'DMBTR'.
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
      extension = 80.

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

  DATA : lv_text TYPE sdydo_text_element,
         lv_icon TYPE icon-id.

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
    WHERE bukrs EQ pa_bukrs.

  PERFORM add_row USING lr_dd_table col_field col_value '회사코드' lv_text.

*-- 회계연도
  lv_text = '2023' && '년'.
  PERFORM add_row USING lr_dd_table col_field col_value '회계연도' lv_text.

  lv_text = 'Send mail'.
  PERFORM add_row USING lr_dd_table col_field col_value '작업구분' lv_text.

*-- Icon
  lv_text = 'GREEN'.
  PERFORM add_row USING lr_dd_table col_field col_value '결재대기' lv_text.

  lv_text = 'YELLOW'.
  PERFORM add_row USING lr_dd_table col_field col_value '결재보류' lv_text.

  lv_text = 'RED'.
  PERFORM add_row USING lr_dd_table col_field col_value '결재반려' lv_text.

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

  DATA : lv_text TYPE sdydo_text_element,
         lv_icon TYPE iconname.

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
  CASE pv_text.
    WHEN 'GREEN'.
      CALL METHOD pv_col_value->add_icon
        EXPORTING
          sap_icon = 'ICON_LED_GREEN'
          sap_size = cl_dd_area=>large.
    WHEN 'RED'.
      CALL METHOD pv_col_value->add_icon
        EXPORTING
          sap_icon = 'ICON_LED_RED'.
    WHEN 'YELLOW'.
      CALL METHOD pv_col_value->add_icon
        EXPORTING
          sap_icon = 'ICON_LED_YELLOW'.
    WHEN OTHERS.
      lv_text = pv_text.
      CALL METHOD pv_col_value->add_text
        EXPORTING
          text         = lv_text
          sap_emphasis = cl_dd_document=>heading
          sap_color    = cl_dd_document=>list_negative_inv.
  ENDCASE.

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
*& Form mail_process
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM mail_process .

  DATA : lt_roid TYPE lvc_t_roid,
         ls_roid TYPE lvc_s_roid,
         lv_msg  TYPE string,
         lv_cnt  TYPE i.

  CALL METHOD go_alv_grid->get_selected_rows
    IMPORTING
      et_row_no = lt_roid.

  IF lt_roid IS INITIAL.
    MESSAGE s001 WITH TEXT-e02 DISPLAY LIKE 'E'.
    EXIT.
  ENDIF.

  lv_cnt = lines( lt_roid ).

  IF lv_cnt GT 1.
    MESSAGE s001 WITH TEXT-e03 DISPLAY LIKE 'E'.
    EXIT.
  ENDIF.

  ls_roid = VALUE #( lt_roid[ 1 ] OPTIONAL ).

  CLEAR gs_body.
  gs_body = VALUE #( gt_body[ ls_roid-row_id ] OPTIONAL ).

**********************************************************************
* Mail process
**********************************************************************
  PERFORM make_email_content.

*-- Call send mail function
  _init reclist.
  reclist-receiver = 'freeway77@daum.net'.
  reclist-rec_type = 'U'.
  APPEND reclist.

  CALL FUNCTION 'SO_NEW_DOCUMENT_ATT_SEND_API1'
    EXPORTING
      document_data              = doc_chng
      put_in_outbox              = 'X'
      commit_work                = 'X'
    TABLES
      packing_list               = objpack
      contents_bin               = objbin
      contents_txt               = objtxt
      receivers                  = reclist
    EXCEPTIONS
      too_many_receivers         = 1
      document_not_sent          = 2
      document_type_not_exist    = 3
      operation_no_authorization = 4
      parameter_error            = 5
      x_error                    = 6
      enqueue_error              = 7
      OTHERS                     = 8.

  IF sy-subrc NE 0.
    _msg_build sy-msgid sy-msgno sy-msgv1 sy-msgv2
               sy-msgv3 sy-msgv4 lv_msg.
    MESSAGE s001 WITH lv_msg DISPLAY LIKE 'E'.
  ELSE.
    MESSAGE s001 WITH TEXT-s01.
  ENDIF.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form make_email_content
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM make_email_content .
**********************************************************************
* Create mail content - HTML Format
**********************************************************************
  DATA : lv_email      TYPE pa0105-usrid,
         lv_str        TYPE string,
         lv_objbin     LIKE TABLE OF objbin WITH HEADER LINE,
         lv_amount(20),
         lv_date(10),
         lv_title(200).

*-- Begin of attachment
  DATA act_filetype     LIKE  rlgrap-filetype.
  DATA act_filename     LIKE  rlgrap-filename.
*-- End of attachment

  CLASS cl_abap_char_utilities DEFINITION LOAD.
  CONSTANTS : lv_vt   TYPE c VALUE cl_abap_char_utilities=>horizontal_tab,
              lv_cret TYPE c VALUE cl_abap_char_utilities=>cr_lf.

**********************************************************************
* 첨부파일
**********************************************************************
  DATA : file      LIKE rlgrap-filename,
         mail_file TYPE string.

  IF pa_attch IS NOT INITIAL.

    CALL FUNCTION 'UPLOAD'
      EXPORTING
        filetype     = 'BIN'
      IMPORTING
        act_filename = act_filename
        act_filetype = act_filetype
      TABLES
        data_tab     = objbin.

  ENDIF.

  _init : objtxt, objpack.
**********************************************************************
* Set mail title
**********************************************************************
  CLEAR : lv_date, lv_title.
  lv_date  = gs_body-budat(4) && '년' && gs_body-budat+4(2) && '월'.
  CONCATENATE lv_date ' ' '전표내역' INTO lv_title RESPECTING BLANKS.

  CLEAR : doc_chng, lv_str.
  CONCATENATE
    '<META http-equiv=Content-Type content="text/html; charset=euc-kr">' ''
    INTO lv_str SEPARATED BY lv_vt.
  CONCATENATE lv_cret lv_str lv_vt INTO lv_str.
  objtxt = lv_str.
  APPEND objtxt.

  CLEAR lv_str.
  CONCATENATE
    '<tbody>' ''
    INTO lv_str SEPARATED BY lv_vt.
  CONCATENATE lv_cret lv_str lv_vt INTO lv_str.
  objtxt = lv_str.
  APPEND objtxt.

  CLEAR lv_str.
  CONCATENATE
    '<center>' ''
    INTO lv_str SEPARATED BY lv_vt.
  CONCATENATE lv_cret lv_str lv_vt INTO lv_str.
  objtxt = lv_str.
  APPEND objtxt.

  CLEAR lv_str.
  CONCATENATE
    '<hr><h1>' lv_date '전표내역.</h1><hr><br><br>'
    INTO lv_str SEPARATED BY lv_vt.
  CONCATENATE lv_cret lv_str lv_vt INTO lv_str.
  objtxt = lv_str.
  APPEND objtxt.

**********************************************************************
* Mail contents
**********************************************************************
  CLEAR lv_str.
  CONCATENATE
    '<table cellpadding="0" cellspacing="0" bgcolor="royalblue">'
      '<tr>'
        '<td>'
          '<table cellpadding="0" cellspacing="1" width="800">'
    INTO lv_str SEPARATED BY lv_vt.
  CONCATENATE lv_cret lv_str lv_vt INTO lv_str.
  objtxt = lv_str.
  APPEND objtxt.

*-- 전표번호
  CLEAR lv_str.
  CONCATENATE
    '<tr>'
    '<td bgcolor="lightgray" valign="center" align="left" height="30" width="150">'
    '<font size="2"> <b>전표번호</b></font></td>'
    '<td bgcolor="white" valign="center" align="left" height="30"><font size="2">'
    ' ' gs_body-belnr '</font>'
    '</td>'
    '</tr>'
    INTO lv_str SEPARATED BY lv_vt.
  CONCATENATE lv_cret lv_str lv_vt INTO lv_str.
  objtxt = lv_str.
  APPEND objtxt.

*-- 전기일자
  CLEAR lv_str.
  CONCATENATE
    '<tr>'
    '<td bgcolor="lightgray" valign="center" align="left" height="30">'
    '<font size="2"> <b>전기일자</b></font></td>'
    '<td bgcolor="white" valign="center" align="left" height="30"><font size="2">'
    ' ' gs_body-budat '</font>'
    '</td>'
    '</tr>'
    INTO lv_str SEPARATED BY lv_vt.
  CONCATENATE lv_cret lv_str lv_vt INTO lv_str.
  objtxt = lv_str.
  APPEND objtxt.

*-- 전표유형
  CLEAR lv_str.
  CONCATENATE
    '<tr>'
    '<td bgcolor="lightgray" valign="center" align="left" height="30">'
    '<font size="2"> <b>전표유형</b></font></td>'
    '<td bgcolor="white" valign="center" align="left" height="30"><font size="2">'
    ' ' gs_body-blart '</font>'
    '</td>'
    '</tr>'
    INTO lv_str SEPARATED BY lv_vt.
  CONCATENATE lv_cret lv_str lv_vt INTO lv_str.
  objtxt = lv_str.
  APPEND objtxt.

*-- 전표헤더 텍스트
  CLEAR lv_str.
  CONCATENATE
    '<tr>'
    '<td bgcolor="lightgray" valign="center" align="left" height="30">'
    '<font size="2"> <b>전표헤더 텍스트</b></font></td>'
    '<td bgcolor="white" valign="center" align="left" height="30"><font size="2">'
    ' ' gs_body-bktxt '</font>'
    '</td>'
    '</tr>'
    INTO lv_str SEPARATED BY lv_vt.
  CONCATENATE lv_cret lv_str lv_vt INTO lv_str.
  objtxt = lv_str.
  APPEND objtxt.

*-- 통화키
  CLEAR lv_str.
  CONCATENATE
    '<tr>'
    '<td bgcolor="lightgray" valign="center" align="left" height="30">'
    '<font size="2"> <b>통화키</b></font></td>'
    '<td bgcolor="white" valign="center" align="left" height="30"><font size="2">'
    ' ' gs_body-waers '</font>'
    '</td>'
    '</tr>'
    INTO lv_str SEPARATED BY lv_vt.
  CONCATENATE lv_cret lv_str lv_vt INTO lv_str.
  objtxt = lv_str.
  APPEND objtxt.

*-- 전표통화금액
  CLEAR lv_amount.
  WRITE gs_body-dmbtr CURRENCY gs_body-waers TO lv_amount.

  CLEAR lv_str.
  CONCATENATE
    '<tr>'
    '<td bgcolor="lightgray" valign="center" align="left" height="30">'
    '<font size="2"> <b>전표통화금액</b></font></td>'
    '<td bgcolor="white" valign="center" align="left" height="30"><font size="2">'
    ' ' lv_amount '</font>'
    '</td>'
    '</tr>'
    INTO lv_str SEPARATED BY lv_vt.
  CONCATENATE lv_cret lv_str lv_vt INTO lv_str.
  objtxt = lv_str.
  APPEND objtxt.

*-- HTML Table 마무리
  CLEAR lv_str.
  CONCATENATE
    '</table>'
      '</td></tr>'
      '</table>' '<br>'
      '</center>'
      '</tbody>'
   INTO lv_str SEPARATED BY lv_vt.
  CONCATENATE lv_cret lv_str lv_vt INTO lv_str.
  objtxt = lv_str.
  APPEND objtxt.

*-- Content option
  CLEAR : tab_lines, lv_str.
  DESCRIBE TABLE objtxt LINES tab_lines.
  READ TABLE objtxt INDEX tab_lines.
  doc_chng-doc_size = ( tab_lines - 1 ) * 12255 + strlen( objtxt ).

*-- 메일 헤더, 본문 정보
  CLEAR objpack-transf_bin.
  objpack-head_start = 1.
  objpack-head_num = 0.
  objpack-body_start = 1.
  objpack-body_num = tab_lines.
  objpack-doc_type = 'HTM'.
  APPEND objpack.

  DATA: objhead LIKE solisti1 OCCURS 1 WITH HEADER LINE.
  objhead = act_filename.
  APPEND objhead.

  DATA f_lines TYPE i.
  DATA doc_size TYPE i.

  DATA : BEGIN OF lt_split OCCURS 0,
           content(50),
         END OF lt_split.

  CLEAR : tab_lines.
  DESCRIBE TABLE objbin LINES f_lines.
  doc_chng-doc_size = f_lines * 255.
  doc_chng-obj_name = 'Request mail'.
  doc_chng-obj_descr = lv_title.

  DATA: doc_type(3).

  IF pa_attch IS NOT INITIAL.

*-- Get file name
    SPLIT act_filename AT '\' INTO TABLE lt_split.
    READ TABLE lt_split INDEX lines( lt_split ).

*-- Get document type
    SPLIT act_filename AT '.' INTO act_filename doc_type.

*-- Attachment (pdf-Attachment)
    objpack-transf_bin = 'X'. " 이진형식의 전송을 위한 플래그
    objpack-head_start = 1.   " 본문의 시작
    objpack-head_num   = 0.   " 본문 헤더라인 수
    objpack-body_start = 1.   " 첨부파일 본문의 시작
    objpack-doc_size = doc_chng-doc_size.
    objpack-body_num = f_lines.         " 첨부 파일 본문의 길이
    objpack-doc_type = doc_type.        " 문서타입
    objpack-obj_name = doc_type && '_file'.  " 객체이름
    objpack-obj_descr = lt_split-content.    " 파일이름
    APPEND objpack.

  ENDIF.

ENDFORM.
