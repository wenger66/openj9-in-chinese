# -Xcompressedrefs

   **只能在64位虚拟机上使用**

* 作用：启用或禁用引用压缩
* 限制：参数文件中该参数无效

## 语法

|Setting	|Action	|Default|
| --------   | -----:   | :----: |
|-Xcompressedrefs|	启用引用压缩|	请参考[默认行为](#默认行为)|
|-Xnocompressedrefs	|禁用引用压缩||

## 默认行为
当-Xmx参数设置小于57G时，默认启用引用压缩特性

z/OS®: This threshold value assumes that you have APAR OA49416 installed. If you do not have the APAR installed, the threshold value is 25 GB.

AIX® and Linux®: For the metronome garbage collection policy, the threshold is 25 GB.

