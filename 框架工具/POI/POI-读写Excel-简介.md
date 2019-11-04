### POI简介

POI是一个操作微软Office文档的Java软件工具包，其归属于Apache。

### POI(Excel)  结构

POI(Excel) 分为POI-HSSF和POI-XSSF。

##### POI-HSSF

POI-HSSF是用来操作Excel2003之前的版本（包括2003版本），Excel后缀名为.xls。

POI-HSSF分为eventmodel和usermodel（HSSF）。

##### POI-XSSF

POI-XSSF是用来操作Excel2007之后的版本（包括2007版本），Excel后缀名为.xlsx。

POI-XSSF分为eventmodel,usermodel（XSSF）和SXSSF。

##### 详细说明

|POI-HSSF|POI-XSSF|
|:----:|:----:|
|**eventmodel**&nbsp;&nbsp;&nbsp;&nbsp;**usermodel（HSSF）**|**eventmodel**&nbsp;&nbsp;&nbsp;&nbsp;**usermodel（XSSF）**&nbsp;&nbsp;&nbsp;&nbsp;**SXSSF**|

***

|-|eventmodel|usermodel|SXSSF|
|:----:|:----:|:----:|:----:|
|CPU内存消耗|低|高|低|
|Excel操作|读|读，写|写|
|使用难度|相对复杂|相对简单|相对简单|
|基于模式|SAX|DOM|-|
|EXCEL大数据处理|支持|不支持|支持|
|支持Excel类型|.xls .xlsx|.xls .xlsx|.xlsx|
|Forward Only|Yes|No|Yes|

##### 补充说明

* 基于模式，SAX，DOM是两种不同的解析方式，SAX是逐行读取文件，是通过事件触发的形式来获取所需的数据，不需要把文档读取到内存中，DOM是将文档以DOM树的形式存放到内存中，然后通过对DOM树的操作来操作文档。SAX只能读取，DOM可以进行增删查改操作。SAX占用内存小，适合读取大文档。DOM适合对文档进行增删查改时使用，但是内存占用很大。

* usermodel中，HSSF，XSSF使用起来需要区分Excel的版本，使用起来有些不便，故usermodel提供了SS来处理所有类型的Excel。

* SXSSF建立在XSSF上，是基于XSSF的一个流拓展，相对比XSSF，消耗的资源更小。
  SXSSF相对XSSF来说，会设定一个值，这个值会限制内存中的行数。写EXCEL文档时，XSSF会把数据都写到内存中，SXSSF则不是，SXSSF会设置写入内存中的数据的行数，其他的存放在硬盘中。

* SXSSF有俩种模式来实现数据由内存向硬盘的转移，一种是，当内存中数据的行数到达设定的值时，下一行的数据写入到内存时，会自动将最早写入内存的一行数据转移到硬盘中。另一种是，通过flushRows方法手动批量的将数据由内存转移到硬盘中。

* .xlsx 是基于xml的，处理起来比.xls的二进制文件占用内存大。

### POI操作Excel 示例

* 示例操作的是.xlsx格式的Excel。

* 示例使用的POI版本为3.17

* 示例仅仅是方便更好的理解，忽略了很多细节，也不支持所有的情况。

* 示例代码为了简洁，条例化，便于理解。在注释格式，代码上进行了简化处理，会有一些不严谨之处。

##### 事件模式（eventmodel）

* 基本代码

``` java
// Excel路径和Excel列数
List<String[]> dataList = ExcelReader.readExcel("F:\\Demo.xlsx", 4);
dataList.forEach(strings -> {
    Arrays.stream(strings).map(str -> str + " ").forEach(System.out::print);
    System.out.println();
});
```

