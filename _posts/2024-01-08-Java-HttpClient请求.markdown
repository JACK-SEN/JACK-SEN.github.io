---
layout: post
title:  "JAVA-发送HTTP client 请求"
datetime:   2024-01-08 10:27:26
categories: Java
---
[TOC]

# 发送HTTP请求

## HttpURLConnection

### GET

```java
public static String sendRestApi(String apiUrl) {
    try {
        URL url = new URL(apiUrl);
        HttpURLConnection connection = (HttpURLConnection) url.openConnection();
        connection.setRequestMethod("GET");
        BufferedReader reader = new BufferedReader(new InputStreamReader(connection.getInputStream()));
        String line;
        StringBuilder response = new StringBuilder();
        while ((line = reader.readLine()) != null) {
            response.append(line);
        }
        reader.close();
        String jsonString = response.toString();
        log.info("高德restApi请求结果:{}",jsonString);
        return jsonString;
    } catch (IOException e) {
        e.printStackTrace();
    }
    return null;
}
//普通请求，直接传url

//带有参数时
String deptCode = "015";
Date beginTime = new SimpleDateFormat("yyyy-MM-dd").parse("2023-01-01");
Date endTime = new SimpleDateFormat("yyyy-MM-dd").parse("2024-03-01");
String encodedDeptCode = URLEncoder.encode(deptCode, "UTF-8");
String encodedBeginTime = URLEncoder.encode(new SimpleDateFormat("yyyy-MM-dd").format(beginTime), "UTF-8");
String encodedEndTime = URLEncoder.encode(new SimpleDateFormat("yyyy-MM-dd").format(endTime), "UTF-8");
//通过占位符将将参数传递，形成api+param的方式
String param = String.format("/dptUtilizationRateController/getUtilizationRateList?deptCode=%s&beginTime=%s&endTime=%s",encodedDeptCode, encodedBeginTime, encodedEndTime);
String url = "http://xxxx:xxxx";
String jsonString = DptHttpClientUtils.sendRestApi(url+param);
```

### POST

```java
public static void main(String[] args) throws Exception {
        String url = "https://example.com/api";
        URL obj = new URL(url);
        HttpURLConnection connection = (HttpURLConnection) obj.openConnection();
        // 设置请求方法为POST
        connection.setRequestMethod("POST");
        // 设置请求头参数
        connection.setRequestProperty("Content-Type", "application/json");
        connection.setRequestProperty("User-Agent", "Mozilla/5.0");
        // 启用输入输出流
        connection.setDoOutput(true);
        // 构建请求体数据
        String postData = "{\"key1\":\"value1\",\"key2\":\"value2\"}";
        // 获取输出流并写入请求体数据
        try (DataOutputStream wr = new DataOutputStream(connection.getOutputStream())) {
            wr.writeBytes(postData);
            wr.flush();
        }
        // 获取响应码
        int responseCode = connection.getResponseCode();
        System.out.println("Response Code: " + responseCode);
        // 读取响应内容
        try (BufferedReader in = new BufferedReader(new InputStreamReader(connection.getInputStream()))) {
            String inputLine;
            StringBuilder response = new StringBuilder();

            while ((inputLine = in.readLine()) != null) {
                response.append(inputLine);
            }
            // 打印响应内容
            System.out.println(response.toString());
        }
    }
```

## HttpClients

### GET

#### 发送form表单数据

```java
 public static JSONObject sendGet(String url, List<NameValuePair> params) {
        try (CloseableHttpClient httpClient = HttpClients.createDefault()) {
            URIBuilder builder = new URIBuilder(url);
            builder.setParameters(params);
            HttpGet httpGet = new HttpGet(builder.build());
            httpGet.setHeader(HttpHeaders.CONTENT_TYPE, "application/x-www-form-urlencoded;charset=utf-8");
            httpGet.setHeader(HttpHeaders.ACCEPT, "*/*;charset=utf-8");
            try (CloseableHttpResponse response = httpClient.execute(httpGet)) {
                HttpEntity responseEntity = response.getEntity();
                String responseStr = EntityUtils.toString(responseEntity);
                return JSON.parseObject(responseStr);
            }
        } catch (Exception e) {
            log.error("sendGet发送数据失败:{}", e.getMessage());
        }
        return null;
    }
```

