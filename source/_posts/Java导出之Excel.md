---
title: Java导出之Excel
abbrlink: 55014
date: 2021-10-22 22:33:06
tags: [Java,POI]
categories: Java
description: Java通过POI生成Excel并返回前台下载

---

# Java通过模板导出Excel

## POI

Apache POI是Apache软件基金会的开放源码库，POI提供API给Java程序对Microsoft Office格式文件读和写的功能。

## HSSF和XSSF

针对不同版本的Excel，在POI中提供了HSSF和XSSF不同的包。
HSSF  － 提供读写Microsoft Excel XLS格式档案的功能。
XSSF  － 提供读写Microsoft Excel OOXML XLSX格式档案的功能。

## 相关依赖

```
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>4.1.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>4.1.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml-schemas</artifactId>
            <version>4.1.2</version>
        </dependency>
```

## 工具类

writeExcel()f方法根据模板生成Excel，writeToResponse()方法将生成的Excel写入response中返回前端下载。

```java
public class ExcelUtils {
    private static final String REG = "\\{([a-zA-Z_1-9]+)\\}";// 匹配"{exp}"
    private static final String REG_LIST = "\\{\\.([a-zA-Z_1-9]+)\\}";// 匹配"{.exp}"
    private static final Pattern PATTERN = Pattern.compile(REG);
    private static final Pattern PATTERN_LIST = Pattern.compile(REG_LIST);



    private ExcelUtils() {
    }

    /**
     * 根据模板生成Excel文件
     *
     * @param templateFilePath 模版文件路径
     * @param context      表头或表尾数据集合
     * @param dataList     列表
     * @return
     */
    public static byte[] writeExcel(String templateFilePath, Map<String, Object> context,
                                    List<?> dataList) {
       // File templateFile = null;
        ClassPathResource classPathResource = new ClassPathResource(templateFilePath);
        InputStream inputStream = null;

        try {
            inputStream = classPathResource.getInputStream();
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException("获取模板失败!", e);
        }
        try (HSSFWorkbook workbook = new HSSFWorkbook(inputStream)) {
            Sheet sheet = workbook.getSheetAt(0);// 获取配置文件sheet 页
            int listStartRowNum = -1;
            for (int i = sheet.getFirstRowNum(); i <= sheet.getLastRowNum(); i++) {
                Row row = sheet.getRow(i);
                if (row != null) {
                    for (int j = 0; j < row.getLastCellNum(); j++) {
                        Cell cell = row.getCell(j);
                        if (cell != null && cell.getCellType() == CellType.STRING) {
                            String cellValue = cell.getStringCellValue();
                            // 获取到列表数据所在行
                            if (listStartRowNum == -1 && cellValue.matches(REG_LIST)) {
                                listStartRowNum = i;
                            }

                            Object newValue = cellValue;
                            Matcher matcher = PATTERN.matcher(cellValue);
                            while (matcher.find()) {
                                String replaceExp = matcher.group();// 匹配到的表达式
                                String key = matcher.group(1);// 获取key
                                Object replaceValue = context.get(key);
                                if (replaceValue == null) {
                                    replaceValue = "";
                                }
                                if (replaceExp.equals(cellValue)) {// 单元格是一个表达式
                                    newValue = replaceValue;
                                } else {// 以字符串替换
                                    newValue = ((String) newValue).replace(replaceExp, replaceValue.toString());
                                }
                            }
                            setCellValue(cell, newValue);

                        }

                    }

                }
            }
            if (-1 != listStartRowNum) {// 如果不为 -1 说明有需要循环的列表表达式
                Row listStartRow = sheet.getRow(listStartRowNum);
                if (CollectionUtils.isEmpty(dataList)) {// 列表数据为空，清空列表表达式行
                    for (int i = 0; i < listStartRow.getLastCellNum(); i++) {
                        Cell cell = listStartRow.getCell(i);
                        if (cell != null) {
                            cell.setCellValue("");
                        }
                    }
                } else {
                    int lastCellNum = listStartRow.getLastCellNum();
                    if (listStartRowNum + 1 <= sheet.getLastRowNum()) {
                        sheet.shiftRows(listStartRowNum + 1, sheet.getLastRowNum(), dataList.size(), true, false);// 列表数据行后面行下移，留出数据填充区域
                    }
                    for (int i = 0; i < dataList.size(); i++) {// 循环列表数据 生成行
                        JSONObject jsonObj  = (JSONObject) JSON.toJSON(dataList.get(i));
                        //Map<String, Object> map = dataList.get(i);// 一行数据
                        int newRowNum = listStartRowNum + i + 1;// 保留表达式行
                        Row newRow = sheet.createRow(newRowNum);// 创建新行
                        for (int j = 0; j < lastCellNum; j++) {// 循环遍历单元格
                            Cell cell = listStartRow.getCell(j);// 列表数据行

                            // 填充数据
                            if (cell != null) {
                                Cell newCell = newRow.createCell(j);
                                newCell.setCellStyle(cell.getCellStyle());// 设置单元格格式

                                if (cell.getCellType() == CellType.STRING
                                        && cell.getStringCellValue().matches(REG_LIST)) {// 单元格是一个表达式
                                    String cellExp = cell.getStringCellValue();
                                    Matcher matcher = PATTERN_LIST.matcher(cellExp);
                                    matcher.find();
                                    String key = matcher.group(1);// 获取key
                                    Object newValue = jsonObj.get(key);
                                    if (newValue == null) {
                                        newValue = "";
                                    }
                                    setCellValue(newCell, newValue);
                                } else {// 不是表达式复制单元格数据
                                    CellType cellType = cell.getCellType();
                                    if (cellType == CellType.NUMERIC) {
                                        newCell.setCellValue(cell.getNumericCellValue());
                                    } else if (cellType == CellType.BOOLEAN) {
                                        newCell.setCellValue(cell.getBooleanCellValue());
                                    } else if (cellType == CellType.STRING) {
                                        newCell.setCellValue(cell.getStringCellValue());
                                    } else if (cellType == CellType.FORMULA) {
                                        // 处理公式，待实现
                                    } else {
                                        newCell.setCellValue(cell.getStringCellValue());
                                    }
                                }
                            }
                        }
                    }
                    sheet.removeRow(listStartRow);// 删除list表达式行
                    sheet.shiftRows(listStartRowNum + 1, sheet.getLastRowNum(), -1, true, false);// 数据区域上移一行，覆盖表达式行

                    // 合并单元格处理
                    for (int i = 0; i < lastCellNum; i++) {
                        CellRangeAddress mergedRangeAddress = getMergedRangeAddress(sheet, listStartRowNum, i);
                        if (mergedRangeAddress != null) {// 合并的单元格
                            i = mergedRangeAddress.getLastColumn();
                            for (int j = 1; j < dataList.size(); j++) {
                                int newRowNum = listStartRowNum + j;
                                sheet.addMergedRegionUnsafe(new CellRangeAddress(newRowNum, newRowNum,
                                        mergedRangeAddress.getFirstColumn(), mergedRangeAddress.getLastColumn()));
                            }
                        }
                    }
                }
            }
            // 公式生效
            sheet.setForceFormulaRecalculation(true);
            sheet.getPrintSetup().setPaperSize(PrintSetup.A4_PAPERSIZE);
            sheet.getPrintSetup().setLandscape(true);
            // FitHeight=1, 将所有行都缩放显示在一页上（设置1表示一页显示完，如果设置2表示分2页显示完）
            // FitWidth=1, 将所有列都缩放显示在一页上
            // 两个都等于1时，如果行数太多则会挤压列，一般来说只设置一个FitWidth=1，让行数自动换页
            // 要使这两个参数有效，则需要设置FitToPage=true
            sheet.setFitToPage(true);
            sheet.getPrintSetup().setFitWidth((short) 1);
//          sheet.getPrintSetup().setFitHeight((short)1);
            // 是否显示自动换页符
            sheet.setAutobreaks(true);
            ByteArrayOutputStream out = new ByteArrayOutputStream();
            workbook.write(out);
            return out.toByteArray();
        } catch (Exception e) {
            throw new ExcelException("生成excel失败!", e);
        }
    }

    private static void setCellValue(Cell cell, Object value) {
        if (value instanceof Number) {// 如果是数字类型的设置为数值
            cell.setCellValue(Double.parseDouble(value.toString()));
        } else if (value instanceof Date) {// 如果为时间类型的设置为时间
            cell.setCellValue((Date) value);
        } else if (value instanceof String) {
            cell.setCellValue((String) value);
        } else if (value instanceof Boolean) {
            cell.setCellValue((Boolean) value);
        } else {
            cell.setCellValue(value.toString());
        }
    }

    /**
     * 获取指定行/列的合并单元格区域
     *
     * @param sheet
     * @param row
     * @param column
     * @return CellRangeAddress 不是合并单元格返回null
     */
    private static CellRangeAddress getMergedRangeAddress(Sheet sheet, int row, int column) {
        List<CellRangeAddress> mergedRegions = sheet.getMergedRegions();
        for (CellRangeAddress cellAddresses : mergedRegions) {
            if (row >= cellAddresses.getFirstRow() && row <= cellAddresses.getLastRow()
                    && column >= cellAddresses.getFirstColumn() && column <= cellAddresses.getLastColumn()) {
                return cellAddresses;
            }
        }
        return null;
    }



    static class ExcelException extends RuntimeException {

        private static final long serialVersionUID = -2772261598232964002L;

        public ExcelException(String msg, Throwable e) {
            super(msg, e);
        }

        public ExcelException(String msg) {
            super(msg);
        }
    }

    /**
     * Description: 1、通过浏览器以流的形式输出,为了处理中文表名问题.
     *
     * @param bytes 文件对象
     * @param request
     * @param response
     * @param fileName 文件名
     */
    public static void writeToResponse(byte[] bytes, HttpServletRequest request, HttpServletResponse response, String fileName) {
        try {
            String userAgent = request.getHeader("User-Agent");
            // 解决中文乱码问题
            String fileName1 =  fileName + ".xls";
            String newFilename = URLEncoder.encode(fileName1, "UTF8");
            // 如果没有userAgent，则默认使用IE的方式进行编码，因为毕竟IE还是占多数的
            String rtn = "filename=\"" + newFilename + "\"";
            if (userAgent != null) {
                userAgent = userAgent.toLowerCase();
                // IE浏览器，只能采用URLEncoder编码
                if (userAgent.indexOf("IE") != -1)
                {
                    rtn = "filename=\"" + newFilename + "\"";
                }
                // Opera浏览器只能采用filename*
                else if (userAgent.indexOf("OPERA") != -1)
                {
                    rtn = "filename*=UTF-8''" + newFilename;
                }
                // Safari浏览器，只能采用ISO编码的中文输出
                else if (userAgent.indexOf("SAFARI") != -1)
                {
                    rtn = "filename=\"" + new String(fileName1.getBytes("UTF-8"), "ISO8859-1")
                            + "\"";
                }
                // FireFox浏览器，可以使用MimeUtility或filename*或ISO编码的中文输出
                else if (userAgent.indexOf("FIREFOX") != -1)
                {
                    rtn = "filename*=UTF-8''" + newFilename;
                }
            }

            String headStr = "attachment;  " + rtn;
            response.setContentType("multipart/form-data");
            response.setCharacterEncoding("utf-8");
            response.setHeader("Content-Disposition", headStr);
            // 响应到客户端
            OutputStream outputStream = response.getOutputStream();
            outputStream.write(bytes);
            outputStream.flush();

        } catch (UnsupportedEncodingException e) {

            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

}
```

## 模板制作

制作模板时注意表格内数据与表头表尾数据变量命名区别，表格内数据变量前添加【.】用于标识表格数据。
![](https://raw.githubusercontent.com/zhjAdm/ImageHosting/main/20211101104241.png)
