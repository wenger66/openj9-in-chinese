# -XcompilationThreads

使用该参数来指定JIT编译的线程数 

## 语法

    -XcompilationThreads<n>
    
<n>为线程数，可选值为1-4，如果n设置为大于4的值，JVM会启动失败；如果n设置为0，并不会禁止JIT。如果试图禁止JIT，需要使用参数
[-Xint](-Xint.md)

## 解释

当使用多个线程编译时，JIT也许会生成多个诊断日志文件，有几个编译线程就有几个日志文件。第一个编译线程对应的日志文件格式如下：

    <specified_filename>.<date>.<time>.<pid>
    
第一个编译线程的ID是0。第二个和接下来的编译线程会将ID值添加到日志文件名的后缀中，文件格式如下：

    <specified_filename>.<date>.<time>.<pid>.<compilation_thread_ID>
    
举个例子，第二个编译线程的ID是1，该线程对应的日志文件名就是下面的格式：

    <specified_filename>.<date>.<time>.<pid>.1