#### GET发送 Https请求

```java
public static String sendGetFormData(String sDate) {
    String url = ResourceBundle.getBundle("site").getString("url");
    String unitType = ResourceBundle.getBundle("site").getString("unitType");
    String unitCode = ResourceBundle.getBundle("site").getString("unitCode");
    String apiUrl = url + "?" + "unitType=" + unitType + "&unitCode=" + unitCode + "&sdate=" + sDate;
    try {
        CloseableHttpClient httpClient = HttpClients.custom()
            .setSSLHostnameVerifier(NoopHostnameVerifier.INSTANCE)
            .setSSLContext(SSLContextBuilder.create().loadTrustMaterial((chain, authType) -> true).build())
            .build();
        HttpGet httpGet = new HttpGet(apiUrl);
        HttpResponse response = httpClient.execute(httpGet);
        String json = EntityUtils.toString(response.getEntity());
        httpClient.close();
        return json;
    } catch (Exception e) {
        LOGGER.error("同步车次站点数据失败:{}", e.getMessage());
    }
    return null;
}
```



### POST

发送form表数据

```java
public static JSONObject sendPostParam(String url, List<NameValuePair> params) {
        try (CloseableHttpClient httpClient = HttpClients.createDefault()) {
            HttpPost httpPost = new HttpPost(url);
            httpPost.setHeader(HttpHeaders.CONTENT_TYPE,"application/x-www-form-urlencoded;charset=utf-8");
            StringEntity entity = new UrlEncodedFormEntity(params, StandardCharsets.UTF_8);
            httpPost.setEntity(entity);
            httpPost.setHeader(HttpHeaders.ACCEPT,"*/*;charset=utf-8");
            try (CloseableHttpResponse response = httpClient.execute(httpPost)) {
                HttpEntity responseEntity = response.getEntity();
                if (responseEntity != null) {
                    String responseStr = EntityUtils.toString(responseEntity);
                    return JSON.parseObject(responseStr);
                }
            }
        } catch (Exception e) {
            log.error("sendPostParam发送数据失败:{}", e.getMessage());
        }
        return null;
    }
```

#### 发送json数据

```java
public static String sendPostJson(String url, String json) {
        try (CloseableHttpClient httpClient = HttpClients.createDefault()) {
            HttpPost httpPost = new HttpPost(url);
            httpPost.setHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_UTF8_VALUE);
            StringEntity requestEntity = new StringEntity(json, ContentType.APPLICATION_JSON);
            httpPost.setEntity(requestEntity);
            try (CloseableHttpResponse response = httpClient.execute(httpPost)) {
                HttpEntity responseEntity = response.getEntity();
                if (responseEntity != null) {
                    return EntityUtils.toString(responseEntity);
                }
            }
        } catch (Exception e) {
            log.error("sendPostJson发送数据失败:{}", e.getMessage());
        }
        return null;
    }
```

#### Post发送Form-data

```java
//发送图片文件  和普通form-data数据
public static void multipartPost(String url,File file, Map<String,String> map){
    try (CloseableHttpClient httpClient = HttpClients.createDefault()){
        HttpPost httpPost = new HttpPost(url);
        MultipartEntityBuilder builder = MultipartEntityBuilder.create();
        builder.setCharset(StandardCharsets.UTF_8);
        builder.addBinaryBody("file",file,ContentType.DEFAULT_BINARY,file.getName());
        Set<Map.Entry<String, String>> entrySet = map.entrySet();
        for (Map.Entry<String, String> entry : entrySet) {
            builder.addTextBody(entry.getKey(), entry.getValue());
        }
        HttpEntity httpEntity = builder.build();
        httpPost.setEntity(httpEntity);
        try (CloseableHttpResponse response = httpClient.execute(httpPost)) {
            HttpEntity responseEntity = response.getEntity();
            if (responseEntity != null) {
                String responseStr = EntityUtils.toString(responseEntity);
                JSONObject jsonObject = JSON.parseObject(responseStr);
                System.out.println(jsonObject);
            }
        }
    }catch (Exception e){
        log.error("multipartPost发送数据失败:{}", e.getMessage());
    }
}
```

