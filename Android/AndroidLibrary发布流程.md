# Android Library 发布流程

## 发布到本地maven仓库

1. Project根目录下gradle.properties中添加一下信息

    ```text
    #包信息  需要修改成你的
    GROUP_ID = com.kcode.github.pubutils

    # Licence信息  基本是固定的
    PROJ_LICENCE_NAME=The Apache Software License, Version 2.0
    PROJ_LICENCE_URL=http://www.apache.org/licenses/LICENSE-2.0.txt
    PROJ_LICENCE_DEST=repo
    ```

2. Library根目录下新建gradle.properties文件

    ```text
    #包信息
    ARTIFACTID = androidLib
    LIBRARY_VERSION = 0.0.2

    #Mac下地址:file:///Users/<username>/my/local/repo
    #Ubuntu下地址:file:///home/<username>/.m2/repository/
    LOCAL_REPO_URL=file:///home/terry/.m2/repository/
    ```

3. Library根目录下的gradle.build中添加

    ```text
    apply plugin: 'maven'
    uploadArchives{
        repositories.mavenDeployer{
            repository(url:LOCAL_REPO_URL)
            pom.groupId = GROUP_ID
            pom.artifactId = ARTIFACTID
            pom.version = LIBRARY_VERSION
        }
    }
    ```

4. 在项目路径下执行

    ```text
    ./gradlew -p yourLibName clean build uploadArchives --info
    ```

## 在其他项目中引用

1. 先在最外层的gradle.build中添加本地maven库路径

    ```gradle
    maven { url 'file:///home/terry/.m2/repository/' }
    ```

2. 在需要引用这个库的项目的gradle.build中添加

    ```gradle
    compile 'com.xxx.xxx:yourLibName:version'
    ```

## 参考文章

[Android Studio 将Library 发布到本地maven仓库并在项目中引用(转)](https://www.jianshu.com/p/b53322daf8f2)
