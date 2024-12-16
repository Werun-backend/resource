# 	JAVA爬虫

## 一**、HTTP协议**

### 1、简介

HTTP（超文本传输协议，HyperText Transfer Protocol）是一种用于在Web上传输数据的协议。它是Web的基础，定义了客户端（通常是浏览器）和服务器之间通信的规则。HTTP是无状态的协议，这意味着每个请求都是独立的，与之前或之后的请求无关。

### 2、请求方法

- **GET**：请求资源，通常通过 URL 查询参数传递。
- **POST**：提交数据（如表单、文件），通过请求体传递。
- **PUT**：更新资源，通过请求体传递新数据。
- **DELETE**：删除资源，通常通过 URL 传递标识符。
- **PATCH**：部分更新资源，通过请求体传递部分数据。
- **HEAD**：请求资源的元数据，不返回内容。
- **OPTIONS**：查询服务器支持的 HTTP 方法或功能。
- **TRACE**：调试工具，回显服务器请求。
- **CONNECT**：建立代理隧道，通常用于 SSL/TLS。

### 3、状态码

#### 1xx：信息性状态码

> - **100 Continue**：初步请求已接收，客户端可以继续发送请求的其余部分。
> - **101 Switching Protocols**：服务器同意客户端的请求，切换到其他协议（例如，HTTP/1.1切换到WebSocket）。

#### 2xx：成功状态码

> - **200 OK**：请求成功，服务器返回所请求的数据。
> - **201 Created**：请求成功并导致资源的创建，通常在使用POST方法时返回。
> - **202 Accepted**：请求已接受，但尚未处理。
> - **204 No Content**：请求成功，但不返回任何内容。
> - **205 Reset Content**：请求成功，但要求客户端重置视图。
> - **206 Partial Content**：服务器成功处理部分GET请求，通常用于分块下载。

#### 3xx：重定向状态码

> - **300 Multiple Choices**：请求的资源有多种可用选项，客户端需要选择。
> - **301 Moved Permanently**：请求的资源已永久移动到新位置，新的URL会在响应中提供。
> - **302 Found**（临时重定向）：请求的资源临时移动到另一个位置，新的URL在响应中提供。
> - **303 See Other**：响应的资源在不同的URI下可以找到，使用GET方法访问。
> - **304 Not Modified**：客户端的缓存副本是最新的，服务器未提供新的内容。
> - **305 Use Proxy**：请求的资源必须通过代理访问（现已不常用）。
> - **307 Temporary Redirect**：请求的资源临时移动到新位置，但客户端应使用原始请求方法。
> - **308 Permanent Redirect**：请求的资源永久移动到新位置，但客户端应使用原始请求方法。

#### 4xx：客户端错误状态码

> - **400 Bad Request**：请求有误，服务器无法理解。
> - **401 Unauthorized**：请求需要身份验证，未提供或提供的凭证无效。
> - **402 Payment Required**：保留状态码，通常用于需要支付的请求（未广泛使用）。
> - **403 Forbidden**：服务器理解请求，但拒绝执行，通常因为权限不足。
> - **404 Not Found**：请求的资源未找到。
> - **405 Method Not Allowed**：请求方法不被允许，服务器不支持所请求的方法。
> - **406 Not Acceptable**：请求的资源无法根据Accept头部的内容进行返回。
> - **407 Proxy Authentication Required**：需要通过代理进行身份验证。
> - **408 Request Timeout**：请求超时，服务器未能在指定时间内收到完整请求。
> - **409 Conflict**：请求与服务器的当前状态冲突。
> - **410 Gone**：请求的资源已被永久删除，且没有可用的转发地址。
> - **411 Length Required**：服务器要求请求中必须包含Content-Length头部。
> - **412 Precondition Failed**：请求中包含的前置条件未满足。
> - **413 Payload Too Large**：请求实体过大，服务器无法处理。
> - **414 URI Too Long**：请求的URI过长，服务器无法处理。
> - **415 Unsupported Media Type**：请求的媒体类型不被服务器支持。
> - **416 Range Not Satisfiable**：请求的范围不在可用的范围内。
> - **417 Expectation Failed**：服务器无法满足Expect头部的要求。

#### 5xx：服务器错误状态码

> - **500 Internal Server Error**：服务器遇到意外情况，无法完成请求。
> - **501 Not Implemented**：服务器不支持请求的功能，无法完成请求。
> - **502 Bad Gateway**：作为网关或代理的服务器收到无效响应。
> - **503 Service Unavailable**：服务器当前无法处理请求，通常由于过载或维护。
> - **504 Gateway Timeout**：作为网关或代理的服务器未能及时从上游服务器获取请求。

## 二、**JSOUP获取网页信息**

###  **Connection**

`Connection` 是 Jsoup 用于处理 HTTP 连接的接口。它提供了一系列的方法来构建和发送 HTTP 请求。可以通过 `Jsoup.connect()` 方法创建一个 `Connection` 对象，然后设置请求的 URL 和各种请求参数。

- `url(String url)`：设置请求的 URL。
- `method(Method method)`：设置请求方法（GET 或 POST）。
- `header(String name, String value)`：设置请求头。
- `timeout(int millis)`：设置请求超时时间。
- `execute()`：发送请求并返回 `Response` 对象。
- `get()`发送请求并返回 `Document` 对象。

###  **Response**

`Response` 类表示从 HTTP 连接中获取的响应。它包含了服务器返回的数据和元信息，比如响应的 HTML 内容、状态码、响应头等。

- `body()`：获取响应体的内容（HTML 文档）。
- `statusCode()`：获取响应状态码（例如 200、404）。
- `header(String name)`：获取特定响应头的值。
- `contentType()`：获取响应的内容类型。

### **Document** 

`Document` 类是 Jsoup 的核心对象，代表了 HTML 页面中整个文档结构。它是从服务器返回的 HTML 内容的解析结果，并可以被用来：

- 提取网页中的数据（例如获取元素、文本等）
- 修改 HTML 内容（例如修改元素、属性等）
- 进行 DOM 操作（例如添加、删除、替换节点等）

### **Element**

- `Element`：元素对象。一个 Document 中可以着包含着多个 Element 对象，可以使用 Element 对象来遍历节点提取数据或者直接操作HTML。
- `Elements`：元素对象集合，类似于`List<Element>`

### 1、导入依赖

```xml
<dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.11.3</version>
</dependency>
```

### 2、请求url

#### 2.1 方法一

```java
//创建连接
Connection connect = Jsoup.connect("https://searchcustomerexperience.techtarget.com/info/news");
//请求网页
Document document = connect.get();
//输出HTML
System.out.println(document.html());
```

#### 2.2 方法二