#### 发送soap数据

```java
package com.ydzb.dzjb.util;

import cn.hutool.core.util.XmlUtil;
import lombok.extern.slf4j.Slf4j;
import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;
import org.w3c.dom.Document;
import org.w3c.dom.Node;
import org.w3c.dom.NodeList;
import java.nio.charset.StandardCharsets;

@Slf4j
public class SoapUtils {
    /**
     * 适用于单标签结果数据
     * */
    public static String parseXml(String xmlContent,String targetName) {
        Document document = XmlUtil.parseXml(xmlContent);
        NodeList refreshDictResultNode = document.getElementsByTagName(targetName);
        Node item = refreshDictResultNode.item(0);
        String refreshDictResult = item.getTextContent();
        if (refreshDictResult != null && !refreshDictResult.trim().isEmpty()) {
            return refreshDictResult;
        }
        return null;
    }
    /**
     * 通用调用
     * */
    public static String recognition(String url,String body){
        try (CloseableHttpClient httpClient = HttpClients.createDefault()) {
            HttpPost httpPost = new HttpPost(url);
            httpPost.setHeader("Content-Type", "text/xml;charset=utf-8");
            httpPost.setEntity(new StringEntity(body, StandardCharsets.UTF_8));
            log.info("请求报文:{}",body);
            try (CloseableHttpResponse response = httpClient.execute(httpPost)) {
                HttpEntity responseEntity = response.getEntity();
                if (responseEntity != null) {
                    String result = EntityUtils.toString(responseEntity, "UTF-8");
                    log.info("响应结果:{}",result);
                    return result;
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
package com.ydzb.dzjb.util;
import lombok.extern.slf4j.Slf4j;
/**
 * soap报文获取
 * */
@Slf4j
public class SoapBodyUtils {
    /**
     * 5T运用率报文
     * */
    public static String getUtilizationRateSoap(String beginTime,String endTime,String deptCode,String accessId){
        return "<soap:Envelope xmlns:soap=\"http://www.w3.org/2003/05/soap-envelope\" xmlns:tem=\"http://tempuri.org/\">\n"
                + "   <soap:Header/>\n"
                + "   <soap:Body>\n"
                + "      <tem:GetUtilizationRateList>\n"
                + "         <tem:beginTime>" + beginTime + "</tem:beginTime>\n"
                + "         <tem:endTime>" + endTime + "</tem:endTime>\n"
                + "         <tem:deptCode>" + deptCode + "</tem:deptCode>\n"
                + "         <tem:AccessID>" + accessId + "</tem:AccessID>\n"
                + "      </tem:GetUtilizationRateList>\n"
                + "   </soap:Body>\n"
                + "</soap:Envelope>";
    }
}
//调用示例
@GetMapping("/list")
public Object list() {
    try {
        // 请求参数
        String url = CommonUtils.getConfigValue("utilizationRateUrl");
        String deptCode = CommonUtils.getConfigValue("deptCode");
        String beginTime = CommonUtils.getConfigValue("beginTime");
        String endTime = CommonUtils.getConfigValue("endTime");
        String accessId = CommonUtils.getConfigValue("accessId");
        String xmlContent = SoapUtils.recognition(url, SoapBodyUtils.getUtilizationRateSoap(beginTime, endTime, deptCode, accessId));
        String jsonString = SoapUtils.parseXml(xmlContent, "GetUtilizationRateListResult");
        return JSON.parseArray(jsonString, UtilizationRate.class);
    } catch (Exception e) {
        return e.getMessage();
    }
}

```

完成案例备份

