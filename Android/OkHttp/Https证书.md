# Https配置

## 证书类型

bks

cer

jks

## BKS证书

1. 准备.cer文件

    点击网站网址栏前的小锁按钮，选择详细信息，选择view certificate。显示证书之后，点击详细信息，然后一直下一步，直到导出.cer文件。

2. 将.cer转换为.bks

    下载特定版本的JCE Provider包

    命令行输入以下命令

    ```bash
    keytool -importcert -v -trustcacerts -alias 位置1 \
    -file 位置2 \
    -keystore 位置3 -storetype BKS \
    -providerclass org.bouncycastle.jce.provider.BouncyCastleProvider \
    -providerpath 位置4 -storepass 位置5
    ```

    位置1:是个随便取的别名
    位置2:cer或crt证书的全地址
    位置3:生成后bks文件的位置,建议写全地址
    位置4:上面下载JCE Provider包的位置
    位置5:生成后证书的密码。下边获取sslsocketfactory中会用到密码

    ```bash
    keytool -importcert -v -trustcacerts -alias xx -file E:\bks\xx.cer -keystore E:\bks\xx.bks -storetype BKS -providerclass org.bouncycastle.jce.provider.BouncyCastleProvider -providerpath E:\bks\bcprov-jdk15on-146.jar -storepass xxxxxx
    ```

    成功之后会在你指定的位置生成bks文件.然后将文件放到项目raw目录下。

3. 获取SSLSocketFactory

    password和设置keystore的bks类型一定不要搞错

    ```java
    /**
    * 获取bks文件的sslsocketfactory
    * @param context
    * @return
    */
    public static SSLSocketFactory getSSLSocketFactory(Context context) {
        final String CLIENT_TRUST_PASSWORD = "123456";//信任证书密码，该证书默认密码是123456
        final String CLIENT_AGREEMENT = "TLS";//使用协议
        final String CLIENT_TRUST_KEYSTORE = "BKS";
        SSLContext sslContext = null;
        try {
            //取得SSL的SSLContext实例
            sslContext = SSLContext.getInstance(CLIENT_AGREEMENT);
            //取得TrustManagerFactory的X509密钥管理器实例
            TrustManagerFactory trustManager = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
            //取得BKS密库实例
            KeyStore tks = KeyStore.getInstance(CLIENT_TRUST_KEYSTORE);
            InputStream is = context.getResources().openRawResource(R.raw.traint);
            try {
                tks.load(is, CLIENT_TRUST_PASSWORD.toCharArray());
            } finally {
                is.close();
            }
            //初始化密钥管理器
            trustManager.init(tks);
            //初始化SSLContext
            sslContext.init(null, trustManager.getTrustManagers(), null);
        } catch (Exception e) {
            e.printStackTrace();
            Log.e("SslContextFactory", e.getMessage());
        }
        return sslContext.getSocketFactory();
    }
    ```

4. 配置OkHttp

    ```java
    OkHttpClient client = new okhttp3.OkHttpClient.Builder()
                .addInterceptor(new HttpLoggingInterceptor().setLevel(HttpLoggingInterceptor.Level.BODY))
                .sslSocketFactory(HTTPSUtils.getSSLSocketFactory(context))
                //.hostnameVerifier(HTTPSUtils.getHostNameVerifier(hostUrls))
                .readTimeout(10, TimeUnit.SECONDS)
                .connectTimeout(10, TimeUnit.SECONDS)
                .build();
    ```

5. 参考博客

[Retrofit Https踩坑记录](https://www.jianshu.com/p/41bb549317ff)

[(张鸿洋的，需要看完整理)Android Https相关完全解析 当OkHttp遇到Https](https://blog.csdn.net/lmj623565791/article/details/48129405)

[retrofit遇上https自签名证书](https://blog.csdn.net/u013768203/article/details/72874242)

[Retrofit系列之HTTPS信任所有证书](https://www.jianshu.com/p/a9f425594349)

[OkHttp加载https的配置](https://www.jianshu.com/p/c146ea1fc1ca)