```java
//获取响应
Response response = Jsoup.connect("https://searchcustomerexperience.techtarget.com/info/news")
       .method(Method.GET).execute();
URL url = response.url();   //查看请求的URL
System.out.println("请求的URL为:" + url);
int statusCode = response.statusCode();  //获取响应状态码
System.out.println("响应状态码为:" + statusCode);  
String contentType = response.contentType();
System.out.println("响应类型为:" + contentType);  //获取响应数据类型
String statusMessage = response.statusMessage();  //响应信息 200-OK
System.out.println("响应信息为:" + statusMessage);
//判断响应状态码是否为200
if (statusCode == 200) {
    String html = new String(response.bodyAsBytes(),"gbk");  //通过这种方式可以获得响应的HTML文件
    Document document = response.parse();   //获取html内容,但对应的是Document类型
    System.out.println(html);  //这里html和document数据是一样的，但document是经过格式化的
}
```

### 3、设置请求头

```java
Connection header(String var1, String var2);

Connection headers(Map<String, String> var1);
```

#### 3.1设置单个请求头

```java
Connection connect = Jsoup.connect("https://searchcustomerexperience.techtarget.com/info/news");
//设置单个请求头
Connection conheader = connect.header("User-Agent","Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.108 Safari/537.36");
Document document = conheader.get();
System.out.println(document);
```

#### 3.2设置多个请求头

```java
Connection connect = Jsoup.connect("https://searchcustomerexperience.techtarget.com/info/news");
//设置多个请求头。头信息保存到Map集合中
Map<String, String> header = new HashMap<String, String>();
header.put("Host", "searchcustomerexperience.techtarget.com");
header.put("User-Agent", " Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.108 Safari/537.36");
header.put("Accept", "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8");
header.put("Accept-Language", "zh-cn,zh;q=0.5");
header.put("Accept-Encoding", "gzip, deflate");
header.put("Cache-Control", "max-age=0");
header.put("Connection", "keep-alive");
Connection conheader = connect.headers(header);
Document document = conheader.get();
System.out.println(document);
```

#### 3.3添加User-Agent库和Refer库

```java
public static void main(String[] args) throws IOException {
    Connection connect = Jsoup.connect("https://searchcustomerexperience.techtarget.com/info/news");
    //实例化静态类
    Builder builder = new Builder();
    //请求网页不同添加host,也可以不设置
    builder.host = "searchcustomerexperience.techtarget.com";
    //builder中的信息添加到Map集合中
    Map<String, String> header = new HashMap<String, String>();
    header.put("Host", builder.host);
    header.put("User-Agent", 
          builder.userAgentList.get(new Random().nextInt(builder.userAgentSize)) );
    header.put("Accept", builder.accept);
    header.put("Referer", builder.refererList.get(new Random().nextInt(builder.refererSize)));
    header.put("Accept-Language", builder.acceptLanguage);
    header.put("Accept-Encoding", builder.acceptEncoding);
    //设置头
    Connection conheader = connect.headers(header);
    Document document = conheader.get();  //发送GET请求
    System.out.println(document);  //输出HTML
}
/**
 * 封装请求头信息的静态类
 */
static class Builder {
    //设置userAgent库;读者根据需求添加更多userAgent
    String[] userAgentStrs = {"Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_6_8; en-us) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50",
          "Mozilla/5.0 (Windows; U; Windows NT 6.1; en-us) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50",
          "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:38.0) Gecko/20100101 Firefox/38.0",
          "Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; .NET4.0C; .NET4.0E; .NET CLR 2.0.50727; .NET CLR 3.0.30729; .NET CLR 3.5.30729; InfoPath.3; rv:11.0) like Gecko",
          "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)",
          "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0)",
          "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.0)"};
    List<String> userAgentList = Arrays.asList(userAgentStrs);
    int userAgentSize = userAgentList.size();
    //设置referer库;读者根据需求添加更多referer
    String[] refererStrs = {"https://www.baidu.com/",
          "https://www.sogou.com/",
          "http://www.bing.com",
          "https://www.so.com/"};
    List<String> refererList = Arrays.asList(refererStrs);
    int refererSize = refererList.size();
    //设置Accept、Accept-Language以及Accept-Encoding
    String accept = "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8";
    String acceptLanguage = "zh-cn,zh;q=0.5";
    String acceptEncoding = "gzip, deflate";
    String host;
}
```

### 4、设置请求参数

Jsoup提供了多种添加请求参数的方法

```java
Connection data(String var1, String var2);

Connection data(String var1, String var2, InputStream var3);

Connection data(String var1, String var2, InputStream var3, String var4);

Connection data(Collection<KeyVal> var1);

Connection data(Map<String, String> var1);

Connection data(String... var1);
```

```java
Connection connect = Jsoup.connect("http://www.*****.com/ems.php");
//添加参数
connect.data("wen","EH629625211CS").data("action", "ajax");
Response response = connect.method(Method.GET).ignoreContentType(true).execute();  
//获取数据,处理成HTML
Document document = response.parse();
System.out.println(document);
```

```java
Connection connect = Jsoup.connect("http://www.*****.com/ems.php");
//需要提交的参数
Map<String, String> data = new HashMap<String, String>();  
   data.put("wen", "EH629625211CS");  
   data.put("action", "ajax");  
   //获取响应
   Response response = connect.data(data).method(Method.GET).ignoreContentType(true).execute();  
   //获取数据,处理成HTML
   Document document = response.parse();
   System.out.println(document);
```

```java
Connection connect = Jsoup.connect("http://www.*****.com/ems.php");
//添加参数
connect.data("wen", "EH629625211CS", "action", "ajax");
Response response = connect.method(Method.GET).ignoreContentType(true).execute();  
//获取数据,处理成HTML
Document document = response.parse();
System.out.println(document);
```

### 5、超时设置

```java
//基于timeout设置超时时间
Response response = Jsoup.connect("https://twitter.com/")
       .method(Method.GET).timeout(3*1000).execute();
Document document = Jsoup.connect("https://twitter.com/").timeout(10*1000)
       .header("User-Agent","Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0")
        .get();
```

### 6、代理服务器

```java
//设置代理
Connection connection = Jsoup.connect("https://searchcustomerexperience.techtarget.com/info/news")
       .proxy("163.204.240.107",9999);
Response response = connection.method(Method.GET).timeout(20*1000).execute();
//获取响应状态码
int statusCode = response.statusCode();  
System.out.println("响应状态码为:" + statusCode);  
```

### 7、响应转输出流

需要用Connection ignoreContentType(boolean var1);设置为ignoreContentType(true)，否则会报错