``` java
public class ExcelReader {

    enum DataType {
        SST_INDEX, NUMBER, INLINE_STR
    }

    // Excel路径
    private String path;
    // 列总数
    private int columns;

    private ExcelReader(String path, int columns) {
        this.path = path;
        this.columns = columns;
    }

    // Excel处理
    private List<String[]> process() throws IOException, SAXException, OpenXML4JException, ParserConfigurationException {

        OPCPackage opcPackage = OPCPackage.open(path, PackageAccess.READ);
        ReadOnlySharedStringsTable stringsTable = new ReadOnlySharedStringsTable(opcPackage);
        XSSFReader xssfReader = new XSSFReader(opcPackage);
        List<String[]> list = new ArrayList<>();

        Iterator<InputStream> sheets = xssfReader.getSheetsData();
        while (sheets.hasNext()) {

            InputStream stream = sheets.next();
            list.addAll(processSheet(stringsTable, stream));
            stream.close();
        }
        opcPackage.close();
        return list;
    }

    // 处理sheet
    private List<String[]> processSheet(ReadOnlySharedStringsTable strings, InputStream sheetInputStream)
            throws IOException, ParserConfigurationException, SAXException {

        InputSource sheetSource = new InputSource(sheetInputStream);
        SAXParserFactory saxFactory = SAXParserFactory.newInstance();
        SAXParser saxParser = saxFactory.newSAXParser();
        XMLReader sheetParser = saxParser.getXMLReader();

        ExcelReader.SheetHandler handler = new ExcelReader.SheetHandler(strings, columns);
        sheetParser.setContentHandler(handler);
        sheetParser.parse(sheetSource);
        return handler.getRows();
    }

    // 读取Excel入口方法
    public static List<String[]> readExcel(String path, int columns) throws OpenXML4JException, IOException, ParserConfigurationException, SAXException {

        ExcelReader excelReader = new ExcelReader(path, columns);
        return excelReader.process();
    }

    private class SheetHandler extends DefaultHandler {

        private ReadOnlySharedStringsTable sharedStringsTable;
        // 最大列数
        private final int maxColumnCount;
        // 是否是Value
        private boolean isValue;
        // 下一个数据类型
        private DataType nextDataType;
        // 当前列
        private int currentColumn = -1;
        private StringBuffer value;
        // Excel行数据
        private String[] record;
        // Excel表数据
        private List<String[]> rows = new ArrayList<>();
        private DecimalFormat df = new DecimalFormat("0");

        private SheetHandler(ReadOnlySharedStringsTable strings, int maxColumnCount) {

            this.sharedStringsTable = strings;
            this.maxColumnCount = maxColumnCount;
            this.value = new StringBuffer();
            record = new String[this.maxColumnCount];
            rows.clear();
        }

        @Override
        public void startElement(String uri, String localName, String name, Attributes attributes) {

            if ("c".equals(name)) {

                currentColumn = getCurrentColumn(attributes.getValue("r"));
                nextDataType = DataType.NUMBER;

                if ("s".equals(attributes.getValue("t"))) {
                    nextDataType = DataType.SST_INDEX;
                }
                if ("inlineStr".equals(attributes.getValue("t"))) {
                    nextDataType = DataType.INLINE_STR;
                }
            }

            if ("v".equals(name) || "t".equals(name)) {

                isValue = true;
                value.setLength(0);
            }
        }

        @Override
        public void endElement(String uri, String localName, String name) {

            String str = null;
            if ("v".equals(name) || "t".equals(name)) {

                if (DataType.SST_INDEX.equals(nextDataType)) {
                    str = new XSSFRichTextString(sharedStringsTable.getEntryAt(Integer.parseInt(value.toString()))).toString();
                }
                if (DataType.INLINE_STR.equals(nextDataType)) {
                    str = value.toString();
                }
                if (DataType.NUMBER.equals(nextDataType)) {
                    str = df.format(new BigDecimal(value.toString()));
                }

                if (null == str) {
                    str = "";
                }
                record[currentColumn] = str;
            }

            if ("row".equals(name)) {

                if (columns > 0) {
                    if (isNullArray()) {
                        return;
                    }
                    rows.add(record.clone());
                    for (int i = 0; i < record.length; i++) {
                        record[i] = null;
                    }
                }
            }
        }

        @Override
        public void characters(char[] ch, int start, int length) {

            if (isValue) {
                value.append(ch, start, length);
            }
        }

        private List<String[]> getRows() {
            return rows;
        }

        // 获取当前列数
        private int getCurrentColumn(String str) {

            int firstDigit = -1;
            for (int c = 0; c < str.length(); c++) {
                if (Character.isDigit(str.charAt(c))) {
                    firstDigit = c;
                    break;
                }
            }
            return nameToColumn(str.substring(0, firstDigit));
        }

        // 通过名字转换为列数（从0开始）
        private int nameToColumn(String name) {

            int column = -1;
            for (int i = 0; i < name.length(); i++) {
                int j = name.charAt(i);
                column = (column + 1) * 26 + j - 'A';
            }
            return column;
        }

        // 判断是否为空数组
        private Boolean isNullArray() {

            if (Arrays.stream(record).filter(str -> null != str).count() > 0) {
                return false;
            }
            return true;
        }
    }
}
```

* Excel内容

sheet名：Demo

sheet内容：

|学号|姓名|性别|专业|
|:----:|:----:|:----:|:----:|
|1001|小白|女|计算机科学与技术|
|1002|小黑|男|软件工程|

