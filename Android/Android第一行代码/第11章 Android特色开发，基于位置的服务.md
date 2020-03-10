# 第11章 Android特色开发，基于位置的服务

------

## LocationManager的基本用法

权限

    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

使用

    locationManager = (LocationManager) getSystemService(Context.LOCATION_SERVICE);
    // 获取所有可用的位置提供器
    List<String> providerList = locationManager.getProviders(true);
    if (providerList.contains(LocationManager.GPS_PROVIDER)) {
        provider = LocationManager.GPS_PROVIDER;
    } else if (providerList.contains(LocationManager.NETWORK_PROVIDER)) {
        provider = LocationManager.NETWORK_PROVIDER;
    } else {
        // 当没有可用的位置提供器时，弹出Toast提示用户
        Toast.makeText(this, "No location provider to use", Toast.LENGTH_SHORT).show();
        return;
    }
    Location location = locationManager.getLastKnownLocation(provider);
    if (location != null) {
        // 显示当前设备的位置信息
        showLocation(location);
    }
    locationManager.requestLocationUpdates(provider, 5000, 1, locationListener);
    
    LocationListener locationListener = new LocationListener() {
        @Override
        public void onStatusChanged(String provider, int status, Bundle extras) {
        }
        @Override
        public void onProviderEnabled(String provider) {
        }
        @Override
        public void onProviderDisabled(String provider) {
        }
        @Override
        public void onLocationChanged(Location location) {
            // 更新当前设备的位置信息
            showLocation(location);
        }
    };
    private void showLocation(Location location) {
        String currentPosition = "latitude is " + location.getLatitude() + "\n"
                        + "longitude is " + location.getLongitude();
        positionTextView.setText(currentPosition);
    }

使用完毕后关闭

    if (locationManager != null) {
        // 关闭程序时将监听器移除
        locationManager.removeUpdates(locationListener);
    }

------

## Google方向地理编码

Geocoding API 中规定了很多接口，其中反向地理编码的接口如下：

    http://maps.googleapis.com/maps/api/geocode/json?latlng=40.714224,-73.961452&sensor=true_or_false

json 表示希望服务器能够返回 JSON 格式的数据，这里也可以指定成 xml；
latlng=40.714224,-73.96145表示传递给服务器去解码的经纬值是北纬40.714224度，西经73.96145度；
sensor=true_or_false表示这条请求是否来自于某个设备的位置传感器，通常指定成 false 即可。

示列 中国北京市西城区

    http://maps.googleapis.com/maps/api/geocode/json?latlng=39.905547,116.39145&sensor=false

示列代码

    <uses-permission android:name="android.permission.INTERNET" />

    try {
        // 组装反向地理编码的接口地址
        StringBuilder url = new StringBuilder();
        url.append("http://maps.googleapis.com/maps/api/geocode/json?latlng=");
        url.append(location.getLatitude()).append(",")
        url.append(location.getLongitude());
        url.append("&sensor=false");
        HttpClient httpClient = new DefaultHttpClient();
        HttpGet httpGet = new HttpGet(url.toString());
        // 在请求消息头中指定语言，保证服务器会返回中文数据
        httpGet.addHeader("Accept-Language", "zh-CN");
        HttpResponse httpResponse = httpClient.execute(httpGet);
        if (httpResponse.getStatusLine().getStatusCode() == 200) {
            HttpEntity entity = httpResponse.getEntity();
            String response = EntityUtils.toString(entity, "utf-8");
            JSONObject jsonObject = new JSONObject(response);
            // 获取results节点下的位置信息
            JSONArray resultArray = jsonObject.getJSONArray("results");
            if (resultArray.length() > 0) {
                JSONObject subObject = resultArray.getJSONObject(0);
                // 取出格式化后的位置信息
                String address = subObject.getString("formatted_address");
            }
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
    
------

## 百度地图

### 1. 创建应用

#### 百度地图开放平台 API控制台

    http://lbsyun.baidu.com/apiconsole/key

#### 创建应用

    略

#### 应用名称 

    测试

#### 应用类型 

    Android SDK

#### 发布版SHA1

方法一：使用jdk自带工具keytool

     1. 控制台，cmd
     2. 输入cd .android，定位到.android文件夹下
     3. 输入keytool -list -v -keystore debug.keystore
     4. 输入android(密钥口令是android)
     5. 会得到三种指纹证书，选取SHA1类型的证书

示列结果如下

    密钥库类型: JKS
    密钥库提供方: SUN
    
    您的密钥库包含 1 个条目
    
    别名: androiddebugkey
    创建日期: 2016-8-10
    条目类型: PrivateKeyEntry
    证书链长度: 1
    证书[1]:
    所有者: C=US, O=Android, CN=Android Debug
    发布者: C=US, O=Android, CN=Android Debug
    序列号: 1
    有效期开始日期: Wed Aug 10 17:22:54 CST 2016, 截止日期: Fri Aug 03 17:22:54 CST 2046
    证书指纹:
             MD5: 44:C2:CA:63:EF:DD:F4:A7:78:F7:CE:C0:0A:DF:7F:25
             SHA1: 93:54:FF:F9:21:08:4E:17:C3:AC:29:4E:45:F8:CD:A4:3A:08:86:12
             SHA256: 5B:8B:5C:5C:98:09:28:CA:58:25:F2:E9:BB:E1:4E:55:9A:40:77:30:A1:49:5A:7D:D3:D6:E1:56:0A:B7:AC:61
             签名算法名称: SHA1withRSA
             版本: 1


#### 包名

    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
              package="com.terry.imageloaderdemo">
              
    即 com.terry.imageloaderdemo

#### 提交得到安全码

    93:54:FF:F9:21:08:4E:17:C3:AC:29:4E:45:F8:CD:A4:3A:08:86:12;com.terry.imageloaderdemo

### 2. 使用方法

参考如下网址的帮助文档


> 百度地图开放平台首页

    http://lbsyun.baidu.com/index.php?title=首页

> Android定位服务 开发指南

    http://lbsyun.baidu.com/index.php?title=android-locsdk

> Android地图服务 开发指南

    http://lbsyun.baidu.com/index.php?title=androidsdk/guide/basicmap

> Android出行服务 开发指南

    http://lbsyun.baidu.com/index.php?title=android-navsdk