```java
public static void main(String[] args) throws IOException {
    String imageUrl = "https://www.makro.co.za/sys-master/images/h98/h64/9152530710558/06cf39e4-7e43-42d4-ab30-72c81ab0e941-qpn13_medium";
    Connection connect = Jsoup.connect(imageUrl);
    Response response = connect.method(Method.GET).ignoreContentType(true).execute();  
    System.out.println("文件类型为:" + response.contentType());
    //如果响应成功，则执行下面的操作
    if (response.statusCode() ==200) {
       //响应转化成输出流
       BufferedInputStream bufferedInputStream = response.bodyStream();
       //保存图片
       saveImage(bufferedInputStream,"image/1.jpg");
    }
}

/**
    * 保存图片操作
    * @param  输入流
    * @param  保存的文件目录
 * @throws IOException
    */
static void saveImage(BufferedInputStream inputStream, String savePath) throws IOException  {
    
    byte[] buffer = new byte[1024];
    int len = 0;
    //创建缓冲流
    FileOutputStream fileOutStream = new FileOutputStream(new File(savePath));
    BufferedOutputStream bufferedOut = new BufferedOutputStream(fileOutStream);
    //图片写入
    while ((len = inputStream.read(buffer, 0, 1024)) != -1) {
       bufferedOut.write(buffer, 0, len);
    }
    //缓冲流释放与关闭
    bufferedOut.flush();
    bufferedOut.close();
}
```

### 8、通过https请求认证

- HTTPS（Hypertext Transfer Protocol Secure）是安全的 HTTP。它通过在 HTTP 的基础上添加 SSL/TLS（安全套接层/传输层安全）协议来确保数据传输的安全性。
- HTTPS 提供了数据加密、身份验证和数据完整性。
- 通过 SSL/TLS 加密层，数据在客户端和服务器之间进行加密传输。其工作过程通常包括以下步骤：
  1. **SSL/TLS 握手**：客户端和服务器通过一个握手过程建立安全连接，验证服务器的身份，并协商加密算法和密钥。
  2. **数据加密**：一旦握手完成，后续的数据都通过加密通道进行传输。
  3. **数据解密**：接收方接收到加密的数据后，将其解密并处理。

#### 8.1通过Jsoup提供的方法

初始化一个接受所有 SSL 证书的 `SSLSocketFactory`，通过实现一个不检查证书的 `TrustManager`

```java
private static synchronized void initUnSecureTSL() throws IOException {
    if (sslSocketFactory == null) {
        TrustManager[] trustAllCerts = new TrustManager[]{new X509TrustManager() {
            public void checkClientTrusted(X509Certificate[] chain, String authType) {
            }
            public void checkServerTrusted(X509Certificate[] chain, String authType) {
            }
            public X509Certificate[] getAcceptedIssuers() {
                return null;
            }
        }};
        try {
            SSLContext sslContext = SSLContext.getInstance("SSL");
            sslContext.init((KeyManager[])null, trustAllCerts, new SecureRandom());
            sslSocketFactory = sslContext.getSocketFactory();
        } catch (KeyManagementException | NoSuchAlgorithmException var3) {
            throw new IOException("Can't create unsecure trust manager");
        }
    }
}
```

该方法在构建connect时当req.validateTLSCertificates()为FALSE时才会调用，req.validateTLSCertificates()返回的是静态类Requst中的private boolean validateTSLCertificates成员变量，所以只需在构建时将validateTSLCertificates修改为FLASE即可通过调用initUnSecureTSL()通过Https验证，具体代码如下



```java
public static void main(String[] args) throws IOException {
    Connection connect = Jsoup.connect("https://cn.kompass.com/a/hospitality-tourism-hotel-and-catering-industries/78/")
          .validateTLSCertificates(false);
    Document document = connect.get();
    System.out.println(document);
}
```

#### 8.2通过手写创建信任管理器

```java
public static void main(String[] args) throws IOException {
    initUnSecureTSL();
    String url = "https://cn.kompass.com/a/hospitality-tourism-hotel-and-catering-industries/78/";
    //创建连接
    Connection connect = Jsoup.connect(url);
    //请求网页
    Document document = connect.get();
    //输出HTML
    System.out.println(document.html());
}
private static void initUnSecureTSL()  {
    // 创建信任管理器(不验证证书)
    final TrustManager[] trustAllCerts = new TrustManager[]{new X509TrustManager() {
       //检查客户端证书
       public void checkClientTrusted(final X509Certificate[] chain, final String authType) {
          //do nothing 接受任意客户端证书
       }
       //检查服务器端证书  
       public void checkServerTrusted(final X509Certificate[] chain, final String authType) {
          //do nothing  接受任意服务端证书
       }
       //返回受信任的X509证书
       public X509Certificate[] getAcceptedIssuers() {
          return null; //或者return new X509Certificate[0];
       }
    }};
    try {
       // 创建SSLContext对象,并使用指定的信任管理器初始化
       SSLContext sslContext = SSLContext.getInstance("SSL");
       sslContext.init(null, trustAllCerts, new java.security.SecureRandom());
       ////基于信任管理器，创建套接字工厂 (ssl socket factory)
       SSLSocketFactory sslSocketFactory = sslContext.getSocketFactory();
       //给HttpsURLConnection配置SSLSocketFactory
       HttpsURLConnection.setDefaultSSLSocketFactory(sslSocketFactory);
    } catch (Exception e) {
       e.printStackTrace();
    }
```

### 9、大文件处理

Jsoup默认只能获取1M文件，若获取大文件要自定义上限，并注意延长响应时间

```java
//如果不设置maxBodySize,会导致网页不全
String url = "http://poi.mapbar.com/shanghai/F10";
Response response = Jsoup.connect(url).timeout(10*10*1000).maxBodySize(Integer.MAX_VALUE)
       .method(Method.GET).ignoreContentType(true).execute();
System.out.println(response.parse());
```

## 三、**HTTPClient获取网页信息**

### 1、导入依赖

```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.5</version>
</dependency>
```

### 2、请求url

#### 2.1创建HttpClient实例

在使用Httpclient前要先实例化HttpClient

有六种方法，第一种不推荐再使用

```java
HttpClient httpClient1 = new DefaultHttpClient();  
HttpClient httpClient2 = HttpClients.custom().build(); 
HttpClient httpClient3 = HttpClientBuilder.create().build();
CloseableHttpClient httpClient4 = HttpClients.createDefault(); 
HttpClient httpClient5 = HttpClients.createSystem();
HttpClient httpClient6 = HttpClients.createMinimal();
```

#### 2. 2创建请求方法实例

在HttpClient中，支持所有HTTP方法，即GET,POST,HEAD,PUT,DELETE,OPINIONS和TEACE，他们继承于HttpRequestBase

其中每一种方法都对应一个类，即HttpGet,HttpPost,HttpHead等

这些类的实例化方式各有三种

```java
public HttpGet() {
}

public HttpGet(URI uri) {
    this.setURI(uri);
}

public HttpGet(String uri) {
    this.setURI(URI.create(uri));
}
```

使用第一种时要设置请求URL

使用后两种参数分别为统一资源标识符URI和字符串类型的URI

```java
URI uri = new URIBuilder("https://searchcustomerexperience.techtarget.com/info/news").build();  //创建URI
HttpGet getMethod = new HttpGet();  //  get方法请求
getMethod.setURI(uri);  //设置
```