* 程序输出结果

```
学号 姓名 性别 专业 
1001 小白 女 计算机科学与技术 
1002 小黑 男 软件工程 
```

##### 用户模式（usermodel）

###### 读Excel（XSSF）

* 基本代码

``` java
// 获得Excel操作对象
XSSFWorkbook xwb = new XSSFWorkbook("F:\\Demo.xlsx");

// 遍历Sheet
for (int i = 0; i < xwb.getNumberOfSheets(); i++) {

    // 获得当前Sheet
    XSSFSheet sheet = xwb.getSheetAt(i);
    // 获得当前Sheet的名字
    String sheetName = sheet.getSheetName();
    System.out.println(sheetName);

    // 遍历当前Sheet的行
    for (int j = 0; j <= sheet.getLastRowNum(); j++) {

        // 获得当前行
        XSSFRow row = sheet.getRow(j);

        // 遍历当前行的单元格
        for (int k = 0; k < row.getLastCellNum(); k++) {

            // 获得当前单元格
            XSSFCell cell = row.getCell(k);

            // 当前单元格为字符串格式
            if (CellType.STRING == cell.getCellTypeEnum()) {
                String stringContent = cell.getStringCellValue();
                System.out.print(stringContent + " ");
            }

            // 当前单元格为数字格式
            if (CellType.NUMERIC == cell.getCellTypeEnum()) {
                double doubleContent = cell.getNumericCellValue();
                System.out.print((int) doubleContent + " ");
            }
        }
        System.out.println();
    }
}
```

* Excel内容

sheet名： Demo

sheet内容：

|学号|姓名|性别|专业|
|:----:|:----:|:----:|:----:|
|1001|小白|女|计算机科学与技术|
|1002|小黑|男|软件工程|

* 程序输出结果

```
Demo
学号 姓名 性别 专业 
1001 小白 女 计算机科学与技术 
1002 小黑 男 软件工程 
```

###### 写Excel（XSSF）

* 基本代码

``` java
// 数据源
List<String[]> data = new ArrayList<>();
data.add(new String[]{"学号", "姓名", "性别", "专业"});
data.add(new String[]{"1001", "小白", "女", "计算机科学与技术"});
data.add(new String[]{"1002", "小黑", "男", "软件工程"});

XSSFWorkbook xwb = new XSSFWorkbook();
// 创建sheet
XSSFSheet sheet = xwb.createSheet("Demo");

for (int i = 0; i < data.size(); i++) {

    // 创建行
    XSSFRow row = sheet.createRow(i);
    String[] content = data.get(i);

    for (int j = 0; j < content.length; j++) {

        // 创建单元格
        if (0 != i && j == 0) {
            row.createCell(j).setCellValue(Double.valueOf(content[j]));
            continue;
        }
        row.createCell(j).setCellValue(content[j]);
    }
}

// 生成Excel
FileOutputStream out = new FileOutputStream("F:\\Demo.xlsx");
xwb.write(out);
out.close();
```

* 生成Excel内容

sheet名： Demo

sheet内容：

|学号|姓名|性别|专业|
|:----:|:----:|:----:|:----:|
|1001|小白|女|计算机科学与技术|
|1002|小黑|男|软件工程|

###### 读Excel（SS）

* 基本代码

``` java 
// 获得Excel操作对象
Workbook wb = new XSSFWorkbook("F:\\Demo.xlsx");

// 遍历Sheet
for (int i = 0; i < wb.getNumberOfSheets(); i++) {

    // 获得当前Sheet
    Sheet sheet = wb.getSheetAt(i);
    // 获得当前Sheet的名字
    String sheetName = sheet.getSheetName();
    System.out.println(sheetName);

    // 遍历当前Sheet的行
    for (int j = 0; j <= sheet.getLastRowNum(); j++) {

        // 获得当前行
        Row row = sheet.getRow(j);

        // 遍历当前行的单元格
        for (int k = 0; k < row.getLastCellNum(); k++) {

            // 获得当前单元格
            Cell cell = row.getCell(k);

            // 当前单元格为字符串格式
            if (CellType.STRING == cell.getCellTypeEnum()) {
                String stringContent = cell.getStringCellValue();
                System.out.print(stringContent + " ");
            }

            // 当前单元格为数字格式
            if (CellType.NUMERIC == cell.getCellTypeEnum()) {
                double doubleContent = cell.getNumericCellValue();
                System.out.print((int) doubleContent + " ");
            }
        }
        System.out.println();
    }
}
```

