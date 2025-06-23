``` abap
*&---------------------------------------------------------------------*
*& Include          ZHTML_EVENTI01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Module  EXIT  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE exit INPUT.

  CALL METHOD : go_html_viewer->free,
                go_alv_grid->free,
                go_html_cont->free,
                go_alv_cont->free.

  FREE : go_html_viewer, go_alv_grid, go_html_cont, go_alv_cont.

  LEAVE TO SCREEN 0.

ENDMODULE.