#### 2.3执行请求

基于实例化的HttpClient，可以调用HttpResponse execute(HttpUriRequest Request)执行数据请求，返回类型为HttpResponse

```java
HttpClient httpClient = HttpClients.custom().build();
HttpGet httpGet = new HttpGet("http://www.baidu.com");
HttpResponse httpResponse = httpClient.execute(httpGet);//try-catch包围
```

也可以使用HttpContext提供Http上下文环境

```java
//初始化HttpContext
HttpContext localContext = new BasicHttpContext();
httpResponse = httpClient.execute(httpGet,localContext);//try-catch包围
```

### 3、获取响应信息

使用获取到的HttpResponse可以继续执行一些方法获取相应的信息

```java
httpResponse = httpClient.execute(httpGet);
//获取具体响应信息
System.out.println("response:" + httpResponse );
String status = httpResponse .getStatusLine().toString();    //响应状态
int StatusCode = httpResponse .getStatusLine().getStatusCode(); //获取响应状态码
ProtocolVersion protocolVersion = httpResponse .getProtocolVersion(); //协议的版本号
String phrase = httpResponse .getStatusLine().getReasonPhrase(); //是否ok
Header[] headers = httpResponse.getAllHeaders();//获取头信息
for (Header header : headers) {System.out.println(header.toString());}//输出头信息
if(StatusCode == HttpStatus.SC_OK){                          //状态码200表示响应成功
    //获取实体内容
    HttpEntity entity = httpResponse.getEntity();
    String entityString = EntityUtils.toString (entity,"gbk"); //注意设置编码
    //输出实体内容
    System.out.println(entityString);
    EntityUtils.consume(httpResponse.getEntity());       //消耗实体
}else {
    //关闭HttpEntity的流实体
    EntityUtils.consume(httpResponse.getEntity());        //消耗实体
}
```

### 4、EntityUtils类

EntityUtils是Httpclient的响应实体类，可以用它操作响应实体

**数据转换**:

- `toString(HttpEntity entity, Charset defaultCharset)`: 将 `HttpEntity` 的内容转换为字符串，支持字符集设置。
- `toByteArray(HttpEntity entity)`: 将 `HttpEntity` 的内容读取为字节数组。

**资源管理**:

- `consume(HttpEntity entity)`: 关闭输入流，释放与实体相关的资源。
- `consumeQuietly(HttpEntity entity)`: 在处理异常时安静地关闭输入流。

**更新实体**:

- `updateEntity(HttpResponse response, HttpEntity entity)`: 用新的 `HttpEntity` 更新 `HttpResponse` 中的实体，同时确保释放旧的实体资源。

### 5、设置头信息

**5.1通过HttpRequest对象设置头信息**

- `httpget.setHeader(String name, String value)`

```java
HttpClient httpClient = HttpClients.custom().build(); //初始化httpclient
HttpGet httpget = new HttpGet("https://searchcustomerexperience.techtarget.com/info/news"); //使用的请求方法
//请求头配置
httpget.setHeader("Accept", "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8");
httpget.setHeader("Accept-Encoding", "gzip, deflate");
httpget.setHeader("Accept-Language", "zh-CN,zh;q=0.9");
httpget.setHeader("Cache-Control", "max-age=0");
httpget.setHeader("Host", "searchcustomerexperience.techtarget.com");
httpget.setHeader("User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.108 Safari/537.36"); //这项内容很重要
HttpResponse response = httpClient.execute(httpget);  //发出get请求
```

**5.2通过配置HttpClient设置默认请求头**

- `HttpClient.setDefaultHeaders(Collection<? extends Header> defaultHeaders)`

```java
List<Header> headerList = new ArrayList<Header>(); //通过集合封装头信息
headerList.add(new BasicHeader(HttpHeaders.ACCEPT, "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8"));
headerList.add(new BasicHeader(HttpHeaders.USER_AGENT, "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.108 Safari/537.36"));
headerList.add(new BasicHeader(HttpHeaders.ACCEPT_ENCODING, "gzip, deflate"));
headerList.add(new BasicHeader(HttpHeaders.CACHE_CONTROL, "max-age=0"));
headerList.add(new BasicHeader(HttpHeaders.CONNECTION, "keep-alive"));
headerList.add(new BasicHeader(HttpHeaders.ACCEPT_LANGUAGE, "zh-CN,zh;q=0.9"));
headerList.add(new BasicHeader(HttpHeaders.HOST, "searchcustomerexperience.techtarget.com"));
//构造自定义的HttpClient对象
HttpClient httpClient = HttpClients.custom()
       .setDefaultHeaders(headerList).build(); 
HttpGet httpget = new HttpGet("https://searchcustomerexperience.techtarget.com/info/news"); //使用的请求方法
//获取结果
HttpResponse response = httpClient.execute(httpget);  //发出get请求
```

### 6、POST提交表单

利用UrlEncodedFormEntity实例将表单转化为entity

```java
//建立一个NameValuePair数组，用于存储欲传送的参数
List<NameValuePair> nvps = new ArrayList<NameValuePair>();  
nvps.add(new BasicNameValuePair("email", "******"));   //输入你的邮箱地址
nvps.add(new BasicNameValuePair("password", "******"));   //输入你的密码
HttpResponse response = null;
//表单参数提交
httpost.setEntity(new UrlEncodedFormEntity(nvps, HTTP.UTF_8));  
response = httpclient.execute(httpost); 
```

### 7、设置超时时间

建立RequestConfig对象，录入配置信息

```java
RequestConfig requestConfig = RequestConfig.custom()
       .setSocketTimeout(2000)
       .setConnectTimeout(2000)
       .setConnectionRequestTimeout(10000)
       .build();//设置请求和传输超时时间
```

**7.1对HttpRequest对象设置超时时间**

```java
HttpClient httpClient = HttpClients.createDefault(); //初始化httpClient
HttpGet httpGet=new HttpGet("https://baidu");
httpGet.setConfig(requestConfig);  //httpget信息配置
```

**7.2对HttpClient设置默认超时时间**

```java
HttpClient httpClient = HttpClients.custom()
       .setDefaultRequestConfig(requestConfig)
       .build();
HttpGet httpGet = new HttpGet("https://baidu");
```

### 8、代理服务器

和设置超时时间一样，配置RequestConfig类

```java
RequestConfig defaultRequestConfig = RequestConfig.custom()
       .setProxy(new HttpHost("47.107.172.6",8000, null))
       .build();   //添加代理
```

### 9、文件下载

可以使用HttpEntity将相应实体转化为字节数组，再利用输出流的方式写入指定文件。

不过更方便的方式是HttpEntity中提供了writeTo（OutputStream）方法，可以直接将响应实体写入输出流。

```java
OutputStream out = new FileOutputStream("file/httpd-2.4.37.tar.gz");
httpResponse.getEntity().writeTo(out);
```

### 10、Https请求认证

