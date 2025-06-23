``` abap
*&---------------------------------------------------------------------*
*& Include          ZSCREEN_F4F01
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

*-- Get country value
  CLEAR : gt_country_value, gs_country_value.
  SELECT DISTINCT country
    INTO CORRESPONDING FIELDS OF TABLE gt_country_value
    FROM sgeocity.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form f4_country
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM f4_country .

  DATA : lt_return   LIKE TABLE OF ddshretval WITH HEADER LINE.

  _init lt_return.
  CALL FUNCTION 'F4IF_INT_TABLE_VALUE_REQUEST'
    EXPORTING
      retfield        = 'COUNTRY' " ALV 에 박히는 값
      dynpprog        = sy-repid
      dynpnr          = sy-dynnr
      dynprofield     = 'PA_CTRY'
      window_title    = 'Country'
      value_org       = 'S'
    TABLES
      value_tab       = gt_country_value
      return_tab      = lt_return
    EXCEPTIONS
      parameter_error = 1
      no_values_found = 2
      OTHERS          = 3.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form f4_city
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> SO_CITY_LOW
*&---------------------------------------------------------------------*
FORM f4_city  USING pv_gubun.
**********************************************************************
* PARAMETER PA_CTRY의 값에 따라 CITY의 SEARCH HELP 값을
* 유동적으로 가져오는 로직이다.
* 로직을 보고 꼭 필요한 부분만 발췌하도록 한다.
**********************************************************************
  DATA : lt_return LIKE TABLE OF ddshretval WITH HEADER LINE,
         lt_dynp   TYPE TABLE OF dynpread   WITH HEADER LINE,
         lv_field  TYPE help_info-dynprofld.

  lt_dynp-fieldname = 'PA_CTRY'.
  APPEND lt_dynp.

*-- Get parameter value
  CALL FUNCTION 'DYNP_VALUES_READ'
    EXPORTING
      dyname     = sy-cprog
      dynumb     = sy-dynnr
    TABLES
      dynpfields = lt_dynp.

  lt_dynp = VALUE #( lt_dynp[ fieldname = 'PA_CTRY' ] OPTIONAL ).

*-- Get select data by condition
  CLEAR gt_city_value.
  SELECT * INTO CORRESPONDING FIELDS OF TABLE gt_city_value
    FROM sgeocity
    WHERE country EQ lt_dynp-fieldvalue.

  CASE pv_gubun.
    WHEN 'LOW'.
      lv_field = 'SO_CITY-LOW'.
    WHEN 'HIGH'.
      lv_field = 'SO_CITY-HIGH'.
  ENDCASE.

  _init lt_return.
  CALL FUNCTION 'F4IF_INT_TABLE_VALUE_REQUEST'
    EXPORTING
      retfield        = 'CITY' " ALV 에 박히는 값
      dynpprog        = sy-repid
      dynpnr          = sy-dynnr
      dynprofield     = lv_field
      window_title    = 'Country'
      value_org       = 'S'
    TABLES
      value_tab       = gt_city_value
      return_tab      = lt_return
    EXCEPTIONS
      parameter_error = 1
      no_values_found = 2
      OTHERS          = 3.

ENDFORM.
