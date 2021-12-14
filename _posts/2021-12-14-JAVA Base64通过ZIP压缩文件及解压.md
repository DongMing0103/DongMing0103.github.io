---
layout:     post
title:      Base64通过ZIP压缩文件及解压
subtitle:   ZIP文件压缩
date:       2021-12-14
author:     dm
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - JAVA





---

### 图片压缩及文件上传

```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.parser.Feature;
import com.xcb.fun.util.FunctionUtils;
import com.xcb.pay.config.XcbPayProperties;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.inject.Inject;
import java.io.*;
import java.net.URL;
import java.util.Arrays;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

/** @Author dongm @Description: 接口工具类 @Date 2021/3/19 */
@Slf4j
@Component
public class BankUtils {
  @Inject XcbPayProperties xcbPayProperties;

  public static BankUtils bankUtils;

  @PostConstruct
  public void init() {
    bankUtils = this;
  }

  /**
   * @author: dongm @Description: 获取将对象转换成json字符串
   * @date: 2021/3/19
   * @param: [dto]
   * @return: String
   */
  public static <T> String getReqStr(T dto) throws Exception {
    // 数据转换
    Map<String, Object> reqMap = JsonUtils.beanWithOutNullToMap(dto);
    reqMap.remove("serialVersionUID");
    reqMap.remove("index");
    // 加签
    String signData =
        SignUtil.addSign(
            JSON.parseObject(
                JsonUtils.objectToJson(reqMap), LinkedHashMap.class, Feature.OrderedField),
            bankUtils.xcbPayProperties.getBank().getMd5_key());
    log.info("signData ========= {} ", signData);
    reqMap.put("sign_data", signData);
    String reqStr = JsonUtils.objectToJson(reqMap);
    return reqStr;
  }
  /**
   * @author: dongm @Description: 获取将对象转换成json字符串
   * @date: 2021/3/19
   * @param: [dto]
   * @return: String
   */
  public static <T> String getReqStr(T dto, String... removeFields) throws Exception {
    List<String> removeFieldList = (removeFields != null) ? Arrays.asList(removeFields) : null;

    // 数据转换
    Map<String, Object> reqMap = JsonUtils.beanWithOutNullToMap(dto);
    reqMap.remove("serialVersionUID");
    reqMap.remove("index");
    //    移除附加字段
    FunctionUtils.isTure(removeFieldList != null)
        .trueForHandle(
            () -> {
              removeFieldList.forEach(
                  e -> {
                    reqMap.remove(e);
                  });
            });

    // 加签
    String signData =
        SignUtil.addSign(
            JSON.parseObject(
                JsonUtils.objectToJson(reqMap), LinkedHashMap.class, Feature.OrderedField),
            bankUtils.xcbPayProperties.getBank().getMd5_key());
    log.info("signData ========= {} ", signData);
    reqMap.put("sign_data", signData);
    String reqStr = JsonUtils.objectToJson(reqMap);
    return reqStr;
  }

  /**
   * @author: dongm @Description: 获取文件流，并Base64转码
   * @date: 2021/8/3
   * @param: [path]
   * @return: String
   */
  public static String getFileData(String path) {
    String returnStr = null;
    try {
      String fileName = path;
      File file = getFile(path);
      InputStream in = new FileInputStream(file);
      // 图片压缩
      byte[] buff = PicUtils.compressPicForScale(new byte[(int) file.length()], file, 3072);
      in.read(buff);
      in.close();
      returnStr = ZipUtils.zipToString(buff, fileName);
      // returnStr = Base64.getEncoder().encodeToString(buff);
    } catch (Exception e) {
      e.printStackTrace();
    }
    return returnStr;
  }

  /**
   * @author: dongm @Description: 图片链接转为文件实体
   * @date: 2021/8/10
   * @param: [url]
   * @return: File
   */
  public static File getFile(String url) throws Exception {
    // 对本地文件命名
    String fileName = url.substring(url.lastIndexOf("."), url.length());
    File file = null;
    URL urlfile;
    InputStream inStream = null;
    OutputStream os = null;
    try {
      file = File.createTempFile("net_url", fileName);
      // 下载
      urlfile = new URL(url);
      System.out.printf("文件地址 ============== " + file.toString());
      inStream = urlfile.openStream();
      os = new FileOutputStream(file);
      int bytesRead = 0;
      byte[] buffer = new byte[8192];
      while ((bytesRead = inStream.read(buffer, 0, 8192)) != -1) {
        os.write(buffer, 0, bytesRead);
      }
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      try {
        // 退出jvm后删除临时文件
        file.deleteOnExit();
        if (null != os) {
          os.close();
        }
        if (null != inStream) {
          inStream.close();
        }
      } catch (Exception e) {
        e.printStackTrace();
      }
    }
    return file;
  }

  /**
   * @author: dongm @Description: 压缩文件
   * @date: 2021/12/6
   * @param: [url]
   * @return: String
   */
  public static String getZipFile(String url) {
    String returnStr = "";
    try {
      File file = getFile(url);
      FileInputStream fis = new FileInputStream(file);
      ByteArrayOutputStream bos = new ByteArrayOutputStream(1000);
      byte[] b = new byte[1000];
      int n;
      while ((n = fis.read(b)) != -1) {
        bos.write(b, 0, n);
      }
      fis.close();
      byte[] data = bos.toByteArray();
      bos.close();
      returnStr = ZipUtils.zipToString(data, "fileName");
//      log.info("文件base64加密之后的内容(太长, 控制台可能打印不出来):{}" + returnStr);
    } catch (Exception e) {
      e.printStackTrace();
    }
    return returnStr;
  }
}

```





