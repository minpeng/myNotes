# java 导出excel

## maven 所需要的包

```
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>3.15</version>
        </dependency>
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>3.15</version>
        </dependency>
```


## 代码

```
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.util.List;

import javax.servlet.http.HttpServletResponse;

import org.apache.poi.xssf.usermodel.XSSFCell;
import org.apache.poi.xssf.usermodel.XSSFRow;
import org.apache.poi.xssf.usermodel.XSSFSheet;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

public class ExportExcelUtil {

    public static void exportErrorStack( HttpServletResponse response, String[] titles, String fileName,
            List<Object> list )
            throws IOException {
        // 创建excel工作薄
        XSSFWorkbook wb = new XSSFWorkbook();
        XSSFSheet sheet = wb.createSheet( "第一页" );
        // 在sheet里创建一行，参数为行号，第一行
        XSSFRow row = sheet.createRow( (short)0 );

        // 设置列宽
        for( int i = 0; i < titles.length; i++ ) {
            sheet.setColumnWidth( (short)i, (short)(30 * 256) );
        }
        // 设置标题
        for( int i = 0; i < titles.length; i++ ) {
            XSSFCell cell = row.createCell( (short)i );
            cell.setCellValue( titles[i] );
        }
        //填充内容
        for(int i = 0; i < errorInfoList.size(); i++ ){
            XSSFRow contentrow = sheet.createRow( (i + 1) );

            contentrow.createCell( 0 ).setCellValue( "第1列的值");
            contentrow.createCell( 1 ).setCellValue( "第2列的值");
            contentrow.createCell( 2 ).setCellValue( "第3列的值" );
            contentrow.createCell( 3 ).setCellValue( "第4列的值");
            contentrow.createCell( 4 ).setCellValue( "第5列的值" );


        }

        OutputStream out = response.getOutputStream();
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        wb.write( byteArrayOutputStream );
        
        String header = request.getHeader( "User-Agent" ).toUpperCase();
        if( header.contains( "MSIE" ) || header.contains( "TRIDENT" ) || header.contains( "EDGE" ) ) {
            fileName = URLEncoder.encode( fileName, "utf-8" );
            fileName = fileName.replace( "+", "%20" ); // IE下载文件名空格变+号问题
        }
        else {
            fileName = new String( fileName.getBytes(), "ISO8859-1" );
        }
        response.setContentType( "application/binary;charset=utf-8" );
        response.setHeader( "Content-Length", "" + byteArrayOutputStream.size() );
        response.setHeader( "Content-disposition",
            "attachment; filename=" + new String( fileName.getBytes(), "ISO8859-1" ) + ".xlsx" );// 组装附件名称和格式

        wb.write( out );
        out.flush();
        out.close();
    }

}

```


```
//强制函数生效
sheetOne.setForceFormulaRecalculation(true);
```

```
//合并单元格
private static void mergedRegion( XSSFSheet sheet ) {
        for (int i = sheet.getNumMergedRegions() - 1; i >= 0; i--) {
            CellRangeAddress region = sheet.getMergedRegion(i);
            Row firstRow = sheet.getRow(region.getFirstRow());
            Cell firstCellOfFirstRow = firstRow.getCell(region.getFirstColumn());
            String value="";
            if (firstCellOfFirstRow.getCellType() == Cell.CELL_TYPE_STRING) {
                value = firstCellOfFirstRow.getStringCellValue();
            }
            sheet.removeMergedRegion(i);

            for (Row row : sheet) {
                for (Cell cell : row) {
                    if (region.isInRange(cell.getRowIndex(), cell.getColumnIndex())) {
                        cell.setCellType(Cell.CELL_TYPE_STRING);
                        cell.setCellValue(value);
                    }
                }
            }
        }
        
    }
```


```
//修改合并单元格的值

//1.第2列固定，第一行到第三行合并
//此时光标在第一行
sheetOne.getRow( 1 ).getCell( 2 ).setCellValue( "" );

//待验证...
//2.第2行固定，第一列到第三列合并
//此时光标在第三列
sheetOne.getRow( 2 ).getCell( 3 ).setCellValue( "" );
```