```java
package com.ydzb.dzjb.util;

import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;
import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.w3c.dom.Node;
import org.w3c.dom.NodeList;
import org.xml.sax.InputSource;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import java.io.*;
import java.util.ResourceBundle;

/**
 * @author lth
 * @date 2024/3/14
 */
public class UtilizationRateUtils {

    /**
     * 5T运用率接口调用
     * */
    public static String getUtilizationData(){
        // 请求参数
        String wsUrl = CommonUtils.getConfigValue("utilizationRateUrl");
        String deptCode = CommonUtils.getConfigValue("deptCode");
        String beginTime = CommonUtils.getConfigValue("beginTime");
        String endTime = CommonUtils.getConfigValue("endTime");
        String accessId = CommonUtils.getConfigValue("accessId");
        // 构建SOAP请求
        String soapRequest = "<soap:Envelope xmlns:soap=\"http://www.w3.org/2003/05/soap-envelope\" xmlns:tem=\"http://tempuri.org/\">\n"
                + "   <soap:Header/>\n"
                + "   <soap:Body>\n"
                + "      <tem:GetUtilizationRateList>\n"
                + "         <tem:beginTime>" + beginTime + "</tem:beginTime>\n"
                + "         <tem:endTime>" + endTime + "</tem:endTime>\n"
                + "         <tem:deptCode>" + deptCode + "</tem:deptCode>\n"
                + "         <tem:AccessID>" + accessId + "</tem:AccessID>\n"
                + "      </tem:GetUtilizationRateList>\n"
                + "   </soap:Body>\n"
                + "</soap:Envelope>";
        try (CloseableHttpClient httpClient = HttpClients.createDefault()) {
            HttpPost httpPost = new HttpPost(wsUrl);
            httpPost.setHeader("Content-Type", "text/xml");
            httpPost.setEntity(new StringEntity(soapRequest, "UTF-8"));
            CloseableHttpResponse response = httpClient.execute(httpPost);
            if (response.getStatusLine().getStatusCode() == 200) {
                System.out.println("请求成功，开始获取响应内容");
                String responseString = EntityUtils.toString(response.getEntity());
                DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
                DocumentBuilder builder = factory.newDocumentBuilder();
                Document doc = builder.parse(new InputSource(new StringReader(responseString)));
                NodeList nodeList = doc.getElementsByTagName("GetUtilizationRateListResult");
                if (nodeList.getLength() > 0) {
                    Node item = nodeList.item(0);
                    Element element = (Element) item;
                    String textContent = element.getTextContent();
                    System.out.println("GetUtilizationRateListResult 内容：" + textContent);
                    return textContent;
                }
            } else {
                throw new RuntimeException("请求失败：" + response.getStatusLine().getStatusCode());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}

```





## 公共抽取

```java
public static String sendPostFormData(String url, List<NameValuePair> nameValuePairList) {
    try (CloseableHttpClient client = HttpClients.createDefault()) {
        HttpPost httpPost = createHttpPost(url, nameValuePairList);
        try (CloseableHttpResponse response = client.execute(httpPost)) {
            return EntityUtils.toString(response.getEntity(), StandardCharsets.UTF_8);
        }
    } catch (IOException e) {
        LOGGER.error("获取天气信息接口失败. Exception: {}", e.getMessage());
        return null;
    }
}

private static HttpPost createHttpPost(String url, List<NameValuePair> nameValuePairList) {
    HttpPost httpPost = new HttpPost(url);
    RequestConfig requestConfig = RequestConfig.custom()
        .setSocketTimeout(5000)
        .setConnectTimeout(5000)
        .build();
    httpPost.setConfig(requestConfig);
    try {
        StringEntity entity = new UrlEncodedFormEntity(nameValuePairList, StandardCharsets.UTF_8);
        httpPost.setHeader(HttpHeaders.CONTENT_TYPE, "application/x-www-form-urlencoded;charset=utf-8");
        httpPost.setEntity(entity);
        httpPost.setHeader(HttpHeaders.ACCEPT, "*/*;charset=utf-8");
        httpPost.setHeader("requestId", IdUtil.fastSimpleUUID());
        httpPost.setHeader("tokenId", IdUtil.fastSimpleUUID());
    } catch (Exception e) {
        LOGGER.error("创建HTTP POST请求失败. Exception: {}", e.getMessage());
    }
    return httpPost;
}
```