### Zip压缩及解压缩方法

```java
import com.jcraft.jzlib.ZInputStream;
import com.xcb.pay.util.JavaUtil;
import lombok.extern.slf4j.Slf4j;

import java.io.*;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.nio.file.StandardOpenOption;
import java.util.Base64;
import java.util.List;
import java.util.zip.*;

@Slf4j
public class ZipUtils {

  private static final int BUFFER_SIZE = 2 * 1024;

  /**
   * 将bate[] ZIP压缩并base64编码
   *
   * @param data
   * @return
   */
  public static String zipToString(byte[] data, String fileName) {
    return Base64.getEncoder().encodeToString(zip(data, fileName));
  }

  /**
   * 将String base64解码 并ZIP解压缩
   *
   * @param data
   * @return
   */
  public static byte[] unZipToByteArray(String data) {
    return unZip(Base64.getDecoder().decode(data));
  }

  /***
   * 压缩GZip
   *
   * @param data
   * @return
   */
  public static byte[] gZip(byte[] data) {
    byte[] b = null;
    try {
      ByteArrayOutputStream bos = new ByteArrayOutputStream();
      GZIPOutputStream gzip = new GZIPOutputStream(bos);
      gzip.write(data);
      gzip.finish();
      gzip.close();
      b = bos.toByteArray();
      bos.close();
    } catch (Exception ex) {
      ex.printStackTrace();
    }
    return b;
  }
  /***
   * 解压GZip
   *
   * @param data
   * @return
   */
  public static byte[] unGZip(byte[] data) {
    byte[] b = null;
    try {
      ByteArrayInputStream bis = new ByteArrayInputStream(data);
      GZIPInputStream gzip = new GZIPInputStream(bis);
      byte[] buf = new byte[1024];
      int num = -1;
      ByteArrayOutputStream baos = new ByteArrayOutputStream();
      while ((num = gzip.read(buf, 0, buf.length)) != -1) {
        baos.write(buf, 0, num);
      }
      b = baos.toByteArray();
      baos.flush();
      baos.close();
      gzip.close();
      bis.close();
    } catch (Exception ex) {
      ex.printStackTrace();
    }
    return b;
  }

  /***
   * 压缩Zip
   *
   * @param data
   * @return
   */
  public static byte[] zip(byte[] data, String fileName) {
    byte[] b = null;
    try {
      ByteArrayOutputStream bos = new ByteArrayOutputStream();
      ZipOutputStream zip = new ZipOutputStream(bos);
      ZipEntry entry = new ZipEntry(fileName);
      entry.setSize(data.length);
      zip.putNextEntry(entry);
      zip.write(data);
      zip.closeEntry();
      zip.close();
      b = bos.toByteArray();
      bos.close();
    } catch (Exception ex) {
      ex.printStackTrace();
    }
    return b;
  }
  /***
   * 解压Zip
   *
   * @param data
   * @return
   */
  public static byte[] unZip(byte[] data) {
    byte[] b = null;
    try {
      ByteArrayInputStream bis = new ByteArrayInputStream(data);
      ZipInputStream zip = new ZipInputStream(bis);
      while (zip.getNextEntry() != null) {

        byte[] buf = new byte[1024];
        int num = -1;
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        while ((num = zip.read(buf, 0, buf.length)) != -1) {
          baos.write(buf, 0, num);
        }
        b = baos.toByteArray();
        baos.flush();
        baos.close();
      }
      zip.close();
      bis.close();
    } catch (Exception ex) {
      ex.printStackTrace();
    }
    return b;
  }

  /**
   * 压缩BZip2
   *
   * @param data
   * @return
   */
  /*public static byte[] bZip2(byte[] data) {
      byte[] b = null;
      try {
          ByteArrayOutputStream bos = new ByteArrayOutputStream();
          CBZip2OutputStream bzip2 = new CBZip2OutputStream(bos);
          bzip2.write(data);
          bzip2.flush();
          bzip2.close();
          b = bos.toByteArray();
          bos.close();
      } catch (Exception ex) {
          ex.printStackTrace();
      }
      return b;
  }*/

  /**
   * 解压BZip2
   *
   * @param data
   * @return
   */
  /* public static byte[] unBZip2(byte[] data) {
    byte[] b = null;
    try {
      ByteArrayInputStream bis = new ByteArrayInputStream(data);
      CBZip2InputStream bzip2 = new CBZip2InputStream(bis);
      byte[] buf = new byte[1024];
      int num = -1;
      ByteArrayOutputStream baos = new ByteArrayOutputStream();
      while ((num = bzip2.read(buf, 0, buf.length)) != -1) {
        baos.write(buf, 0, num);
      }
      b = baos.toByteArray();
      baos.flush();
      baos.close();
      bzip2.close();
      bis.close();
    } catch (Exception ex) {
      ex.printStackTrace();
    }
    return b;
  }*/

  /**
   * 把字节数组转换成16进制字符串
   *
   * @param bArray
   * @return
   */
  public static String bytesToHexString(byte[] bArray) {
    StringBuffer sb = new StringBuffer(bArray.length);
    String sTemp;
    for (int i = 0; i < bArray.length; i++) {
      sTemp = Integer.toHexString(0xFF & bArray[i]);
      if (sTemp.length() < 2) {
        sb.append(0);
      }
      sb.append(sTemp.toUpperCase());
    }
    return sb.toString();
  }

  /**
   * 压缩数据
   *
   * @param object
   * @return
   * @throws IOException
   */
  /*public static byte[] jzlib(byte[] object) {
      byte[] data = null;
      try {
          ByteArrayOutputStream out = new ByteArrayOutputStream();
          ZOutputStream zOut = new ZOutputStream(out,
                  JZlib.Z_DEFAULT_COMPRESSION);
          DataOutputStream objOut = new DataOutputStream(zOut);
          objOut.write(object);
          objOut.flush();
          zOut.close();
          data = out.toByteArray();
          out.close();
      } catch (IOException e) {
          e.printStackTrace();
      }
      return data;
  }*/

  /**
   * 解压被压缩的数据
   *
   * @return
   * @throws IOException
   */
  public static byte[] unjzlib(byte[] object) {
    byte[] data = null;
    try {
      ByteArrayInputStream in = new ByteArrayInputStream(object);
      ZInputStream zIn = new ZInputStream(in);
      byte[] buf = new byte[1024];
      int num = -1;
      ByteArrayOutputStream baos = new ByteArrayOutputStream();
      while ((num = zIn.read(buf, 0, buf.length)) != -1) {
        baos.write(buf, 0, num);
      }
      data = baos.toByteArray();
      baos.flush();
      baos.close();
      zIn.close();
      in.close();

    } catch (IOException e) {
      e.printStackTrace();
    }
    return data;
  }

  /**
   * 字符串压缩成gz文件
   *
   * @param inStr 压缩字符串
   * @param localPath 本地保存路径
   * @param fileName 文件名称
   * @param charsetName 编码格式
   */
  public static void compressStr(
      String inStr, String localPath, String fileName, String charsetName) {
    try {
      byte[] data = inStr.getBytes(charsetName);
      ByteArrayOutputStream out = new ByteArrayOutputStream();
      GZIPOutputStream gzip = new GZIPOutputStream(out);
      gzip.write(data);
      gzip.close();
      byte[] buf = out.toByteArray();
      File dirFile = new File(localPath + fileName + ".gz");
      if (!dirFile.exists()) {
        File file = new File(localPath);
        file.mkdirs();
      } else {
        dirFile.delete();
        dirFile.createNewFile();
      }
      FileChannel fcout = new RandomAccessFile(dirFile, "rws").getChannel();
      ByteBuffer wBuffer = ByteBuffer.allocateDirect(buf.length);
      fcout.write(wBuffer.wrap(buf), fcout.size());
      if (fcout != null) {
        fcout.close();
      }
    } catch (IOException e) {
      e.printStackTrace();
    }
  }

  /**
   * @author: dongm @Description: 压缩base64Zip
   * @date: 2021/12/13
   * @param: [srcFiles]
   * @return: String
   */
  public static String toBase64Zip(List<File> srcFiles) throws RuntimeException {
    log.info("开始压缩文件    [{}]", srcFiles);
    long start = System.currentTimeMillis();
    String base64toZip = "";
    ZipOutputStream zos = null;
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    try {
      zos = new ZipOutputStream(baos);
      for (File srcFile : srcFiles) {
        byte[] buf = new byte[BUFFER_SIZE];
        log.info("压缩的文件名称    [{}]   ", srcFile.getName() + "压缩的文件大小      [{}] ", srcFile.length());
        zos.putNextEntry(new ZipEntry(srcFile.getName()));
        int len;
        FileInputStream in = new FileInputStream(srcFile);
        while ((len = in.read(buf)) != -1) {
          zos.write(buf, 0, len);
        }
        zos.closeEntry();
        in.close();
      }
      long end = System.currentTimeMillis();
      log.info("压缩完成，耗时：    [{}] ms", (end - start));
    } catch (Exception e) {
      throw new RuntimeException("zip error from ZipToBase64", e);
    } finally {
      if (zos != null) {
        try {
          zos.close();
        } catch (IOException e) {
          e.printStackTrace();
        }
      }
    }

    byte[] refereeFileBase64Bytes = Base64.getEncoder().encode(baos.toByteArray());
    try {
      base64toZip = new String(refereeFileBase64Bytes, "UTF-8");
    } catch (Exception e) {
      throw new RuntimeException("压缩流出现异常", e);
    }
    return base64toZip;
  }

  /**
   * @author: dongm @Description: base64转文件
   * @date: 2021/12/13
   * @param: [base64]
   * @return: void
   */
  public static File Base64ToFile(String base64) throws RuntimeException {
    File file = null;
    try {
      byte[] byteBase64 = Base64.getDecoder().decode(base64);
      ByteArrayInputStream byteArray = new ByteArrayInputStream(byteBase64);
      ZipInputStream zipInput = new ZipInputStream(byteArray);
      ZipEntry entry = zipInput.getNextEntry();
      while (entry != null && !entry.isDirectory()) {
        String name = entry.getName() + JavaUtil.getCurrentTimestamp();
        log.info("文件名称:    [{}]", name);
        file = File.createTempFile("net_url", name);
        if (!file.exists()) {
          (new File(file.getParent())).mkdirs();
        }
        BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(file));
        int offo = -1;
        byte[] buffer = new byte[BUFFER_SIZE];
        while ((offo = zipInput.read(buffer)) != -1) {
          bos.write(buffer, 0, offo);
        }
        bos.close();
        // 获取 下一个文件
        entry = zipInput.getNextEntry();
      }
      zipInput.close();
      byteArray.close();
      // 退出jvm后删除临时文件
      file.deleteOnExit();
    } catch (Exception e) {
      throw new RuntimeException("解压流出现异常", e);
    }
    return file;
  }

  /**
   * @author: dongm @Description: base64转文件方法2
   * @date: 2021/12/13
   * @param: [base64, filePath]
   * @return: void
   */
  public static void base64ToFile(String base64, String filePath) {
    try {
      Files.write(
          Paths.get(filePath),
          java.util.Base64.getDecoder().decode(base64),
          StandardOpenOption.CREATE);
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
}

```
