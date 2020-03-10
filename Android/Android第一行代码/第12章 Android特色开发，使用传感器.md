# 第12章 Android特色开发，使用传感器

标签（空格分隔）： 未分类

---

## 传感器简介
Android 手机通常都会支持多种类型的传感器，如光照传感右器、加速度传感器、地磁传感器、压力传感器、温度传感器等。本章中我们只去学习最常见的几种传感器的用法，中每个传感器的用法其实都比较类似。

> * 获取到 SensorManager 的实例
> * 调用 getDefaultSensor()方法来得到任意的传感器类型
> * 借助 SensorEventListener 来对传感器输出的信号进行监听
> * 调用 SensorManager 的 registerListener()方法来注册 SensorEventListener
> * 调用 SensorManager 的 unregisterListener()方法来释放掉资源

```java
SensorManager senserManager = (SensorManager)getSystemService(Context.SENSOR_SERVICE);
Sensor sensor = senserManager.getDefaultSensor(Sensor.？);
SensorEventListener listener = new SensorEventListener() {
    @Override
    public void onAccuracyChanged(Sensor sensor, int accuracy) {
    }
    @Override
    public void onSensorChanged(SensorEvent event) {
    }
};
senserManager.registerListener(listener, senser, SensorManager.SENSOR_DELAY_NORMAL);

sensorManager.unregisterListener(listener);
```
registerListener第三个参数是用于表示传感器输出信息的更新速率，共有SENSOR_DELAY_UI、SENSOR_DELAY_NORMAL、SENSOR_DELAY_GAME 和 SENSOR_DELAY_FASTEST 这四种值可选，它们的更新速率是
依次递增的

## 光照传感器

### 1.光照传感器的用法

    SensorManager senserManager = (SensorManager)getSystemService(Context.SENSOR_SERVICE);
    Sensor sensor = senserManager.getDefaultSensor(Sensor.TYPE_LIGHT);
    SensorEventListener listener = new SensorEventListener() {
        @Override
        public void onAccuracyChanged(Sensor sensor, int accuracy) {
        }
        @Override
        public void onSensorChanged(SensorEvent event) {
        }
    };
    senserManager.registerListener(listener, senser, SensorManager.SENSOR_DELAY_NORMAL);
    
    sensorManager.unregisterListener(listener);

### 2.简易光照探测器
新建一个 LightSensorTest 项目，修改 activity_main.xml 中的代码，如下所示：

    <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent" >
        <TextView
            android:id="@+id/light_level"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerInParent="true"
            android:textSize="20sp" />
    </RelativeLayout>

修改 MainActivity 中的代码

    public class MainActivity extends Activity {
        private SensorManager sensorManager;
        private TextView lightLevel;
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            lightLevel = (TextView) findViewById(R.id.light_level);
            sensorManager = (SensorManager) getSystemService(Context.SENSOR_SERVICE);
            Sensor sensor = sensorManager.getDefaultSensor(Sensor.TYPE_LIGHT);
            sensorManager.registerListener(listener, sensor, SensorManager.SENSOR_DELAY_NORMAL);
        }
        @Override
        protected void onDestroy() {
            super.onDestroy();
            if (sensorManager != null) {
            sensorManager.unregisterListener(listener);
            }
        }
        private SensorEventListener listener = new SensorEventListener() {
            @Override
            public void onSensorChanged(SensorEvent event) {
                // values数组中第一个下标的值就是当前的光照强度
                float value = event.values[0];
                lightLevel.setText("Current light level is " + value + " lx");
            }
            @Override
            public void onAccuracyChanged(Sensor sensor, int accuracy) {
            }
        };
    }

## 加速度传感器