* Excel内容

sheet名： Demo

sheet内容：

|学号|姓名|性别|专业|
|:----:|:----:|:----:|:----:|
|1001|小白|女|计算机科学与技术|
|1002|小黑|男|软件工程|

* 程序输出结果

```
Demo
学号 姓名 性别 专业 
1001 小白 女 计算机科学与技术 
1002 小黑 男 软件工程 
```

###### 写Excel（SS）

* 基本代码

``` java
// 数据源
List<String[]> data = new ArrayList<>();
data.add(new String[]{"学号", "姓名", "性别", "专业"});
data.add(new String[]{"1001", "小白", "女", "计算机科学与技术"});
data.add(new String[]{"1002", "小黑", "男", "软件工程"});

Workbook wb = new XSSFWorkbook();
// 创建sheet
Sheet sheet = wb.createSheet("Demo");

for (int i = 0; i < data.size(); i++) {
    
    // 创建行
    Row row = sheet.createRow(i);
    String[] content = data.get(i);

    for (int j = 0; j < content.length; j++) {
        
        // 创建单元格
        if (0 != i && 0 == j) {
            row.createCell(j).setCellValue(Double.valueOf(content[j]));
            continue;
        }
        row.createCell(j).setCellValue(content[j]);
    }
}

// 生成Excel
FileOutputStream out = new FileOutputStream("F:\\Demo.xlsx");
wb.write(out);
out.close();
```

* 生成Excel内容

sheet名： Demo

sheet内容：

|学号|姓名|性别|专业|
|:----:|:----:|:----:|:----:|
|1001|小白|女|计算机科学与技术|
|1002|小黑|男|软件工程|

##### SXSSF

* 基本代码（俩种模式）

``` java
// 数据源
List<String[]> data = new ArrayList<>();
data.add(new String[]{"学号", "姓名", "性别", "专业"});
data.add(new String[]{"1001", "小白", "女", "计算机科学与技术"});
data.add(new String[]{"1002", "小黑", "男", "软件工程"});

// 自动转移设置放在内存中的行数
SXSSFWorkbook swb = new SXSSFWorkbook(1000);
// 创建sheet
SXSSFSheet sheet = swb.createSheet("Demo");

for (int i = 0; i < data.size(); i++) {

    // 创建行
    SXSSFRow row = sheet.createRow(i);
    String[] content = data.get(i);

    for (int j = 0; j < content.length; j++) {

        // 创建单元格
        if (0 != i && 0 == j) {
            row.createCell(j).setCellValue(Double.valueOf(content[j]));
            continue;
        }
        row.createCell(j).setCellValue(content[j]);
    }
}

// 生成Excel
FileOutputStream out = new FileOutputStream("F:\\Demo.xlsx");
swb.write(out);
out.close();
```

``` java
// 数据源
List<String[]> data = new ArrayList<>();
data.add(new String[]{"学号", "姓名", "性别", "专业"});
data.add(new String[]{"1001", "小白", "女", "计算机科学与技术"});
data.add(new String[]{"1002", "小黑", "男", "软件工程"});

// 关闭自动转移（内存向硬盘）
SXSSFWorkbook swb = new SXSSFWorkbook(-1);
// 创建sheet
SXSSFSheet sheet = swb.createSheet("Demo");

for (int i = 0; i < data.size(); i++) {

    if (0 != i && 0 == i % 1000) {
        // 手动转移（内存向硬盘）
        sheet.flushRows(1000);
    }

    // 创建行
    SXSSFRow row = sheet.createRow(i);
    String[] content = data.get(i);

    for (int j = 0; j < content.length; j++) {

        // 创建单元格
        if (0 != i && 0 == j) {
            row.createCell(j).setCellValue(Double.valueOf(content[j]));
            continue;
        }
        row.createCell(j).setCellValue(content[j]);
    }
}

// 生成Excel
FileOutputStream out = new FileOutputStream("F:\\Demo.xlsx");
swb.write(out);
out.close();
```

* 生成Excel内容

sheet名： Demo

sheet内容：

|学号|姓名|性别|专业|
|:----:|:----:|:----:|:----:|
|1001|小白|女|计算机科学与技术|
|1002|小黑|男|软件工程|

### 注意事项

* 数字类型处理注意科学计数法
* Excel只支持15位有效数字
* Excel2003最多有65536行和256列
* Excel2007最多有1048576行和16384列

### 源码地址

[https://github.com/CunYu/poi-excel-demo.git](https://github.com/CunYu/poi-excel-demo.git)