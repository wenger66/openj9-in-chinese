# Java dump (https://www.eclipse.org/openj9/docs/dump_javadump/)
## 格式
### 文件格式
Java dump 通常是文本格式(.txt)，因此可以通过一般的文本编辑器进行阅读，阅读时需要注意段与行的格式
### 段落格式
* 每一段的开头，都会用“-----”与上一段明显的区分开来
* 每一段的标题也会用“=====”作为标识
### 行格式
每一行都包含一个标签，这个标签最多由 15 个字符组成。

其中第一位数字代表信息的详细级别（0，1，2，3，4），级别越高代表信息越详细

接下来的两个字符是段标题的缩写，比如
* CI - Command-line interpreter
* CL - Class loader
* LK - Locking
* ST - Storage
* TI - Title
* XE - Execution engine

其余部分为信息的概述

## 内容
Java dump汇总了事件发生时虚拟机的状态，包括大部分虚拟机组件的信息，dump文件由多个部分组成，每个部分提供了不同的信息

感觉段落用英文更加专业，无歧义
### TITLE 
Java dump的第一段是触发dump的事件信息。下面的例子中，你可以看到是vmstop事件在2018/08/30 21:55:47这个时间触发了这个dump

    0SECTION       TITLE subcomponent dump routine
    NULL           ===============================
    1TICHARSET     UTF-8
    1TISIGINFO     Dump Event "vmstop" (00000002) Detail "#0000000000000000" received
    1TIDATETIME    Date: 2018/08/30 at 21:55:47:607
    1TINANOTIME    System nanotime: 22012355276134
    1TIFILENAME    Javacore filename:    /home/doc-javacore/javacore.20180830.215547.30285.0001.txt
    1TIREQFLAGS    Request Flags: 0x81 (exclusive+preempt)
    1TIPREPSTATE   Prep State: 0x106 (vm_access+exclusive_vm_access+trace_disabled)  


