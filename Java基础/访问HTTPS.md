

TestHttps:

```java
import org.apache.commons.io.IOUtils;

import javax.net.ssl.*;
import java.io.*;
import java.net.MalformedURLException;
import java.net.ProtocolException;
import java.net.URL;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.security.KeyManagementException;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;

/**
 * 功能描述：
 *
 * @author Limy
 * @since 2021/6/19 1:26
 */
public class TestHttps {
    public static void main(String[] args) {
        String url = "https://repo1.maven.org/maven2/com/github/j5ik2o/docker-controller-scala-scalatest_3/1.1.37/docker-controller-scala-scalatest_3-1.1.37.jar";
        httpsRequest(url, "GET");
    }

    /*
     * 处理https GET/POST请求
     * 请求地址、请求方法、参数
     * */
    public static void httpsRequest(String requestUrl, String requestMethod) {
        try {
            //创建SSLContext
            SSLContext sslContext = SSLContext.getInstance("SSL");
            TrustManager[] tm = {new MyX509TrustManager()};
            //初始化
            sslContext.init(null, tm, new SecureRandom());
            //获取SSLSocketFactory对象
            SSLSocketFactory ssf = sslContext.getSocketFactory();
            URL url = new URL(requestUrl);
            HttpsURLConnection conn = (HttpsURLConnection) url.openConnection();
            conn.setDoOutput(true);
            conn.setDoInput(true);
            conn.setUseCaches(false);
            conn.setRequestMethod(requestMethod);
            //设置当前实例使用的SSLSoctetFactory
            conn.setSSLSocketFactory(ssf);
            conn.connect();

            //读取服务器端返回的内容
            try (InputStream is = conn.getInputStream();
                 OutputStream os = Files.newOutputStream(Paths.get("E:\\tmp", "docker-controller-scala-scalatest_3-1.1.37.jar"))) {
                IOUtils.copy(is, os);
            }
        } catch (IOException | NoSuchAlgorithmException | KeyManagementException exception) {
            exception.printStackTrace();
        }
    }

    static class MyX509TrustManager implements TrustManager, X509TrustManager {
        @Override
        public void checkClientTrusted(X509Certificate[] x509Certificates, String s) throws CertificateException {

        }

        @Override
        public void checkServerTrusted(X509Certificate[] x509Certificates, String s) throws CertificateException {

        }

        @Override
        public X509Certificate[] getAcceptedIssuers() {
            return new X509Certificate[0];
        }
    }
}

```



TestHttpsClient:

```java
import org.apache.commons.httpclient.HttpClient;
import org.apache.commons.httpclient.methods.GetMethod;
import org.apache.commons.httpclient.methods.PostMethod;

import java.io.IOException;

/**
 * 功能描述：
 *
 * @author Limy
 * @since 2021/6/19 10:23
 */
public class TestHttpClient {
    public static void main(String[] args) {
        HttpClient client = new HttpClient();
        GetMethod getMethod = new GetMethod("https://www.baidu.com/");
//        PostMethod postMethod = new PostMethod("http://java.sun.com");
        try {
            client.executeMethod(getMethod);
            System.out.println(getMethod.getStatusCode());
            // 需要进行解码
            System.out.println(getMethod.getResponseBodyAsString());
        } catch (IOException exception) {
            exception.printStackTrace();
        } finally {
            getMethod.releaseConnection();
        }
    }
}

```

