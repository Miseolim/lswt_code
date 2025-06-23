``` abap
*&---------------------------------------------------------------------*
*& Include          ZFILE_ATTACHF01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*& Form handle_hotspot
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> E_ROW_ID
*&      --> E_COLUMN_ID
*&---------------------------------------------------------------------*
FORM handle_hotspot  USING ps_row TYPE lvc_s_row
                           ps_col TYPE lvc_s_col.

  READ TABLE gt_list INTO gs_list INDEX ps_row-index.

  IF sy-subrc NE 0.
    EXIT.
  ENDIF.

  PERFORM call_gos USING gs_list-ebeln gs_list-ebelp.

  PERFORM get_data.
  PERFORM check_attache.
  PERFORM refresh_grid.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form get_data
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM get_data .
  CLEAR   : gs_list.
  REFRESH : gt_list.

  SELECT a~ebeln a~bsart a~bstyp
         b~ebelp b~bukrs b~werks b~lgort b~matnr b~menge b~meins
    INTO CORRESPONDING FIELDS OF TABLE gt_list
    FROM ekko AS a
   INNER JOIN ekpo AS b
      ON a~ebeln = b~ebeln
   WHERE a~ebeln IN so_ebeln.

  IF sy-subrc NE 0.
    MESSAGE s001 WITH TEXT-m01.
    LEAVE LIST-PROCESSING.
  ENDIF.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form set_fcat_layout
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM set_fcat_layout .

  gs_layout-zebra      = 'X'.
  gs_layout-cwidth_opt = 'A'.
  gs_layout-sel_mode   = 'D'.

  gs_stable-row = 'X'.
  gs_stable-col = 'X'.

  PERFORM make_fcat USING : 'X'   'EBELN'     ' '       'EKKO'  ' ',
                            'X'   'EBELP'     ' '       'EKPO'  ' ',
                            ' '   'BSART'     ' '       'EKKO'  ' ',
                            ' '   'BSTYP'     ' '       'EKKO'  ' ',
                            ' '   'BUKRS'     ' '       'EKPO'  ' ',
                            ' '   'WERKS'     ' '       'EKPO'  ' ',
                            ' '   'LGORT'     ' '       'EKPO'  ' ',
                            ' '   'MATNR'     ' '       'EKPO'  ' ',
                            ' '   'MENGE'     ' '       'EKPO'  ' ',
                            ' '   'MEINS'     ' '       'EKPO'  ' ',
                            ' '   'ATTACHE'   TEXT-f01  ' '     ' '.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form make_fcat
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> P_
*&      --> P_
*&      --> P_
*&      --> P_
*&      --> P_
*&---------------------------------------------------------------------*
FORM make_fcat  USING pv_key pv_field pv_text pv_ref_table pv_ref_field.

  gs_fcat-key       = pv_key.
  gs_fcat-fieldname = pv_field.
  gs_fcat-coltext   = pv_text.
  gs_fcat-ref_table = pv_ref_table.
  gs_fcat-ref_field = pv_ref_field.

  CASE pv_field.
    WHEN 'ATTACHE'.
      gs_fcat-icon    = 'X'.
      gs_fcat-hotspot = 'X'.

    WHEN 'MENGE'.
      gs_fcat-qfieldname = 'MEINS'.

  ENDCASE.

  APPEND gs_fcat TO gt_fcat.
  CLEAR  gs_fcat.

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

  CREATE OBJECT gcl_container
    EXPORTING
      side      = gcl_container->dock_at_left
      extension = 5000.

  CREATE OBJECT gcl_grid
    EXPORTING
      i_parent = gcl_container.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form call_gos
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> GS_LIST_EBELN
*&      --> GS_LIST_EBELP
*&---------------------------------------------------------------------*
FORM call_gos  USING pv_ebeln TYPE ekko-ebeln
                     pv_ebelp TYPE ekpo-ebelp.

  DATA : ls_object TYPE sibflporb,
         lv_mode.

  ls_object-typeid = gc_typeid. "모듈별조별 임의로 ID를 정한다
  ls_object-catid  = gc_catid.

* Unique하게 INSTID 부여해야 업로드한 파일을 찾을 수 있음
  CONCATENATE pv_ebeln pv_ebelp INTO ls_object-instid.

* MODE : 'C' Create
* MODE : 'D' Display
* Mode : 'E' Edit
  lv_mode = 'C'.

  CALL FUNCTION 'GOS_ATTACHMENT_LIST_POPUP'
    EXPORTING
      is_object = ls_object
      ip_mode   = lv_mode.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form check_attache
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM check_attache .

  DATA : ls_attach TYPE srgbtbrel,
         lt_attach LIKE TABLE OF ls_attach,
         lv_instid TYPE srgbtbrel-instid_a,
         lv_tabix  TYPE sy-tabix.

  CLEAR   : ls_attach, lv_tabix.
  REFRESH : lt_attach.

* 업로드한 Typid 로 기존에 업로드 되어 있는지 체크
  SELECT brelguid reltype  instid_a typeid_a catid_a instid_b typeid_b
         catid_b  logsys_a arch_a   logsys_b arch_b  utctime  homesys
    INTO CORRESPONDING FIELDS OF TABLE lt_attach
    FROM srgbtbrel
   WHERE typeid_a = gc_typeid
     AND catid_a  = gc_catid.

  IF sy-subrc NE 0.
    EXIT.
  ENDIF.

  SORT lt_attach BY instid_a.

  LOOP AT gt_list INTO gs_list.

    lv_tabix = sy-tabix.

    CONCATENATE gs_list-ebeln gs_list-ebelp INTO lv_instid.

* 업로드한 instid 로 파일이 존재 하는지 체크
    READ TABLE lt_attach INTO ls_attach WITH KEY instid_a = lv_instid
                                        BINARY SEARCH.

    IF sy-subrc EQ 0.

      gs_list-attache = icon_xls.

      MODIFY gt_list FROM gs_list INDEX lv_tabix TRANSPORTING attache.

    ENDIF.

  ENDLOOP.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form refresh_grid
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM refresh_grid .

  CALL METHOD gcl_grid->refresh_table_display
    EXPORTING
      i_soft_refresh = space
      is_stable      = gs_stable.

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

  IF gcl_container IS NOT BOUND.

    PERFORM set_fcat_layout.
    PERFORM create_object.

    SET HANDLER : lcl_event_handler=>handle_hotspot FOR gcl_grid.

    gs_variant-report = sy-repid.

    CALL METHOD gcl_grid->set_table_for_first_display
      EXPORTING
        i_save          = 'A'
        i_default       = 'X'
        is_layout       = gs_layout
        is_variant      = gs_variant
      CHANGING
        it_fieldcatalog = gt_fcat
        it_outtab       = gt_list.

  ENDIF.

ENDFORM.
