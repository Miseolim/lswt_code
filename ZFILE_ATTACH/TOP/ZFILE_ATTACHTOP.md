``` abap
*&---------------------------------------------------------------------*
*& Include ZFILE_ATTACHTOP                          - Report ZFILE_ATTACH
*&---------------------------------------------------------------------*
REPORT zfile_attach MESSAGE-ID k5.

**********************************************************************
* TABLES
**********************************************************************
TABLES : ekko.

**********************************************************************
* Class instance
**********************************************************************
DATA : gcl_container TYPE REF TO cl_gui_docking_container,
       gcl_grid      TYPE REF TO cl_gui_alv_grid.

**********************************************************************
* Internal table and Work area
**********************************************************************
DATA : BEGIN OF gs_list,
         ebeln   TYPE ekko-ebeln,
         ebelp   TYPE ekpo-ebelp,
         bsart   TYPE ekko-bsart,
         bstyp   TYPE ekko-bstyp,
         bukrs   TYPE ekpo-bukrs,
         werks   TYPE ekpo-werks,
         lgort   TYPE ekpo-lgort,
         matnr   TYPE ekpo-matnr,
         menge   TYPE ekpo-menge,
         meins   TYPE ekpo-meins,
         attache TYPE icon_d,               "파일 업로드 유무
       END OF gs_list,

       gt_list LIKE TABLE OF gs_list.

*-- ALV 관련
DATA : gs_fcat    TYPE lvc_s_fcat,
       gt_fcat    TYPE lvc_t_fcat,
       gs_layout  TYPE lvc_s_layo,
       gs_stable  TYPE lvc_s_stbl,
       gs_variant TYPE disvariant.

**********************************************************************
* Common variable
**********************************************************************
CONSTANTS : gc_typeid TYPE srgbtbrel-typeid_a VALUE 'ZC101MM', "임의로 모듈별로 정한다
            gc_catid  TYPE srgbtbrel-catid_a  VALUE 'BO'.

DATA : gv_okcode TYPE sy-ucomm.
