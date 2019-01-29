## -Xjit

可以通过该选项来控制JIT编译器的行为。

设置-Xjit选项但不携带任何参数，JIT编译器将以默认行为运行。

设置-Xnojit选项会禁用JIT编译器，但不会影响AOT编译器。

### 语法

|选项	|行为	|是否默认|
| --------   | -----:   | :----: |
|-Xjit	|启用JIT编译器	|是|
|-Xjit[:<parameter>=<value>{,<parameter>=<value>}]|	启用JIT编译器并携带参数	||
|-Xnojit|	禁用JIT编译器	  ||  

### 参数

|参数|	效果|
| --------   | -----:   | 
|count	|Forces compilation of methods on first invocation.|
|disableRMODE64	|Allows the JIT to allocate executable code caches above the 2 GB memory bar.|
|exclude	|Excludes the specified method from compilation.|
|<limitFile>|	Compile methods that are listed in the limit file.|
|optlevel	|Forces the JIT compiler to compile all methods at a specific optimization level.|
|verbose|	Reports information about the JIT and AOT compiler configuration and method compilation.|
|vlog	|Sends verbose output to a file.|