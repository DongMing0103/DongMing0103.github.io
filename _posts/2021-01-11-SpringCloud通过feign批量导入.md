---
layout:     post
title:      SpringCloud
subtitle:   SpringCloud通过feign批量导入
date:       2021-01-18
author:     dm
header-img: img/post-sample-image.jpg
catalog: true
tags:
    - SpringCloud
    - feign

---



*问题背景：SpringCloud通过feign服务间调用上传Excel文件及Excel文件下载*

``` 
服务关系：
服务提供方A（接收文件）：解析Excel文件
服务消费方B（发送文件）: 上传Excel文件及逻辑处理
B服务通过feign调用A服务
```



---

# 1.文件上传

## 1.1 添加依赖

`B服务pom文件中添加依赖`

```java
<!-- Feign进行跨服务传递文件依赖 -->
 <dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.3</version>
</dependency>
<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form</artifactId>
    <version>3.4.1</version>
</dependency>
<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form-spring</artifactId>
    <version>3.4.1</version>
</dependency>
```

## 1.2 添加配置类（A/B服务）

`用来编码转换`

```java
import feign.codec.Encoder;
import org.springframework.beans.factory.ObjectFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.web.HttpMessageConverters;
import org.springframework.cloud.netflix.feign.support.SpringEncoder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.context.annotation.Scope;

@Configuration
public class MultipartSupportConfig {

  @Autowired private ObjectFactory<HttpMessageConverters> messageConverters;

  @Bean
  @Primary
  @Scope("prototype")
  public Encoder multipartFormEncoder() {
    return new SpringMultipartEncoder(new SpringEncoder(messageConverters));
  }

  @Bean
  public feign.Logger.Level multipartLoggerLevel() {
    return feign.Logger.Level.FULL;
  }
}
```

## 1.3 client接口引用该配置（A/B服务）

`feign调用，增加@FeignClient(name = "xcb-common", configuration = MultipartSupportConfig.class)注解`

```java
import com.xcb.common.domain.vo.ExcelVo;
import com.xcb.tmscargo.config.MultipartSupportConfig;
import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestPart;
import org.springframework.web.multipart.MultipartFile;

import java.util.List;

/** @Author dongm @Description: @Date 2020/10/16 */
@FeignClient(name = "xcb-common", configuration = MultipartSupportConfig.class)
public interface ExcelClient {

  /**
   * @author: dongm @Description: 导入货源
   * @date: 2021/1/6
   * @param: [file]
   * @return: List<ExcelVo>
   */
  @RequestMapping(
      value = "/api/admin/importExcel",
      method = RequestMethod.POST,
      consumes = {MediaType.MULTIPART_FORM_DATA_VALUE},
      produces = {MediaType.APPLICATION_JSON_UTF8_VALUE})
  List<ExcelVo> importExcel(@RequestPart(value = "file") MultipartFile file);
}
```

## 1.4 接口调用

**注意点 1. @PostMapping(value = "/batchAdd",produces = {MediaType.APPLICATION_JSON_UTF8_VALUE},consumes = MediaType.MULTIPART_FORM_DATA_VALUE) **

**注意点 2. 使用 @RequestPart 非 @RequestParam()**

***注意点2、3     A/B服务相同***

**注意点 3. 使用CommonsMultipartFile转换后上传**

``` java
DiskFileItem fileItem =
        (DiskFileItem)
            new DiskFileItemFactory()
                .createItem(
                    "file",
                    MediaType.MULTIPART_FORM_DATA_VALUE,
                    true,
                    files.getOriginalFilename());
    InputStream input = files.getInputStream();
    OutputStream os = fileItem.getOutputStream();
    IOUtils.copy(input, os);
    MultipartFile file = new CommonsMultipartFile(fileItem);
```

***完整示例***

```java
@ApiOperation(value = "批量添加货源", httpMethod = "POST")
@PostMapping(
    value = "/batchAdd",
    produces = {MediaType.APPLICATION_JSON_UTF8_VALUE},
    consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
@Transactional(rollbackFor = Exception.class)
@AvoidRepeatableCommit
public ResponseEntity<XcbResponse> batchAdd(@RequestPart("file") MultipartFile files)
    throws IOException {
  try {
    // CommonsMultipartFile转换
    DiskFileItem fileItem =
        (DiskFileItem)
            new DiskFileItemFactory()
                .createItem(
                    "file",
                    MediaType.MULTIPART_FORM_DATA_VALUE,
                    true,
                    files.getOriginalFilename());
    InputStream input = files.getInputStream();
    OutputStream os = fileItem.getOutputStream();
    IOUtils.copy(input, os);
    MultipartFile file = new CommonsMultipartFile(fileItem);
    // 解析Excel
    List<ExcelVo> excelVos = excelClient.importExcel(file);

    // 获取解析数据
    for (int i = 0; i < excelVos.size(); i++) {
      // 获取对应参数
      Map<String, String> dataMap = excelVos.get(i).getDataMap();
      String account = dataMap.get("发货用户账号");
      String contactPerson = dataMap.get("联系人姓名");
      String contactPhone = dataMap.get("联系电话");
      String startDateTime = dataMap.get("装货开始日期");
      String vaildDays = dataMap.get("发货周期");
      String startDockCityName = dataMap.get("发货地-市");
      String startDockName = dataMap.get("装货码头");
      String endDockCityName = dataMap.get("卸货地-市");
      String endDockName = dataMap.get("卸货码头");
      String goodsName = dataMap.get("货物名称");
      String deliveryMount = dataMap.get("货物吨位");
      String isSecrecy = dataMap.get("是否为隐私货源");
      String freight = dataMap.get("运价-元/吨");
      String cargoReqMin = dataMap.get("船只最小吨位要求-吨");
      String cargoReqMax = dataMap.get("船只最大吨位要求-吨");
      String remark = dataMap.get("备注");
    }
  } catch (IOException e) {
    e.printStackTrace();
  }
  return new ResponseEntity<>(
      XcbResponse.createBySuccess(batchAddCargoOwnerSupplyVo.returnText()), HttpStatus.OK);
}
```