### 1.光照传感器的用法

    SensorManager senserManager = (SensorManager)getSystemService(Context.SENSOR_SERVICE);
    Sensor sensor = senserManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER);
    SensorEventListener listener = new SensorEventListener() {
        @Override
        public void onAccuracyChanged(Sensor sensor, int accuracy) {
        }
        @Override
        public void onSensorChanged(SensorEvent event) {
        }
    };
    senserManager.registerListener(listener, senser, SensorManager.SENSOR_DELAY_NORMAL);
    
    sensorManager.unregisterListener(listener);
    
### 2.模仿微信摇一摇
新建一个 AccelerometerSensorTest 项目，然后修改 MainActivity 中的代码

    public class MainActivity extends Activity {
        private SensorManager sensorManager;
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            sensorManager = (SensorManager) getSystemService(Context.SENSOR_SERVICE);
            Sensor sensor = sensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER);
            sensorManager.registerListener(listener, sensor, SensorManager.SENSOR_DELAY_NORMAL);
        }
        @Override
        protected void onDestroy() {
            super.onDestroy();
            if (sensorManager != null) {
                sensorManager.unregisterListener(listener);
            }
        }
        private SensorEventListener listener = new SensorEventListener() {
            @Override
            public void onSensorChanged(SensorEvent event) {
                // 加速度可能会是负值，所以要取它们的绝对值
                float xValue = Math.abs(event.values[0]);
                float yValue = Math.abs(event.values[1]);
                float zValue = Math.abs(event.values[2]);
                if (xValue > 15 || yValue > 15 || zValue > 15) {
                // 认为用户摇动了手机，触发摇一摇逻辑
                Toast.makeText(MainActivity.this, "摇一摇",
                Toast.LENGTH_SHORT).show();
                }
            }
            @Override
            public void onAccuracyChanged(Sensor sensor, int accuracy) {
            }
        };
    }

## 方向传感器

### 1.光照传感器的用法

    SensorManager senserManager = (SensorManager)getSystemService(Context.SENSOR_SERVICE);
    Sensor accelerometerSensor = sensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER);
    Sensor magneticSensor = sensorManager.getDefaultSensor(Sensor.TYPE_MAGNETIC_FIELD);
    SensorEventListener listener = new SensorEventListener() {
        @Override
        public void onAccuracyChanged(Sensor sensor, int accuracy) {
        }
        @Override
        public void onSensorChanged(SensorEvent event) {
            // 判断当前是加速度传感器还是地磁传感器
            if (event.sensor.getType() == Sensor.TYPE_ACCELEROMETER) {
                // 注意赋值时要调用clone()方法
                accelerometerValues = event.values.clone();
            } else if (event.sensor.getType() == Sensor.TYPE_MAGNETIC_FIELD) {
                // 注意赋值时要调用clone()方法
                magneticValues = event.values.clone();
            }
            float[] R = new float[9];
            float[] values = new float[3];
            SensorManager.getRotationMatrix(R, null, accelerometerValues,magneticValues);
            SensorManager.getOrientation(R, values);
        }
    };
    sensorManager.registerListener(listener,accelerometerSensor,SensorManager.SENSOR_DELAY_GAME);
    sensorManager.registerListener(listener, magneticSensor,SensorManager.SENSOR_DELAY_GAME);
    
    sensorManager.unregisterListener(listener);

##### 1.分别获取到加速度传感器和地磁传感器的实例，并给它们注册监听器;     
##### 2.onSensorChanged()方法中可以获取到 SensorEvent 的 values 数组，分别记录着加 速 度 传 感 器 和 地 磁 传 感 器 输 出 的 值 。 然 后 将 这 两 个 值 传 入 到 SensorManager的getRotationMatrix()方法中就可以得到一个包含旋转矩阵的 R 数组;
##### 3.调用 SensorManager 的 getOrientation()方法来计算手机的旋转数据了;
##### 4.其中 values[0]记录着手机围绕着图 12.3 中 Z 轴的旋转弧度，values[1]记录着手机围绕 X 轴的旋转弧度，values[2]记录着手机围绕 Y 轴的旋转弧度,如果你想将它们转换成角度还需要调用如下方法：Math.toDegrees(values[0]);