首先实现X509证书信任管理器

```java
//实现X509TrustManager接口
private static class SSL509TrustManager implements X509TrustManager {
    //检查客户端证书
    public void checkClientTrusted(X509Certificate[] x509Certificates, String s) {
       //do nothing 接受任意客户端证书
    }
    //检查服务器端证书  
    public void checkServerTrusted(X509Certificate[] x509Certificates, String s)  {
       //do nothing  接受任意服务端证书
    }
    //返回受信任的X509证书
    public X509Certificate[] getAcceptedIssuers() {
       return new X509Certificate[0];
    }
};
```

使用SSLConnectionSocketFactory()方法实例化连接池管理器

基于实例化的连接池连接池管理器和RequestConfig配置信息实例化一个可以执行HTTPS请求的HttpClient

```java
/**
	 * 基于SSL配置httpClient
	 * @param  SSLProtocolVersion(SSL, SSLv3, TLS, TLSv1, TLSv1.1, TLSv1.2)
	 * @return httpClient
	 */
public HttpClient initSSLClient(String SSLProtocolVersion){
    //创建默认的请求配置
    RequestConfig defaultConfig = null;  
    //创建连接池
    PoolingHttpClientConnectionManager pcm = null;
    try {
        //创建信任管理
        X509TrustManager xtm = new SSL509TrustManager(); 
        //创建SSLContext对象,，并使用指定的信任管理器初始化
        SSLContext context = SSLContext.getInstance(SSLProtocolVersion);
        context.init(null, new X509TrustManager[]{xtm}, null);
        /*从SSLContext对象中得到SSLConnectionSocketFactory对象
			 NoopHostnameVerifier.INSTANCE表示接受接受任何有效的和符合目标主机的SSL会话*/
        SSLConnectionSocketFactory sslConnectionSocketFactory = new SSLConnectionSocketFactory(context, NoopHostnameVerifier.INSTANCE);
        //创建连接工厂
        Registry<ConnectionSocketFactory> sfr = RegistryBuilder.<ConnectionSocketFactory>create()
            .register("http", PlainConnectionSocketFactory.INSTANCE)
            .register("https", sslConnectionSocketFactory).build();
        //基于配置创建连接池
        pcm = new PoolingHttpClientConnectionManager(sfr);
    }catch(NoSuchAlgorithmException | KeyManagementException e){
        e.printStackTrace();
    }
    //设置全局请求配置,包括Cookie规范,HTTP认证,超时
    // 创建一个RequestConfig对象，并设置其属性
    defaultConfig = RequestConfig.custom()
        // 设置Cookie规范为STANDARD_STRICT
        .setCookieSpec(CookieSpecs.STANDARD_STRICT)
        // 设置是否启用Expect-Continue握手
        .setExpectContinueEnabled(true)
        // 设置目标服务器首选的认证方案为NTLM和DIGEST
        .setTargetPreferredAuthSchemes(Arrays.asList(AuthSchemes.NTLM, AuthSchemes.DIGEST))
        // 设置代理服务器首选的认证方案为BASIC
        .setProxyPreferredAuthSchemes(Arrays.asList(AuthSchemes.BASIC))
        // 设置连接请求、连接、套接字超时时间均为30秒
        .setConnectionRequestTimeout(30*1000)
        .setConnectTimeout(30*1000)
        .setSocketTimeout(30*1000)
        .build();
    //初始化httpclient
    HttpClient httpClient = HttpClients.custom().setConnectionManager(pcm).setDefaultRequestConfig(defaultConfig)
        .build();
    return httpClient;
}
```

调用上述的initSSLClient方法并设置SSLProtocolVersion为“YLS”，实现可以对HTTPS请求的HttpClient

```java
String url = "https://cn.kompass.com/a/hospitality-tourism-hotel-and-catering-industries/78/";
SSLClient sslClient = new SSLClient();   //实例化
HttpClient httpClientSSL = sslClient.initSSLClient("TLS");
HttpGet httpGet = new HttpGet(url);
//获取结果
HttpResponse httpResponse = null;
try {
    httpResponse = httpClientSSL.execute(httpGet);
} catch (IOException e) {
    e.printStackTrace();
}
if(httpResponse .getStatusLine().getStatusCode() == HttpStatus.SC_OK){ //状态码200表示响应成功
    //获取实体内容
    String entity = EntityUtils.toString (httpResponse.getEntity(),"UTF-8");
    //输出实体内容
    System.out.println(entity);
    EntityUtils.consume(httpResponse.getEntity());       //消耗实体
}else {
    //关闭HttpEntity的流实体
    EntityUtils.consume(httpResponse.getEntity());        //消耗实体
}
```

### 11、多线程

```java
public static void main(String[] args) throws FileNotFoundException {
    // 创建连接配置，设置字符编码和错误处理方式
    ConnectionConfig connectionConfig = ConnectionConfig.custom()
          // 忽略字符编码错误的处理
          .setMalformedInputAction(CodingErrorAction.IGNORE)
          .setUnmappableInputAction(CodingErrorAction.IGNORE)
          // 设置字符集为UTF-8
          .setCharset(Consts.UTF_8)
          .build();
    
    // 创建Socket配置，设置TCP连接的延迟选项
    SocketConfig socketConfig = SocketConfig.custom()
          // 启用TCP无延迟选项，以减少网络延迟
          .setTcpNoDelay(true)
          .build();
    
    // 创建连接池管理器
    PoolingHttpClientConnectionManager pcm = new PoolingHttpClientConnectionManager();
    // 设置连接池的最大连接数
    pcm.setMaxTotal(100);
    // 设置每个路由的最大连接数
    pcm.setDefaultMaxPerRoute(10);
    // 将连接配置信息应用于连接池
    pcm.setDefaultConnectionConfig(connectionConfig);
    // 将Socket配置信息应用于连接池
    pcm.setDefaultSocketConfig(socketConfig);
    
    // 配置全局请求设置，包括Cookie策略、HTTP认证和超时设置
    RequestConfig defaultConfig = RequestConfig.custom()
          // 设置Cookie策略为标准严格模式
          .setCookieSpec(CookieSpecs.STANDARD_STRICT)
          // 启用期望继续
          .setExpectContinueEnabled(true)
          // 设置目标认证方案的优先级
          .setTargetPreferredAuthSchemes(Arrays
                .asList(AuthSchemes.NTLM, AuthSchemes.DIGEST))
          // 设置代理认证方案的优先级
          .setProxyPreferredAuthSchemes(Arrays.asList(AuthSchemes.BASIC))
          // 设置连接请求超时时间为30秒
          .setConnectionRequestTimeout(30 * 1000)
          // 设置连接超时时间为30秒
          .setConnectTimeout(30 * 1000)
          // 设置Socket超时时间为30秒
          .setSocketTimeout(30 * 1000)
          .build();
    
    // 创建一个可关闭的HTTP客户端，并应用连接池和请求配置
    CloseableHttpClient httpClient = HttpClients.custom()
          .setConnectionManager(pcm)
          .setDefaultRequestConfig(defaultConfig)
          .build();
    
    // 定义需要请求的URL数组
    String[] urlArr = {
          "https://hbr.org/podcasts",
          "https://hbr.org/magazine",
          "https://hbr.org/most-popular",
          "https://hbr.org/big-ideas",
          "https://hbr.org/reading-lists"
    };
    
    // 创建一个固定大小的线程池，最多3个线程并行执行
    ExecutorService exec = Executors.newFixedThreadPool(3);
    
    // 遍历每个URL，下载HTML文件
    for (int i = 0; i < urlArr.length; i++) {
       // 提取文件名用于输出
       String filename = urlArr[i].split("org/")[1]; // HTML文件的输出名称
       // 创建文件输出流，指定输出目录和文件名
       OutputStream out = new FileOutputStream("file/" + filename);
       // 创建HTTP GET请求对象
       HttpGet httpget = new HttpGet(urlArr[i]);
       // 启动线程执行下载请求，使用DownHtmlFileThread类处理下载
       exec.execute(new DownHtmlFileThread(httpClient, httpget, out));
    }
    // 关闭线程池，禁止提交新的任务
    exec.shutdown();
}

```



