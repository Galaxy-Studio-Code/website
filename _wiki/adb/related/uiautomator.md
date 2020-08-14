# uiautomator 相关命令

* [获取当前页面布局](#获取当前页面布局)


### 获取当前页面布局
```sh
adb shell uiautomator dump /data/local/tmp/uidump.xml  && adb pull /data/local/tmp/uidump.xml uidump.xml
```
