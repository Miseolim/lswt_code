``` abap
*&---------------------------------------------------------------------*
*& Include          ZALV_F4_EVENTI01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Module  EXIT  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE exit INPUT.

  CALL METHOD : go_alv_grid->free,
                go_html_cntrl->free,
                go_container->free,
                go_top_container->free.

  FREE : go_alv_grid, go_html_cntrl,
         go_container, go_top_container.

  LEAVE TO SCREEN 0.

ENDMODULE.
