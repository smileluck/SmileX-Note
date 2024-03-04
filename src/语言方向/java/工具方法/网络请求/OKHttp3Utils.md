[toc]

---

# OkHttp3

## Maven引入

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.10.0</version>
</dependency>
```




## Utils

```java
import java.util.Map;
import java.util.concurrent.TimeUnit;

import okhttp3.FormBody;
import okhttp3.MediaType;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.RequestBody;
import okhttp3.Response;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;


public class OkHttpUtils {
    private static final Logger log = LoggerFactory.getLogger(OkHttpUtils.class);
    public static MediaType JSON = MediaType.parse("application/json; charset=utf-8");
    public static MediaType XML = MediaType.parse("application/xml; charset=utf-8");

    private OkHttpUtils() {
    }

    public static String get(String url, Map<String, String> queries) {
        return get(url, (Map) null, queries);
    }

    public static String get(String url, Map<String, String> header, Map<String, String> queries) {
        String queriesStr = buildQueries(url, queries);
        Request.Builder builder = buildRequest(header);
        Request request = builder.url(queriesStr).build();
        return getBody(request);
    }


    public static String post(String url, Map<String, String> params) {
        return post(url, (Map) null, params);
    }

    public static String post(String url, Map<String, String> header, Map<String, String> params) {
        Request.Builder builder = buildRequest(header);
        Request request = builder.url(url).post(buildFormBody(params)).build();
        return getBody(request);
    }

    public static String postJson(String url, String json) {
        return postJson(url, (Map) null, json);
    }

    public static String postJson(String url, Map<String, String> header, String json) {
        return postContent(url, header, json, JSON);
    }

    public static String postXml(String url, String xml) {
        return postXml(url, (Map) null, xml);
    }

    public static String postXml(String url, Map<String, String> header, String xml) {
        return postContent(url, header, xml, XML);
    }

    public static String postContent(String url, Map<String, String> header, String content, MediaType mediaType) {
        RequestBody requestBody = RequestBody.create(mediaType, content);
        Request.Builder builder = new Request.Builder();
        if (header != null && header.keySet().size() > 0) {
            header.forEach(builder::addHeader);
        }

        Request request = builder.url(url).post(requestBody).build();
        return getBody(request);
    }

    private static String getBody(Request request) {
        String responseBody = "";
        Response response = null;

        String var4;
        try {
            OkHttpClient okHttpClient = getClient(request);
            response = okHttpClient.newCall(request).execute();
            if (!response.isSuccessful()) {
                return responseBody;
            }
            var4 = response.body().string();
        } catch (Exception var8) {
            log.error("okhttp3 post error >> ex = {}", var8.getMessage());
            return responseBody;
        } finally {
            if (response != null) {
                response.close();
            }
        }
        return var4;
    }

    private static OkHttpClient getClient(Request request) {
        int timeout = 30;
        String timeoutHeader = request.header("timeout");
        if (null != timeoutHeader && !timeoutHeader.isEmpty()) {
            timeout = Integer.parseInt(timeoutHeader);
        }
        OkHttpClient okHttpClient = new OkHttpClient.Builder()
                .connectTimeout(timeout, TimeUnit.SECONDS)
                .writeTimeout(timeout, TimeUnit.SECONDS)
                .readTimeout(timeout, TimeUnit.SECONDS)
                .build();
        return okHttpClient;
    }

    public static byte[] downloadPost(String url, Map<String, String> params) {
        return downloadPost(url, (Map) null, params);
    }

    public static byte[] downloadPost(String url, Map<String, String> header, Map<String, String> params) {
        Request.Builder builder = buildRequest(header);
        Request request = builder.url(url).post(buildFormBody(params)).build();
        return getBodyByte(request);
    }

    private static byte[] getBodyByte(Request request) {
        byte[] responseBytes = null;
        Response response = null;

        try {
            OkHttpClient okHttpClient = getClient(request);
            response = okHttpClient.newCall(request).execute();
            if (!response.isSuccessful()) {
                return responseBytes;
            }
            responseBytes = response.body().bytes();
        } catch (Exception var8) {
            log.error("okhttp3 post error >> ex = {}", var8.getMessage());
            return responseBytes;
        } finally {
            if (response != null) {
                response.close();
            }
        }
        return responseBytes;
    }

    public static RequestBody buildFormBody(Map<String, String> params) {
        FormBody.Builder formBuilder = new FormBody.Builder();
        if (params != null && params.keySet().size() > 0) {
            params.forEach(formBuilder::add);
        }
        return formBuilder.build();
    }

    public static Request.Builder buildRequest(Map<String, String> header) {
        Request.Builder builder = new Request.Builder();
        if (header != null && header.keySet().size() > 0) {
            header.forEach(builder::addHeader);
        }
        return builder;
    }

    public static String buildQueries(String url, Map<String, String> queries) {
        StringBuffer sb = new StringBuffer(url);
        if (queries != null && queries.keySet().size() > 0) {
            queries.forEach((k, v) -> {
                sb.append("&").append(k).append("=").append(v);
            });
        }
        return sb.toString();
    }
}

```

