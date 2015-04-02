---
layout: default
title: HttpAsyncClient和httpclient
category: tech
---
##{{ page.title }}
最近一直挺忙(lan)的，也没心思写东西；
今儿个，把工作中用到的一个东东介绍一下吧；
HttpAsyncClient，就是这个，官网地址：[link](http://hc.apache.org/httpcomponents-asyncclient-4.0.x/index.html)
为什么用这个客户端工具，而不用httpclient？
官方是这么说的：
Designed for extension while providing **robust** support for the base HTTP protocol, 
HttpAsyncClient may be of interest to anyone building HTTP-aware client applications based on asynchronous, event driven I/O model.
这个是异步事件驱动模型来做的，简单来说就是性能高！至于有多高？我做了个测试；

见图：![12](/images/httpclient/client.png "picture")

程序员还是看代码比较直观：

    package com.tudou.is.util;
    
    import java.io.IOException;
    import java.util.ArrayList;
    import java.util.List;
    import java.util.concurrent.CountDownLatch;
    
    import org.apache.commons.io.IOUtils;
    import org.apache.http.HttpHost;
    import org.apache.http.HttpResponse;
    import org.apache.http.client.methods.HttpGet;
    import org.apache.http.concurrent.FutureCallback;
    import org.apache.http.conn.routing.HttpRoute;
    import org.apache.http.impl.nio.client.CloseableHttpAsyncClient;
    import org.apache.http.impl.nio.client.HttpAsyncClients;
    import org.apache.http.impl.nio.conn.PoolingNHttpClientConnectionManager;
    import org.apache.http.impl.nio.reactor.DefaultConnectingIOReactor;
    import org.apache.http.impl.nio.reactor.IOReactorConfig;
    import org.apache.http.nio.reactor.ConnectingIOReactor;
    import org.apache.http.nio.reactor.IOReactorException;
    
    public class AsyncHttpClient {
    
        private static CloseableHttpAsyncClient httpclient;
    
        // 连接超时时间
        private static final int CONNECTION_TIME_OUT = 3000;
        // 请求超时时间
        private static final int Socket_TIME_OUT = 3000;
        // 线程数，官方建议；其实一个线程也可以，一个线程对应一个selector
        private static final int THREAD_NUMBER = Runtime.getRuntime().availableProcessors();
    
        private AsyncHttpClient() {
    
        }
    
        public static synchronized CloseableHttpAsyncClient getHttpClient() throws IOReactorException {
            if (httpclient == null) {
    
                // Create I/O reactor configuration
                IOReactorConfig ioReactorConfig = IOReactorConfig.custom()
                        .setIoThreadCount(THREAD_NUMBER)
                        .setConnectTimeout(CONNECTION_TIME_OUT)
                        .setSoTimeout(Socket_TIME_OUT)
                        .build();
    
                // Create a custom I/O reactort
                ConnectingIOReactor ioReactor = new DefaultConnectingIOReactor(ioReactorConfig);
                PoolingNHttpClientConnectionManager connManager = new PoolingNHttpClientConnectionManager(ioReactor);
                connManager.setDefaultMaxPerRoute(100);
                // connManager.setMaxPerRoute(new HttpRoute(new HttpHost("a.sina.com", 80)), 100);
                connManager.setMaxPerRoute(new HttpRoute(new HttpHost("10.1.9.5", 80)), 100);//这里一定要配置，表示每个域名的连接池的连接数
    
                // RequestConfig requestConfig = RequestConfig.custom()
                // .setSocketTimeout(3000)
                // .setConnectTimeout(3000).build();
                httpclient = HttpAsyncClients.custom()
                        .setConnectionManager(connManager)
                        // .setDefaultRequestConfig(requestConfig)
                        .build();
                httpclient.start();
            }
            return httpclient;
        }
    
        public static List<String> getResp(final HttpGet[] requests)
                throws InterruptedException {
    
            if (httpclient == null) {
                try {
                    httpclient = getHttpClient();
                } catch (IOReactorException e) {
                    e.printStackTrace();
                }
            }
            final List<String> retList = new ArrayList<String>();
            final CountDownLatch latch = new CountDownLatch(requests.length);
            for (final HttpGet request : requests) {
                long start = System.currentTimeMillis();
                httpclient.execute(request, new FutureCallback<HttpResponse>() {
    
                    public void completed(final HttpResponse response) {

                        try {
                            if (response.getStatusLine().getStatusCode() == 200) {
                                retList.add(IOUtils.toString(response.getEntity().getContent(), "UTF-8"));
                            } else {
                                System.out.println("Error!!!" + response.getStatusLine().getStatusCode());
                            }
                        } catch (IllegalStateException e) {
                            e.printStackTrace();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                        // System.out.println(request.getRequestLine() + "->" +
                        // response.getStatusLine());
                        latch.countDown();
                    }
    
                    public void failed(final Exception ex) {
                        System.out.println(request.getRequestLine() + "->" + ex);
                        latch.countDown();
                    }
    
                    public void cancelled() {                        
                        System.out.println(request.getRequestLine() + " cancelled");
			latch.countDown();
                    }
    
                });
                long end = System.currentTimeMillis();
                // System.out.println("time:" + (end - start));
            }
            latch.await();
            return retList;
        }
    	}



<br /><p>{{ page.date | date_to_string }}</p>
