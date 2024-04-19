+++
title = 'HttpClient踩坑'
date = 2024-04-19T10:24:07+08:00
draft = false
+++

### apache httpClent 使用时出现异常

```java
Caused by: org.apache.http.conn.ConnectionPoolTimeoutException: Timeout waiting for connection from pool
          at org.apache.http.impl.conn.PoolingHttpClientConnectionManager.leaseConnection(PoolingHttpClientConnectionManager.java:316) ~[httpclient-4.5.13.jar:4.5.13]
          at org.apache.http.impl.conn.PoolingHttpClientConnectionManager$1.get(PoolingHttpClientConnectionManager.java:282) ~[httpclient-4.5.13.jar:4.5.13]
          at org.apache.http.impl.execchain.MainClientExec.execute(MainClientExec.java:190) ~[httpclient-4.5.13.jar:4.5.13]
          at org.apache.http.impl.execchain.ProtocolExec.execute(ProtocolExec.java:186) ~[httpclient-4.5.13.jar:4.5.13]
          at org.apache.http.impl.execchain.RetryExec.execute(RetryExec.java:89) ~[httpclient-4.5.13.jar:4.5.13]
          at org.apache.http.impl.execchain.RedirectExec.execute(RedirectExec.java:110) ~[httpclient-4.5.13.jar:4.5.13]
          at org.apache.http.impl.client.InternalHttpClient.doExecute(InternalHttpClient.java:185) ~[httpclient-4.5.13.jar:4.5.13]
          at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:83) ~[httpclient-4.5.13.jar:4.5.13]
          at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:108) ~[httpclient-4.5.13.jar:4.5.13]
          at org.apache.kyuubi.plugin.DscAuthPlugin$.doDscAuth(DscAuthPlugin.scala:78) ~[kyuubi-server_2.12-1.6.1-incubating.jar:1.6.1-incubating]
          ... 24 more
```

 `httpClient`的创建时候使用默认的`PoolingHttpClientConnectionManager`

  ```scala
  private val httpClient = HttpClientBuilder.create().setDefaultRequestConfig(requestConfig).build()
  
  org.apache.http.impl.conn.PoolingHttpClientConnectionManager#setDefaultMaxPerRoute  //default value 2
  org.apache.http.impl.conn.PoolingHttpClientConnectionManager#getMaxPerRoute
  ```
PoolingHttpClientConnectionManager的配置中有两个最大连接数量，分别控制着总的最大连接数量和每个route的最大连接数量，最大路由连接数MaxPerRoute默认为2，总连接数量MaxTotal默认为20。每次新来一个请求，如果连接池中已经存在相同route并且可用的connection，连接池就会直接复用这个connection，当不存在route相同的connection，就会新建一个connection为之服务；如果连接池已满，则请求会等待直到被服务或者超时。每次读取`response`会释放connection

消费resp，此时会close，release conn

```java
	at org.apache.http.impl.execchain.ConnectionHolder.releaseConnection(ConnectionHolder.java:120)
	at org.apache.http.impl.execchain.ResponseEntityProxy.releaseConnection(ResponseEntityProxy.java:76)
	at org.apache.http.impl.execchain.ResponseEntityProxy.streamClosed(ResponseEntityProxy.java:145)
	at org.apache.http.conn.EofSensorInputStream.checkClose(EofSensorInputStream.java:228)
	at org.apache.http.conn.EofSensorInputStream.close(EofSensorInputStream.java:172)
	at org.apache.http.client.entity.LazyDecompressingInputStream.close(LazyDecompressingInputStream.java:97)
	at org.apache.http.util.EntityUtils.consume(EntityUtils.java:90)
```

正常情况下，每个connection会在连接后释放，但是对`dsc` 鉴权处理有问题，当code不为`200`时，未消费response

