``` abap
*&---------------------------------------------------------------------*
*& Include          ZTREEI01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Module  EXIT  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE exit INPUT.

  CALL METHOD : go_alv_grid->free,
                go_tree->free,
                go_left_cont->free,
                go_right_cont->free.

  FREE : go_alv_grid, go_tree, go_left_cont, go_right_cont.

  LEAVE TO SCREEN 0.

ENDMODULE.