## 1.5 A服务解析导入Excel文件信息

```java
@SneakyThrows
@Override
public List<ExcelVo> importExcelList(MultipartFile file) {
  // 判断文件格式
  Workbook workbook = getWorkbo(file);
  // 无效格式，抛出异常
  if (workbook == null) {
    throw new XcbServerException(CommonResultCode.FILE_FORMAT_ERROR);
  }
  List<ExcelVo> list = new ArrayList<ExcelVo>();
  try {
    // 表对象
    Sheet sheet = workbook.getSheetAt(0);
    // 总行数
    int rowLength = sheet.getLastRowNum();
    // 初始化map
    Map<Integer, String> map = new HashMap<>();
    // 正文内容从第3行开始，第一行为标题，第二行为示例
    for (int i = 0; i < rowLength + 1; i++) {
      Row row = sheet.getRow(i);
      // i = 0：获取第一行标题名称
      if (i == 0) {
        for (int j = 0; j < row.getLastCellNum(); j++) {
          // 获取到第j行的数据(单元格)
          Cell cell = row.getCell(j);
          cell.setCellType(CellType.STRING);
          map.put(j, cell.getStringCellValue());
        }
      }
      // i = 1：描述，可跳过
      else if (i == 1) {
        continue;
      }
      // 获取具体值
      else {
        // 初始化map、vo
        Map<String, String> maps = new HashMap<>();
        ExcelVo excelVo = new ExcelVo();
        for (int j = 0; j < row.getLastCellNum(); j++) {
          // 获取到第j行的数据(单元格)
          Cell cell = row.getCell(j);
          if (cell != null) {
            cell.setCellType(CellType.STRING);
            maps.put(map.get(j), cell.getStringCellValue());
          }
        }
        excelVo.setDataMap(maps);
        list.add(excelVo);
      }
    }
  } catch (Exception e) {
    log.error("parse excel file error ：" + e);
  }
  return list;
}
```

### 1.5.1 获取Excel文件格式

```java
/**
 * @author: dongm @Description: 获取文件格式信息
 * @date: 2020/10/17
 * @param: [file]
 * @return: Workbook
 */
private static Workbook getWorkbo(MultipartFile file) throws IOException {
  String fileName = file.getOriginalFilename();
  // 获取后缀格式名
  String fileSuffix = fileName.substring(fileName.lastIndexOf("."));
  Workbook workbo = null;
  // 根据后缀格式，判断返回方式
  if (Objects.equals(ExcelType.XLS.getName(), fileSuffix)) {
    workbo = new HSSFWorkbook(file.getInputStream());
  } else if (Objects.equals(ExcelType.XLSX.getName(), fileSuffix)) {
    workbo = new XSSFWorkbook(file.getInputStream());
  }
  return workbo;
}
```

### 1.5.2 接收解析后返回值实体

```java
@Data
public class ExcelVo {
  /** 返回参数数据 */
  @ApiModelProperty(notes = "返回参数数据")
  public Map<String, String> dataMap;
}
```

# 2. 文件下载Demo

```java
@SneakyThrows
@Override
public void downLoadExcelTemplate(HttpServletRequest request, HttpServletResponse response) {
  String path = "excel/importSupply.xlsx";
  String fileName = path.substring(path.lastIndexOf("/") + 1);

  Resource resource = new ClassPathResource(path);
  InputStream inputStream = resource.getInputStream();
  // 转为POI的Workbook输出流，这样下载后不报错，直接输出流，打开Excel可能提示错误
  @SuppressWarnings("resource")
  XSSFWorkbook workbook = new XSSFWorkbook(inputStream);
  String userAgent = request.getHeader("User-Agent");
  // 针对IE或者以IE为内核的浏览器
  if (userAgent.contains("MSIE") || userAgent.contains("Trident")) {
    fileName = URLUtil.encode(fileName, "UTF-8");
  } else {
    // 非IE浏览器的处理
    fileName = new String(fileName.getBytes("UTF-8"), "ISO-8859-1");
  }
  response.setHeader("Content-disposition", "attachment; filename=" + fileName);
  response.setContentType("application/vnd.ms-excel; charset=utf-8");
  response.setCharacterEncoding("UTF-8");
  // 将流中内容写出去
  try (OutputStream outputStream = response.getOutputStream()) {
    workbook.write(outputStream);
    outputStream.flush();
    outputStream.close();
  }
}
```