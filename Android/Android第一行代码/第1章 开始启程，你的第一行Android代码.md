# 第1章 开始启程，你的第一行Android代码

------

## 系统架构

> * Linux 内核层 (Linux Kernel)
> * 系统运行库层(Libraries) -- (Android Runtime)
> * 应用框架层(Application Framework)
> * 应用层(Applications)

 1. 为 Android 设备的各种硬件提供了底层的驱动，如显示驱动、音频驱动、照相机驱动、蓝牙驱动、Wi-Fi 驱动、电源管理等。
 2. 通过一些 C/C++库来为 Android系统提供了主要的特性支持。如SQLite库提供了数据库的支持，OpenGL|ES 库提供了 3D 绘图的支持，Webkit 库提供了浏览器内核的支持等。
 3. 提供了构建应用程序时可能用到的各种API，Android自带的一些核心应用就是使用这些API完成的，开发者也可以通过使用这些API来构建自己的应用程序。
 4. 所有安装在手机上的应用程序都是属于这一层的，比如系统自带的联系人、短信等程序，或者是你从 Google Play 上下载的小游戏，当然还包括你自己开发的程序

------

## 已发布的版本

<table>
    <tr>
        <th>版本号</th>
        <th>系统代号</th>
        <th>API</th>
    </tr>
    <tr>
        <td>2.2</td>
        <td>Froyo</td>
        <td>8</td>
    </tr>
    <tr>
        <td>2.3.3-2.3.7</td>
        <td>Gingerbread</td>
        <td>10</td>
    </tr>
    <tr>
        <td>3.2</td>
        <td>Honeycomb</td>
        <td>13</td>
    </tr>
    <tr>
        <td>4.0.3 – 4.0.4</td>
        <td>Ice Cream Sandwich</td>
        <td>15</td>
    </tr>
    <tr>
        <td>4.1.x</td>
        <th rowspan="3">Jelly Bean</th>
        <td>16</td>
    </tr>
    <tr>
        <td>4.2.x</td>
        <td>17</td>
    </tr>
        <td>4.3</td>
        <td>18</td>
    </tr>
    <tr>
        <td>4.4</td>
        <td>KitKat</td>
        <td>19</td>
    </tr>
    <tr>
        <td>5.0</td>
        <td>Lollipop</td>
        <td>21</td>
    </tr>
    <tr>
        <td>5.1</td>
        <td>Lollipop</td>
        <td>22</td>
    </tr>
    <tr>
        <td>6.0</td>
        <td>Marshmallow</td>
        <td>23</td>
    </tr>
    <tr>
        <td>7.0</td>
        <td>?</td>
        <td>24</td>
    </tr>
    
    
   
</table>

------

## 创建第一个Hello World!

略
