``` abap
*&---------------------------------------------------------------------*
*& Include          ZR3_CHARTI01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Module  EXIT  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE exit INPUT.

  CALL METHOD : go_alv_grid->free,
                go_left_cont->free,
                go_right_cont->free,
                go_split_cont->free,
                go_container->free.

  FREE : go_alv_grid, go_left_cont, go_right_cont,
         go_split_cont, go_container.

  LEAVE TO SCREEN 0.

ENDMODULE.
*&---------------------------------------------------------------------*
*&      Module  USER_COMMAND_0100  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE user_command_0100 INPUT.

  CASE gv_okcode.
    WHEN 'SRCH'.
      PERFORM data_change.
  ENDCASE.

ENDMODULE.
