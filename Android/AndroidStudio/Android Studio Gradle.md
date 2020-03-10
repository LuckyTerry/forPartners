# Android Studio Gradle 基本配置

标签（空格分隔）： 未分类

---

Issue 1 ： 下载依赖jar包非常缓慢
是因为 gradle 没有走代理。

如果是 socks5 代理 ，如下这样设置其实并没有什么卵用
systemProp.socks.proxyHost=127.0.0.1
systemProp.socks.proxyPort=7077
systemProp.https.proxyHost=127.0.0.1
systemProp.https.proxyPort=7077
systemProp.https.proxyHost=socks5://127.0.0.1
systemProp.https.proxyPort=7077

正确设置方法应该是这样：
org.gradle.jvmargs=-DsocksProxyHost=127.0.0.1 -DsocksProxyPort=1080

修改 $HOME/.gradle/gradle.properties 文件,加入上面那句，这样就可以全局开启 gradle 代理

如果发现项目中 gradle.properties 设置了正确的代理，但依旧失败，则需要看一看 $HOME/.gradle/gradle.properties 是不是设置错了，因为全局的会覆盖项目中的。因此强烈建议代理设置在 $HOME/.gradle/gradle.properties ，而不是项目中的 gradle.properties 。