code 为`200`时，使用jsckson处理entity，会close response

  ```scala
  val content = objectMapper.readTree(response.getEntity.getContent)
            
  at org.apache.http.impl.execchain.ConnectionHolder.releaseConnection(ConnectionHolder.java:120)
  	at org.apache.http.impl.execchain.ResponseEntityProxy.releaseConnection(ResponseEntityProxy.java:76)
  	at org.apache.http.impl.execchain.ResponseEntityProxy.streamClosed(ResponseEntityProxy.java:145)
  	at org.apache.http.conn.EofSensorInputStream.checkClose(EofSensorInputStream.java:228)
  	at org.apache.http.conn.EofSensorInputStream.close(EofSensorInputStream.java:172)
  	at java.util.zip.InflaterInputStream.close(InflaterInputStream.java:227)
  	at java.util.zip.GZIPInputStream.close(GZIPInputStream.java:136)
  	at org.apache.http.client.entity.LazyDecompressingInputStream.close(LazyDecompressingInputStream.java:94)
  	at com.fasterxml.jackson.core.json.UTF8StreamJsonParser._closeInput(UTF8StreamJsonParser.java:242)
  	at com.fasterxml.jackson.core.base.ParserBase.close(ParserBase.java:347)
  	at com.fasterxml.jackson.databind.ObjectMapper._readMapAndClose(ObjectMapper.java:2994)
  	at com.fasterxml.jackson.databind.ObjectMapper.readTree(ObjectMapper.java:1737)
  ```
 code不为`200`时抛出异常，response 未close

  ```scala
  24/01/17 10:03:08,099 [KyuubiTBinaryFrontendHandler-Pool: Thread-8956845] ERROR KyuubiTBinaryFrontendService: Error executing statement:
  org.apache.kyuubi.KyuubiException: Do dsc auth failed, http code: 504
          at org.apache.kyuubi.plugin.DscAuthPlugin$.doDscAuth(DscAuthPlugin.scala:89) ~[kyuubi-server_2.12-1.6.1-incubating.jar:1.6.1-incubating]
          at org.apache.kyuubi.operation.ExecuteStatement.beforeRun(ExecuteStatement.scala:60) ~[kyuubi-server_2.12-1.6.1-incubating.jar:1.6.1-incubating]
          at org.apache.kyuubi.operation.AbstractOperation.run(AbstractOperation.scala:162) ~[kyuubi-common_2.12-1.6.1-incubating.jar:1.6.1-incubating]
          at org.apache.kyuubi.session.AbstractSession.runOperation(AbstractSession.scala:99) ~[kyuubi-common_2.12-1.6.1-incubating.jar:1.6.1-incubating]
          at org.apache.kyuubi.session.KyuubiSessionImpl.runOperation(KyuubiSessionImpl.scala:191) ~[kyuubi-server_2.12-1.6.1-incubating.jar:1.6.1-incubating]
          at org.apache.kyuubi.session.AbstractSession.$anonfun$executeStatement$1(AbstractSession.scala:129) ~[kyuubi-common_2.12-1.6.1-incubating.jar:1.6.1-incubatin
  
  ```

 查询日志，2次504 code，connection pool 被占满，后续连接抛出异常

验证，未消费Resp，http 请求失败

  ```java
    public static void main(String[] args) throws IOException, InterruptedException {
      CloseableHttpClient httpclient = HttpClients.createDefault();
      HttpGet httpGet = new HttpGet("https://www.baidu.com");
      CloseableHttpResponse response1 = httpclient.execute(httpGet);
      System.out.println(response1.getStatusLine());
      CloseableHttpResponse response2 = httpclient.execute(httpGet);
      System.out.println(response2.getStatusLine());
      CloseableHttpResponse response3 = httpclient.execute(httpGet);
      System.out.println(response3.getStatusLine());
    }
  
  ```

使用httpClient时，最好每次消费Response， 最后 close response

### REF

 https://hc.apache.org/httpcomponents-client-4.5.x/quickstart.html

  > // The underlying HTTP connection is still held by the response object // to allow the response content to be streamed directly from the network socket. // In order to ensure correct deallocation of system resources // the user MUST call CloseableHttpResponse#close() from a finally clause. // Please note that if response content is not fully consumed the underlying // connection cannot be safely re-used and will be shut down and discarded // by the connection manager.