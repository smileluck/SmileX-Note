[toc]

---

# Maven导入

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>easyexcel</artifactId>
    <version>3.2.0</version>
</dependency>
```



# 指定字段（Integer）转文本渲染

1. 配置`Converter` 文件

```java
public class TypeConverter implements Converter<Integer> {
    @Override
    public Class supportJavaTypeKey() {
        return null;
    }

    @Override
    public CellDataTypeEnum supportExcelTypeKey() {
        return null;
    }

    @Override
    public Integer convertToJavaData(CellData cellData, ExcelContentProperty excelContentProperty, GlobalConfiguration globalConfiguration) throws Exception {
        return null;
    }

    @Override
    public CellData convertToExcelData(Integer integer, ExcelContentProperty excelContentProperty, GlobalConfiguration globalConfiguration) throws Exception {
        if (integer.equals(1)) {
            return new CellData("安卓");
        } else {
            return new CellData("苹果");
        }
    }
}
```

2. 再对应字段配置转换

```java
@ExcelProperty(value = "充值端口", converter = TypeConverter.class)
private Integer sysType;
```



# 设置Strategy 属性

## 注解使用

- 设置列宽@ColumnWidth

```java
@ColumnWidth(value=40)
private String name;
```

- 设置列属性。@ContentStyle
  - 设置边框
    - borderLeft
    - borderRight
    - borderTop
    - borderBottom
  - 设置对齐方式
    - horizontalAlignment（水平对齐）
    - verticalAlignment（垂直对齐）
  - 设置自动换行
    - wrapped。默认不换行

## 设置列宽度

### AbstractColumnWidthStyleStrategy

```java

public class WithdrawExcelStrategy extends AbstractColumnWidthStyleStrategy {
    @Override
    protected void setColumnWidth(WriteSheetHolder writeSheetHolder, List<CellData> list, Cell cell, Head head, Integer integer, Boolean aBoolean) {
        Sheet sheet = writeSheetHolder.getSheet();
        sheet.setColumnWidth(cell.getColumnIndex(), 10440);
    }
}

```

 **列宽计算逻辑** ：

1.  单位是1/256个字符宽度 ，所以代码中需要乘以256，且两个参数都必须为整数。
2. 正常的默认列宽是 (33英寸)`33*256`。但因为字体大小、单元格边框等占用额外的像素，所以实际的列宽比设置的**小0.78英寸**。
3. 举个例子，加入我们想要设置40英寸，那么就是`40.78*256`，但是因为参数要求为整数，那么就是`40.78*256=10439.68`，向上取整，结果为10440。
4. 最后。0.78是不同版本的误差值，这个可以根据个人情况决定误差允许范围。