```java
static class DownHtmlFileThread extends Thread {
    private final CloseableHttpClient httpClient;
    private final HttpContext context;
    private final HttpGet httpget;
    private final OutputStream out;

    //输入的参数
    public DownHtmlFileThread(CloseableHttpClient httpClient,
                        HttpGet httpget, OutputStream out) {
       this.httpClient = httpClient;
       this.context = HttpClientContext.create();
       this.httpget = httpget;
       this.out = out;
    }

    @Override
    public void run() {
       System.out.println(Thread.currentThread().getName() +
             "线程请求的URL为:" + httpget.getURI());
       try {
          CloseableHttpResponse response = httpClient.execute(
                httpget, context);  //执行请求
          try {
             //HTML文件写入文档
             out.write(EntityUtils.toString(response.getEntity(), "gbk")
                   .getBytes());
             out.close();
             //消耗实体
             EntityUtils.consume(response.getEntity());
          } finally {
             response.close(); //关闭响应
          }
       } catch (ClientProtocolException ex) {
          ex.printStackTrace(); // 处理 protocol错误
       } catch (IOException ex) {
          ex.printStackTrace(); // 处理I/O错误
       }
    }
}
```

## 四、网页内容解析

### **（一）HTML解析**

### 1、CSS选择器

#### 1.1基础选择器

| 选择器  | 描述           | 案例 | 含义                  |
| ------- | -------------- | ---- | --------------------- |
| .class  | 对类别选择     | .aaa | 匹配class=“aaa”的元素 |
| element | 对标签选择     | aaa  | 匹配标签为<aaa>的元素 |
| #id     | 对id选择       | #aaa | 选择id="aaa"的元素    |
| *       | 通用元素选择器 | *    | 选择所有              |

#### 1.2属性选择器

| 选择器           | 含义                                 |
| ---------------- | ------------------------------------ |
| [aaa]            | 选择带有aaa属性的所有元素            |
| [^aaa]           | 选择以aaa为前缀的元素                |
| [title="value"]  | 选择title="value"的所有元素          |
| [title~="value"] | 选择title包含"value"的所有元素       |
| [title^="value"] | 选择以"value"为前缀的title属性的元素 |
| [title$="value"] | 选择以"value"为后缀的title属性的元素 |

#### 1.3组合选择器

| 选择器  | 含义                             | 描述               |
| ------- | -------------------------------- | ------------------ |
| aaa,bbb | 选择所有<aaa>元素和<bbb>元素     | 多元素选择器       |
| aaa bbb | 匹配<aaa>元素内部所有<bbb>元素   | 后代选择器         |
| aaa>bbb | 查找<aaa>下一层的<bbb>元素       | 子代选择器         |
| aaa~bbb | 选择<aaa>之后所有同级<bbb>元素   | 兄弟选择器         |
| aaa+bbb | 选择<aaa>之后下一个同级<bbb>元素 | 直接相邻兄弟选择器 |

### 2、Xpath语法

| 表达式   | 描述                                                 | 例子                    | 含义                                        |
| -------- | ---------------------------------------------------- | ----------------------- | ------------------------------------------- |
| nodename | 选取此节点所有子节点                                 | body                    | 选取<body>元素的所有字节点                  |
| /        | 从根节点选取                                         | /html                   | 选取根节点<html>                            |
| //       | 从选择的当前节点选择文档中的节点，而不考虑他们的位置 | //div                   | 选取所有<div>节点，而不管他们在文档中的位置 |
| .        | 选取当前节点                                         | ./p                     | 选取当前的<p>节点                           |
| @        | 选取属性                                             | //a[@herf]              | 选取所有拥有herf属性的<a>节点               |
| @        | 选取属性                                             | ``//div[@id=`course`]`` | 选取所有id属性为`course`的<div>节点         |

Xpath语法的组合使用

``//div[@id=`aaa`]/h1``选取所有id属性为aaa的所有<div>节点下的<h1>节点

`//body/a[1]`选取<body>下第一个<a>节点

`//body/a[last()]`选取<body>下最后一个<a>节点

### 3、Jsoup解析HTML文件

Node节点：HTML中标所有内容都可以看成一个节点，包括很多种如Attribute（属性），Note（注释），Text（文本），元素（Element）

Element元素：节点的子集，一个元素也是一个节点

Document文档：整个HTML的源码内容,继承于Element

#### 3.1解析静态HTML文件

给定HTML字符串可以用Jsoup.prase(String html)方法,将String类型的HTML转化为Document类型

利用ELement类下的select(String cssQuery)方法进行定位

```java
//HTML静态文件
String html = "<html><body><div id=\"w3\"> <h1>浏览器脚本教程</h1> <p><strong>从左侧的菜单选择你需要的教程！</strong></p> </div>"
+ "<div>  <div id=\"course\"> <ul> <li><a href=\"/js/index.asp\" title=\"JavaScript 教程\">JavaScript</a></li> </ul> </div> </body></html>";
//转化成Document
Document doc = Jsoup.parse(html); 
//基于CSS选择器获取元素,也可写成[id=w3]
Element element = doc.select("div[id=w3]").get(0);
System.out.println("输出解析的元素内容为:");
System.out.println(element);
//从Element提取内容(抽取一个Node对应的信息)
String text1 = element.select("h1").text(); 
//从Element提取内容(抽取一个Node对应的信息)
String text2 = element.select("p").text(); 
System.out.println("抽取的文本信息为:");
System.out.println(text1 + "\t" + text2);
```

#### 3.2解析url加载的Document

利用Jsoup请求url,获得对应Document,再用select解析

select(String cssQuery);

