## excel校验

1.校验单元格是否不为空

```

if (cell == null ||
    cell.getCellType()==Cell.CELL_TYPE_BLANK) {
                continue;
            }
```

2.格式化数据

```
 DataFormatter formatter = new DataFormatter();
 String name = formatter.formatCellValue(cell);

```