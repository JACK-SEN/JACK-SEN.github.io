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

发送form表单数据

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

发送json数据

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