```java
public static void main(String[] args) throws IOException {
    //获取URL对应的Document
    Document doc = Jsoup.connect("http://www.baidu.com").timeout(5000).get();
    //从Element提取内容(抽取一个Node对应的信息)
    System.out.println(doc.text());
    String text1 = doc.select("h1").text();
    //从Element提取内容(抽取一个Node对应的信息)
    String text2 = doc.select("p").text();
    System.out.println("抽取的文本信息为:");
    System.out.println(text1 + "\t" + text2);
```

#### 3.3Jsoup遍历元素

Element类继承自ArrayList<Element>，所以可以用遍历ArrayList的方法遍历

```java
//获取URL对应的Document
Document doc = Jsoup.connect("https://ssr1.scrape.center/").timeout(5000).get();
//从Element提取内容(抽取一个Node对应的信息)
Elements select = doc.select("#app .name");
System.out.println("抽取的文本信息为:");
for (Element element : select) {
    System.out.println(element.text());
}
```

#### 3.4Jsoup筛选元素的方法

```java
//获取URL对应的HTML内容
Document doc = Jsoup.connect("http://www.********.com.cn/b.asp").timeout(5000).get();
//[attr=value]: 利用属性值来查找元素,例如[id=course]; 通过tagname: 通过标签查找元素，比如：a
System.out.println(doc.select("[id=course]").select("a").get(0).text());
//fb[[attr=value]:利用标签属性联合查找
System.out.println(doc.select("div[id=course]").select("a").get(0).text());
//#id: 通过ID查找元素,例如，#course
System.out.println(doc.select("#course").select("a").get(0).text());
//通过属性属性查找元素，比如：[href]
System.out.println(doc.select("#course").select("[href]").get(0).text());
//.class通过class名称查找元素
System.out.println(doc.select(".browserscripting").text());
//[attr^=value], [attr$=value], [attr*=value]利用匹配属性值开头、结尾或包含属性值来查找元素(很常用的方法)
System.out.println(doc.select("#course").select("[href$=index.asp]").text());
//[attr~=regex]: 利用属性值匹配正则表达式来查找元素,*指匹配所有元素
System.out.println(doc.select("#course").select("[href~=/*]").text());
```

```java
//获取URL对应的Document
Document doc = Jsoup.connect("http://www.********.com.cn/b.asp").timeout(5000).get();
//基于id获取元素
Element element_id = doc.getElementById("course");
//基于标签名称获取元素集合
Elements element_tag = doc.getElementById("course").getElementsByTag("a");
//基于属性获取元素集合
Elements element_A = doc.getElementById("course").getElementsByAttribute("href");
//通过类名获取元素集合
Elements elements = doc.getElementsByClass("browserscripting"); 
//基于属性前缀获取元素集合
Elements element_As = doc.getElementsByAttributeStarting("hre"); 
//基于属性与属性值获取元素
Elements element_Av = doc.getElementsByAttributeValue("id","tools"); 
//获取兄弟元素集合
Elements element_Se = doc.getElementById("navfirst").siblingElements(); 
//获取下一个兄弟元素
Element element_Ns = doc.getElementById("navfirst").nextElementSibling();
//获取上一个兄弟元素
Element element_Ps = doc.getElementById("navfirst").previousElementSibling();
```

#### 3.5支持Xpath语法的JsoupXpath

Jsoup在select时使用的是css选择器，不支持Xpath

JsoupXpath是在Jsoup上拓展的支持Xpath语法的HTML文件解析器

依赖：

```xml
<dependency>
    <groupId>cn.wanghaomiao</groupId>
    <artifactId>JsoupXpath</artifactId>
    <version>2.2</version>
</dependency>
```





在JsoupXath中操作的对象为JXDocument

JsoupXath提供了四种方法实例化JXDocument对象

- Document类型的HTML文档
- String类型的HTML字符串
- Element类型的元素集合
- Sting类型的url

```java
public static JXDocument create(Document doc) {
    Elements els = doc.children();
    return new JXDocument(els);
}

public static JXDocument create(Elements els) {
    return new JXDocument(els);
}

public static JXDocument create(String html) {
    Elements els = Jsoup.parse(html).children();
    return new JXDocument(els);
}

public static JXDocument createByUrl(String url) {
    Elements els;
    try {
        els = Jsoup.connect(url).get().children();
    } catch (Exception var3) {
        Exception e = var3;
        throw new XpathParserException("url资源获取失败", e);
    }
    return new JXDocument(els);
}
```

利用JXDocument.selN(String xpath)解析HTML

```java
//基于URL创建JXDocument
JXDocument jxd = JXDocument.createByUrl("http://www.w3school.com.cn/b.asp");
//Xpath语句获取节点集合
List<JXNode> list = jxd.selN("//*[@id='course']/ul/li/a");
//遍历节点
      for (JXNode node : list) {
          System.out.println("标题为:" + node.asElement().text() +
                  "\tURL为:" + node.asElement().attr("href"));
      }
```

### 4、Jsoup解析XML

Jsoup解析XML和HTML的方法相同，皆使用CSS选择器

```java
//获取URL对应的HTML内容
String url = "http://db.auto.****.com/cxdata/xml/sales/model/model1001sales.xml";
Document doc = Jsoup.connect(url).timeout(5000).get();
//Jsoup选择器解析
Elements sales_ele = doc.select("sales");
for (Element elem:sales_ele) {
    int salesnum=Integer.valueOf(elem.attr("salesnum"));
        String date = elem.attr("date");
        System.out.println("月份:" + date + "\t销量:" + salesnum);
```

### 5、JSON解析

#### 5.1 JOSN校正

对提取出的JSON进行手动矫正，如掐头去尾排除多余内容

