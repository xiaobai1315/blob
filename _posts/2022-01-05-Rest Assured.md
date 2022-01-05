---
layout: post
title: "Rest Assured"
date:   2022-01-05
tags: [Rest Assured]
comments: true
author: xiaobai
---

Rest Assured

<!-- more -->

## 基本结构

given(): 配置请求参数、配置项
when():发起get post 请求，when 中的log 打印的是请求体的log
then(): 响应结果输出、断言, then 中的log 打印的是响应体的log



## 断言

log():打印日志，在when()代码块打印的是请求日志，在then()代码块中打印的是响应日志
```
public void test2() {
            given().
            when().
                log().headers(). // 请求体的header
                    log().body(). // 请求体的body
                get("https://www.baidu.com"). // 请求地址
            then()
                .log().body()    // 响应体body
                .log().headers()  // 响应体header
                .statusCode(200)  // 响应结果码断言
                .header("Content-Type", "text/html")  // 响应header断言
                .body("html.head.title", Matchers.equalTo("百度一下，你就知道"));  // 响应body断言
    }
```



## 设置请求参数

### get请求
get请求添加参数有两种方式，一种是直接在请求URL中拼接参数，另外一种是使用param()指定参数
```
public void test() {
given().
when().
		get("https://xxx.xxx.xxx?username=username").  // 直接在请求URL中拼接参数
then()
		.log().all()
		.statusCode(200);
}
```
```
 given().
 			param("username", "hogwords").  // 指定参数
 when().
 			get("https://xxxxxxxxx.com/get").
 then()
 			.log().all()
 			.statusCode(200);
    }
```

### post 请求
```
public void test2() {
given().
		queryParam("username", "hogwords").// 查询参数
		formParam("username", "hogwords").// 表单参数
when().
		post("https://xxxxxxxxx/post").
then()
		.log().all()
		.statusCode(200);
}
```



## 设置请求体

### json格式请求体
```
public void test2() {
String json = "{\"hello\": \"hogwords\"}";// json请求体
given().
		body(json).   // 设置请求体
		contentType("application/json"). // 设置请求体参数类型
		.log().headers()
		.log().body()
when().
		post("https://xxx/post").
then()
		.log().all()
		.statusCode(200);
}
```

### hashmap 格式请求体
```
HashMap<String, String> map = new HashMap<>();
map.put("hello", "hogwords");
given().
		body(map). // 设置请求体
		contentType("application/json"). // 设置请求体参数类型
		.log().headers()
		.log().body()
when().
		post("https://xxx/post").
then()
		.log().all()
		.statusCode(200);
```



## 获取响应结果

```
Response s = given()
		.header("hello", "word")
.when()
		.get("https://httpbin.ceshiren.com/get")
.then()
		.statusCode(200)
		.log().body()
		.extract().response();

String path = s.path("headers.Hello");  // 直接通过path读取
System.out.println(s);
System.out.println(s.jsonPath().getString("headers.Hello"));  // 转成JSONpath读取
```



## 设置代理

用户抓包数据

```
// 设置代理
RestAssured.proxy = host("127.0.0.1").withPort(8889);

// 忽略HTTPS校验
RestAssured.useRelaxedHTTPSValidation();

Response s = given()
		.headers("Header1", "value1", "header2", "value2")
.when()
		.get("https://httpbin.ceshiren.com/get")
.then()
		.statusCode(200)
		.log().body()
		.extract().response();
```



## 设置Cookie 

### 通过header设置

```
// 设置代理
RestAssured.proxy = host("127.0.0.1").withPort(8889);
// 忽略HTTPS校验
RestAssured.useRelaxedHTTPSValidation();

given()
		.headers("Cookie", "my_cookie=test")   // 通过header设置cookie
.when()
		.get("https://httpbin.ceshiren.com/get")
.then()
		.statusCode(200)
		.log().all();
```



### 通过Cookie()设置

```
// 设置代理
RestAssured.proxy = host("127.0.0.1").withPort(8889);

// 忽略HTTPS校验
RestAssured.useRelaxedHTTPSValidation();

given()
		.cookies("my_cookie", "test", "my_cookie2", "test2")  // 通过cookie 设置cookie
.when()
		.get("https://httpbin.ceshiren.com/get")
.then()
		.statusCode(200)
		.log().all();
```

## Form请求

```
// 设置代理
RestAssured.proxy = host("127.0.0.1").withPort(8889);

// 忽略HTTPS校验
RestAssured.useRelaxedHTTPSValidation();

given()
    .formParam("Param1", "value1")
.when()
    .post("https://httpbin.ceshiren.com/post")  单条form请求
.then()
    .statusCode(200)
    .log().all();
    
given()
    .formParams("Param1", "value1", "Param2", "value2")  // 发送多条form请求
.when()
    .post("https://httpbin.ceshiren.com/post")
.then()
    .statusCode(200)
    .log().all();
```

## 设置超时时间
设置超时时间，防止失败的用例阻塞其他的用例执行
3个测试用例，第二个用例会延迟10S返回，设置超时时间为3S， 3S后返回超时，然后继续执行剩下的用例
```
public class TestRestAssured {

    @BeforeAll
    static void before() {
    	// 设置请求的根路径
        RestAssured.baseURI = "https://httpbin.ceshiren.com";
    }

    @Test
    public void test1() {
        given()
                .when()
                .get("/get")
                .then()
                .statusCode(200);
    }

    @Test
    public void test2() {

	// 设置请求的超时时间
        HttpClientConfig httpClientConfig = HttpClientConfig.httpClientConfig().setParam("http.socket.timeout", 3000);
        RestAssuredConfig restAssuredConfig = RestAssuredConfig.config().httpClient(httpClientConfig);

        given()
            .config(restAssuredConfig)
        .when()
            .get("/delay/10")
        .then()
            .statusCode(200);
    }

    @Test
    public void test3() {
        given()
                .when()
                .get("/get")
                .then()
                .statusCode(200);
    }
}

```
