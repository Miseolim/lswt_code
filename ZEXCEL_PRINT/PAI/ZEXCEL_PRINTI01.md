``` abap
*&---------------------------------------------------------------------*
*& Include          ZEXCEL_PRINTI01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Module  USER_COMMAND_0100  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE user_command_0100 INPUT.

  CASE gv_okcode.
    WHEN 'EXCL'.
      PERFORM excel_job.
  ENDCASE.

ENDMODULE.
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

*-- 엑셀관련 오브젝트를 해제한다.
  FREE OBJECT : sheets,
                books,
                excel,
                sheet,
                book,
                workbook,
                cells,
                cell,
                row,
                buffer,
                activesheet.

  LEAVE TO SCREEN 0.

ENDMODULE.
