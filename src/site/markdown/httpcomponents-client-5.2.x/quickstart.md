<!--
    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at
    
      http://www.apache.org/licenses/LICENSE-2.0
    
    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.
-->

HttpClient Quick Start
======================

- Download 'Binary' package of the latest HttpClient 5.2 release or configure dependency on `HttpClient` and `Fluent HC`
  modules using a dependency manager of your choice as described [here](download.md).

- HttpClient 5.2 requires Java 1.8 or newer.

- The below code fragment illustrates the execution of HTTP GET and POST requests using the HttpClient native API.

    ```
    try (CloseableHttpClient httpclient = HttpClients.createDefault()) {
        HttpGet httpGet = new HttpGet("http://httpbin.org/get");
        // The underlying HTTP connection is still held by the response object
        // to allow the response content to be streamed directly from the network socket.
        // In order to ensure correct deallocation of system resources
        // the user MUST call CloseableHttpResponse#close() from a finally clause.
        // Please note that if response content is not fully consumed the underlying
        // connection cannot be safely re-used and will be shut down and discarded
        // by the connection manager.
        try (CloseableHttpResponse response1 = httpclient.execute(httpGet)) {
            System.out.println(response1.getCode() + " " + response1.getReasonPhrase());
            HttpEntity entity1 = response1.getEntity();
            // do something useful with the response body
            // and ensure it is fully consumed
            EntityUtils.consume(entity1);
        }
    
        HttpPost httpPost = new HttpPost("http://httpbin.org/post");
        List<NameValuePair> nvps = new ArrayList<>();
        nvps.add(new BasicNameValuePair("username", "vip"));
        nvps.add(new BasicNameValuePair("password", "secret"));
        httpPost.setEntity(new UrlEncodedFormEntity(nvps));
    
        try (CloseableHttpResponse response2 = httpclient.execute(httpPost)) {
            System.out.println(response2.getCode() + " " + response2.getReasonPhrase());
            HttpEntity entity2 = response2.getEntity();
            // do something useful with the response body
            // and ensure it is fully consumed
            EntityUtils.consume(entity2);
        }
    }
    ```

  Source can be found here
  [here](https://github.com/apache/httpcomponents-website/blob/master/samples/src/main/java/org/apache/hc/client5/http/examples/QuickStart.java)

- The same requests can be executed using a simpler, albeit less flexible, fluent API.

    ```
    // The fluent API relieves the user from having to deal with manual deallocation of system
    // resources at the cost of having to buffer response content in memory in some cases.
    
    Request.Get("http://targethost/homepage")
        .execute().returnContent();
    Request.Post("http://targethost/login")
        .bodyForm(Form.form().add("username",  "vip").add("password",  "secret").build())
        .execute().returnContent();
    ```

  Source can be found here
  [here](https://github.com/apache/httpcomponents-website/blob/master/samples/src/main/java/org/apache/hc/client5/http/examples/fluent/FluentQuickStart.java)

- The below code fragment illustrates the execution of HTTP requests using HttpClient async API.

    ```
    try (CloseableHttpAsyncClient httpclient = HttpAsyncClients.createDefault()) {
        // Start the client
        httpclient.start();
    
        // Execute request
        SimpleHttpRequest request1 = SimpleHttpRequests.get("http://httpbin.org/get");
        Future<SimpleHttpResponse> future = httpclient.execute(request1, null);
        // and wait until response is received
        SimpleHttpResponse response1 = future.get();
        System.out.println(request1.getRequestUri() + "->" + response1.getCode());
    
        // One most likely would want to use a callback for operation result
        CountDownLatch latch1 = new CountDownLatch(1);
        SimpleHttpRequest request2 = SimpleHttpRequests.get("http://httpbin.org/get");
        httpclient.execute(request2, new FutureCallback<SimpleHttpResponse>() {
    
            @Override
            public void completed(SimpleHttpResponse response2) {
                latch1.countDown();
                System.out.println(request2.getRequestUri() + "->" + response2.getCode());
            }
    
            @Override
            public void failed(Exception ex) {
                latch1.countDown();
                System.out.println(request2.getRequestUri() + "->" + ex);
            }
    
            @Override
            public void cancelled() {
                latch1.countDown();
                System.out.println(request2.getRequestUri() + " cancelled");
            }
    
        });
        latch1.await();
    
        // In real world one most likely would want also want to stream
        // request and response body content
        CountDownLatch latch2 = new CountDownLatch(1);
        AsyncRequestProducer producer3 = AsyncRequestBuilder.get("http://httpbin.org/get").build();
        AbstractCharResponseConsumer<HttpResponse> consumer3 = new AbstractCharResponseConsumer<HttpResponse>() {
    
            HttpResponse response;
    
            @Override
            protected void start(HttpResponse response, ContentType contentType) throws HttpException, IOException {
                this.response = response;
            }
    
            @Override
            protected int capacityIncrement() {
                return Integer.MAX_VALUE;
            }
    
            @Override
            protected void data(CharBuffer data, boolean endOfStream) throws IOException {
                // Do something useful
            }
    
            @Override
            protected HttpResponse buildResult() throws IOException {
                return response;
            }
    
            @Override
            public void releaseResources() {
            }
    
        };
        httpclient.execute(producer3, consumer3, new FutureCallback<HttpResponse>() {
    
            @Override
            public void completed(HttpResponse response3) {
                latch2.countDown();
                System.out.println(request2.getRequestUri() + "->" + response3.getCode());
            }
    
            @Override
            public void failed(Exception ex) {
                latch2.countDown();
                System.out.println(request2.getRequestUri() + "->" + ex);
            }
    
            @Override
            public void cancelled() {
                latch2.countDown();
                System.out.println(request2.getRequestUri() + " cancelled");
            }
    
        });
        latch2.await();
    
    }
    ```    

  Source can be found here
  [here](https://github.com/apache/httpcomponents-client/tree/master/httpclient5/src/test/java/org/apache/hc/client5/http/examples/AsyncQuickStart.java)


