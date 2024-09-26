# Makefile 基础命令-语法

- 数据类型，字符串和字符串数组；
- 定义变量：定义变量 val，为 string 类型，值为folder；
    ```makefile
    val := folder
    ```
- 定义数组：定义变量 val，为数组类型，值是["hello", "world", "folder"]
    ```makefile
    val := hello world folder
    ```

- 定义的方式有多种：
    - = 赋值： val = folder，递归赋值，val的值赋值后并不马上生效，等到使用时才真正的赋值（不常用）；例子：
        ```makefile
            x = $(y)
            y = hello
        ```
       此时x的值是hello，可以这样理解：使用等号赋值的时候makefile在分析完整个文件后再把变量展开；
    - := 赋值：val := folder，基本赋值，当前所在位置决定取值（常用）；
    - ?= 赋值：val ?= folder，如果没有赋值，则赋值为 folder；
    - += 赋值：val += folder，添加值，在 val 后面添加值；例子：
        ```makefile
            val := hello
            val += world
        ```
        此时，val 的结果为 hello world；
- $(val)：解析 val 的值；
- $(func param)：调用 Make 提供的内置函数；
    - 例如：
        - 定义 val 的值为执行shell echo hello后的输出：
        ```makefile
            val := $(shell echo hello)
        ```
        - 直接打印 val 变量的内容：
        ```makefile
            $(info $(val))
        ```
        - 替换 aabbcc 种的 a 为 x 后赋值给 val，结果为 xxbbcc：
        ```makefile
           val := $(subst a, x, aabbcc)
        ```
        - 逻辑语法：ifeq、ifneq、ifdef、ifndef
        ```makefile
            ifeq($(val), depends)
                name := hello
            endif
        ```
# 案例
```makefile
    # 定义变量a, 赋值为 folder
    a := folder
    # 为变量 a 增加内容 directory, 结果是：folder directory
    a += directory
    # 定义变量 b, 为字符串拼接，结果是：beginfolder drectoryend
    b := begin$(a)end
    # 定义变量 c, 为执行 ls 指令后的输出字符串
    c := $(shell ls)
    # 定义变量 d, 为把 xiao 中的 x 替换为a, 结果是：aiao
    d := $(subst x, a, xiao)
    # 定义变量e, 为在 a 的每个元素前面增加 -L 符号, 结果是：-Lfolder -Ldirectory
    e := $(patsubst %, -L%, $(a))
    # 打印变量 e 的内容
    $(info e = ${e})
```
- 生成项可以没有依赖项，那么如果该生成项文件不存在，command将永远执行

# 依赖关系定义
- 依赖关系定义：
    ```makefile
        a.o: a.cpp
            @echo 编译 a.cpp, 生成 a.o
            g++ -c a.cpp a.o
    ```
    - 第一次执行make a.o时，由于a.o不存在，执行了command；
    - 第二次执行make a.o时，由于a.cpp时间没有比a.o新，打印a.o is up to date，不需要编译；
    - 生成项和依赖项，从来都是当成文件来看待的；

# 编译和链接结合起来

    - 根据 out.bin 推导其依赖是 a.o b.o
    - 根据 a.o 推导其依赖项是 a.cpp, 执行编译 a.cpp
    - 根据 b.o 推导其依赖项是 b.cpp, 执行编译 b.cpp
    - 有了 a.o 和 b.o 后, 链接成 out.bin

    ```makefile
        a.o: a.cpp
            @echo 编译 a.cpp, 生成 a.o
            g++ -c a.cpp a.o
        b.o: b.cpp
            @echo 编译 b.cpp, 生成 b.o
            g++ -c b.cpp b.o
        out.bin: a.o b.o
            @echo 链接 a.o, b.o, 生成 out.bin
            g++ a.o b.o -o out.bin
    ```
- 定义好依赖后make out.bin后，会自动查找依赖关系，并自动按照顺序执行command
- 这是makefile为我们解决的核心问题，剩下就是如何玩的更方便罢了。比如自动检索a.cpp、b.cpp，自动定义a.o依赖a.cpp。等等

# 总结
- 变量赋值有4种方式var = 123, var := 123, var ?= 123, var += 123。其中var := 123常用，var += 123常用
- 取变量值有两种，$(var)，${var}。小括号大括号均可以
- 数据类型只有字符串和字符串数组，空格隔开表示多个元素
- $(function arguments)是调用make内置函数的方法，具体可以参考官方文档的函数大全。但是常用的其实只有少数两个即可
- 依赖关系定义中，如果代码修改时间比生成的更新/生成不存在时，command会执行。否则只会打印main.o is up to date。这是makefile解决的核心问题
- 依赖关系可以链式的定义，即b依赖a，c依赖b，而make会自动链式的查找并根据时间执行command
- command是shell指令，可以使用$(var)来将变量用到其中。前面加@表示执行执行时不打印原指令内容。否则默认打印指令后再执行指令
- make不写具体生成名称，则会选择依赖关系中的第一项生成