在线校验工具：[JSON在线解析及格式化验证 - JSON.cn](https://www.json.cn/)

#### 5.2 org.json解析JSON

##### 5.2.1导入依赖

```xml
<dependency>
    <groupId>org.json</groupId>
    <artifactId>json</artifactId>
    <version>20180130</version>
</dependency>
```

##### 5.2.2 JSONObject类

JSONObject类的功能是处理JSON对象，包含了实例化JSONObject类和在类中添加数据等方法

JSONObject构造方法

- `JSONObject()` 构造空的JSONObject
- `JSONObject(String sourse)` 将字符串转化为JSONObject
- `JSONObject(Map<?,?> m)` 基于Map集合创建JSONObject  
- `JSONObject(Object Bean)` 基于Bean对象实例化JSONObject

 添加/ 获取数据的方法

- `JSONObject accumulate(String key, Object value)` JSONObject 对象添加数据
- `ISONObject append(String key, Object value)` JSONObiect 对象添加数据
- `Object get(String key)`  获取 JSONObject 对象中键 key 对应的 value,返回的类型为 Object
- `boolean getBoolean(String key)` 判断 JSONObject 对象中键 key 是否存在
- `String getString(String key)` 获取key对应的value，返回为String

##### 5.2.3 JSONArray类

JSONArray类的功能是解析JSON数组

```java
//json数组
String json = "[{\"id\":\"01\",\"language\": \"Java\",\"edition\": \"third\",\"author\": \"Herbert Schildt\"},{\"id\":\"07\", \"language\": \"C++\",\"edition\": \"second\",\"author\": \"E.Balagurusamy\"}]";
/*
 * 转化成JSONArray对象
 * 使用的是JSONArray(String source)构造方法
 * */
JSONArray jsonarray = new JSONArray(json); 
for (int i = 0; i < jsonarray.length(); i++) {
    /*
     * 获取指定json对象
     * 使用的是JSONObject getJSONObject(int index)方法
     * */
    JSONObject jsonobj = jsonarray.getJSONObject(i);   
    String id = jsonobj.getString("id");   
    String language = jsonobj.getString("language");  
    String edition = jsonobj.getString("edition"); 
    //输出解析的结果
    System.out.println(id + "\t" + language + "\t" + edition);
}   
```

#### 5.3 Gson解析JSON

Gson可以将复杂的JSON数据转化为Java对象

##### 5.3.1导入依赖

```xml
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.5</version>
</dependency>
```

##### 5.3.2解析JSON对象

在使用Gson解析数据时，要先将其实例化，具体操作方式有两种

```java
Gson gson = new Gson();
Gson gson = new GsonBulider().creat();
```

在实例化之后，可以用Gson中的fromJson方法将JSON转化为Java中的对象

```java
public <T> T fromJson(String json, Class<T> classOfT)
```

示例java对象：

```java
public class BookModel {
    private String id;
    private String language;
    private String edition;
    private String author;
}
```

Gson转化：

```java
//json对象
String json = "{\"id\":\"07\",\"language\": \"C++\",\"edition\": \"second\",\"author\": \"E.Balagurusamy\"}";
Gson gson = new Gson();  //初始化操作
BookModel model = gson.fromJson(json, BookModel.class); //转化成Java对象
//输出数据
System.out.println(model.getId() + "\t" + model.getLanguage() + "\t" + model.getEdition());
```

##### 5.3.3解析JSON数组

为支持泛型，利用TyprToken来创建Type对象，在使用Gson将JSON数组转化为指定类型的集合

```java
public <T> T fromJson(String json, Type typeOfT)
```

```json
[
  {
    "id": "01",
    "language": "Java",
    "edition": "third",
    "author": "Herbert Schildt"
  },
  {
    "id": "07",
    "language": "C++",
    "edition": "second",
    "author": "E.Balagurusamy"
  }
]
```

```java
//json数组
String json = "[{\"id\":\"01\",\"language\": \"Java\",\"edition\": \"third\",\"author\": \"Herbert Schildt\"},{\"id\":\"07\", \"language\": \"C++\",\"edition\": \"second\",\"author\": \"E.Balagurusamy\"}]";
Gson gson = new Gson();  //实例化操作
Type listType = new TypeToken<List<BookModel>>(){}.getType();  //TypeToken操作
List<BookModel> listmodel = gson.fromJson(json, listType); //转化成集合
//输出数据
for (BookModel model : listmodel) {
    System.out.println(model.getId() + "\t" + model.getLanguage() + "\t" + model.getEdition());
}
```

##### 5.3.4解析复杂嵌套JSON

按照层次创建JavaBean对象，再用gson转化即可

```java
public class BookSummaryModel {
    private String goodRateShow;
    private String poorRateShow;
    private String poorCountStr;
    private List<BookModel> book;
}
```

```json
{
  "goodRateShow": 99,
  "poorRateShow": 1,
  "poorCountStr": "500",
  "book": [
    {
      "id": "01",
      "language": "Java",
      "edition": "third",
      "author": "Herbert Schildt"
    },
    {
      "id": "07",
      "language": "C++",
      "edition": "second",
      "author": "E.Balagurusamy"
    }
  ]
}

```

```java
//复杂一点的JSON数据
String json = "{\"goodRateShow\":99,\"poorRateShow\":1,\"poorCountStr\":\"500+\",\"book\": [{\"id\":\"01\",\"language\": \"Java\",\"edition\": \"third\",\"author\": \"Herbert Schildt\"},{\"id\":\"07\", \"language\": \"C++\",\"edition\": \"second\",\"author\": \"E.Balagurusamy\"}]}";
Gson gson = new Gson();  //初始化操作
BookSummaryModel smodel  = gson.fromJson(json, BookSummaryModel.class); //转化成Java对象
//对象中拿到集合
List<BookModel> listmodel = smodel.getBook();
//输出数据
for (BookModel model : listmodel) {
    System.out.println(smodel .getGoodRateShow() + 
          "\t"+ smodel .getPoorCountStr() + "\t" + model.getId() + 
          "\t" + model.getLanguage() + "\t" + model.getEdition());
}
```

#### 5.4 Fastjson

是阿里巴巴基于java开发的一款高性能JSON操作类库

##### 5.4.1导入依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.47</version>
</dependency>
```

##### 5.4.2解析JSON对象与数组

Fastison 解析 JSON 数据的方式与 Gson类似，即将JSON 数据转化成 JavaBean对象。下面提供了解析 JSON 对象和 JSON 数组的常用方法。

```java
//操作JSON 对象的方法
<T>T parse0bject(String text,Class<T> clazz)
//操作JSON数组的方法
<T> List<T>parseArray(String text,Class<T> clazz)
//基于泛型的方式，可操作JSON数组
<T>T parseObject(String text,TypeReference<T> type, feature.features)
```

```java
//json对象
String json = "{\"id\":\"07\",\"language\": \"C++\",\"edition\": \"second\",\"author\": \"E.Balagurusamy\"}";
//使用fastjson解析Json对象
BookModel model = JSON.parseObject(json, BookModel.class); 
//输出解析结果
System.out.println(model.getId() + "\t" + model.getLanguage() + 
       "\t" + model.getEdition());
```

```java
//json数组
String json = "[{\"id\":\"01\",\"language\": \"Java\",\"edition\": \"third\",\"author\": \"Herbert Schildt\"},{\"id\":\"07\", \"language\": \"C++\",\"edition\": \"second\",\"author\": \"E.Balagurusamy\"}]";
//使用fastjson解析Json数组
List<BookModel> listmodel = JSON.parseObject(json, new TypeReference<List<BookModel>>(){}); //第一种方式
//List<BookModel>  listmodel = JSON.parseArray(json, BookModel.class);  //第二种方式
//输出数据
for (BookModel model : listmodel) {
    System.out.println(model.getId() + "\t" + model.getLanguage() + 
                       "\t" + model.getEdition());
}
```

## 五、数据储存

[Java 流(Stream)、文件(File)和IO | 菜鸟教程 (runoob.com)](https://www.runoob.com/java/java-files-io.html)























