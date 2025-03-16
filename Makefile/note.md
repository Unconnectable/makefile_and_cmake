```makefile
目标:依赖
	执行语句
```

```sh
$ ls -l
Makefile
main.c
#下面是Makefile的命令
```

```makefile
# 这是一个注释
# 目标文件 生成的文件名 : 依赖文件 所需的源文件
main: entry.c bar.c
    gcc -o main main.c # 编译命令


```

```sh
$ ls -l
Makefile
bar.c
entry.c
main
#增添了一些文件
```

```makefile
main: entry.c bar.c
		gcc entry.c bar.c -o main

```

```sh
$ tree
├── entry.c
├── func
│   ├── bar.c
│   └── bar.h
└── makefile

#entry.c 调用了bar.h 的函数
```

## 多行注释和不输出命令

```makefile

正常注释:
#line
#line2
#line3

#多行注释 \
这行也被注释 \
这一行也被注释 \
也就是再跟着上一行的地方接上\这个换行符号 \
要保证第一行是正常注释"#"开头

```

---

## 不输出原始命令,只输出具体结果:

通常的 `makefile`

```makefile
target:dependnece
	command
$ make

#输出
command的具体命令 echo "Hello World"
Hello World #实际执行内容
```

处理方法:

1. 加上"@"符号

```makefile
clean_file:
	touch clean_file

clean:
	@rm -f clean_file
#再make clean的时候就不会输出具体内容
```

2. 所有命令之前加上 ``.SILENT:``都不会输出原始的命令内容

```makefile
.SILENT:

xxxx
xxxx
```



```makefile
clean_file:
	touch clean_file

clean:
	@rm -f clean_file
```

运行 make 的时候只会默认执行`touch clean_file`而不会执行`clean`,也就是说他只会创建文件而不是执行`clean`的命令
需要手动执行`make clean`

来看这个

```makefile
# 默认目标是构建
all: build

# 构建目标
build:
	@echo "Building the project..."
	# 这里放构建命令,例如 gcc -o main main.c

# 清理目标
clean:
	@echo "Cleaning the project..."
	rm -f clean_file main

# 代码风格检查目标
tidy:
	@echo "Running code style check..."
	# 这里放代码风格检查命令,例如 clang-format --dry-run --Werror *.c

# 格式化代码目标
format:
	@echo "Formatting code..."
	# 这里放代码格式化命令,例如 clang-format -i *.c
```

定义变量

```makefile
.SILENT:
x := dude
y := Hello,Sharon
all:
	echo $(x)
	echo ${x}
	# Bad practice, but works
	echo $x

	echo ${y}
	echo $(y)
	echo $y
```

## 目标(Targets)

执行所有的 Target

```makefile
all : Hello World Sharon

Hello:
	touch Hello.rs
World:
	touch World.rs
Sharon:
	touch Sharon.rs
```

### 执行多个目标

```makefile
# $@ 是一个自动变量,包含目标名称
all: hello Sharon

hello Sharon:
	echo $@
```

# 自动变量和通配符

##  \* 通配符 wildcard

``$?``表示所有比目标更新的依赖文件(即那些被修改过的 .c 文件)**

```makefile
print: $(wildcard *.c)
	ls -la  $?
```

`$(wildcard *.o)` 是一个函数调用 如果不加上$符号,`(wildcard *.o)`会被认为是一个普通文本,不是函数调用

```makefile
thing_wrong := *.o  #没有加上wildcard 表示名字为*.o的普通文件 有错误
thing_right := $(wildcard *.o)


all: one two three four

one: $(thing_wrong) #执行错误

two: *.o

three: $(thing_right)

four: $(wildcard *.o)

```

### 自动变量

### `Makefile` 自动变量

| 自动变量 | 含义           | 示例代码  | 运行结果(假设目标为 `output.txt`,依赖为 `input.txt`) |
| :------- | :------------- | :-------- | :--------------------------------------------------- |
| `$@`     | 当前目标的名字 | `echo $@` | `output.txt`                                         |
| `$<`     | 第一个依赖文件 | `echo $<` | `input.txt`                                          |
| `$^`     | 所有依赖文件   | `echo $^` | `input.txt`                                          |
| `$?`     | 更新的依赖文件 | `echo $?` | 如果 `input.txt` 比 `output.txt` 新,输出 `input.txt` |
| `$*`     | 目标的主干部分 | `echo $*` | `output`(去掉后缀 `.txt`)                            |

```makefile
#这里保证 src/input1.txt src/input2.txt存在 否则会报错
output.txt: src/input1.txt src/input2.txt
    @echo "Target: $@"
    @echo "第一个依赖文件: $<"
    @echo "所有依赖文件: $^"
    @echo "更新的依赖文件: $?"
    @echo "主干: $*"
    @echo "依赖文件的目录部分: $(^D)"
    @echo "依赖文件的文件名部分: $(^F)"
    cat $^ > $@
#输出如下
目标: output.txt
第一个依赖文件: src/input1.txt
所有依赖文件: src/input1.txt src/input2.txt
更新的依赖文件: src/input1.txt src/input2.txt
主干: output
依赖文件的目录部分: src src
依赖文件的文件名部分: input1.txt input2.txt
```

---

# Fancy Rules

## 隐式规则(Implicit Rules)

### 简写介绍

**CC:C** 编译器(默认是 gcc)

```makefile
CC = gcc
CC = clang
```

**CXX**: C++ 编译器 (默认是 g++).

```makefile
CXX = g++       # 默认是 g++
CXX = clang++   # 默认是 g++
```

**CFLAGS**: 传递给 C 编译器的额外标志 (例如 `-g` 表示生成调试信息).

```makefile
CFLAGS = -g -Wall -O2 -std=c11   # 示例
```

**CXXFLAGS**: 传递给 C++ 编译器的额外标志.



```makefile
CXXFLAGS = -g -Wall -O2 -std=c++17   # 示例
```

常用选项:

​	启用调试信息: `-g`

​	启用所有警告: `-Wall -Wextra`

​	优化级别: `-O2`

​	指定标准 (如 C++17): `-std=c++17`

**CPPFLAGS**: 传递给 C 预处理器的额外标志.

```
CPPFLAGS = -DDEBUG -Iinclude   # 示例
```

常用选项:

​	定义宏: `-DDEBUG`

​	添加包含目录: `-Iinclude`

**LDFLAGS**: 传递给链接器的额外标志.

```makefile
LDFLAGS = -Llib -pthread   # 示例
```

常用选项:

​	添加库目录: `-Llib`

​	指定链接选项: `-pthread`

**LOADLIBES** 和 **LDLIBS**: 链接的库.

```makefile
LDLIBS = -lm -lfoo   # 示例
```

常用选项:

​	链接数学库: `-lm`

​	链接动态库: `-lfoo`

### 隐式规则的触发条件

1. **目标文件没有显式规则**: 如果 Make 发现目标文件 (例如 `blah` 或 `blah.o`) 没有对应的显式规则,它会尝试使用隐式规则.
2. **依赖文件存在**: 如果目标文件依赖于某个源文件 (例如 `blah.o` 依赖于 `blah.c`),并且该源文件存在,Make 会使用隐式规则来生成目标文件.

### 常见的隐式规则

**编译 C 程序**: 如果存在 `n.c` 文件,Make 会自动生成 `n.o` 文件.

```makefile
$(CC) -c $(CPPFLAGS) $(CFLAGS) $^ -o $@
```

**编译 C++ 程序**: 如果存在 `n.cc` 或 `n.cpp` 文件,Make 会自动生成 `n.o` 文件.

```makefile
$(CXX) -c $(CPPFLAGS) $(CXXFLAGS) $^ -o $@
```

### 示例

```makefile

CC = gcc       # 使用 gcc 作为 C 编译器
CFLAGS = -g    # 启用调试信息

# 隐式规则 #1: 通过 C 链接器隐式规则生成可执行文件 blah
# 隐式规则 #2: 通过 C 编译隐式规则生成 blah.o,因为 blah.c 存在
blah: blah.o

# 生成 blah.c 文件
blah.c:
	echo "int main() { return 0; }" > blah.c

# 清理生成的文件
clean:
	rm -f blah*
```

```makefile
# 编译器设置
更完整的示例
CC = gcc
CXX = g++

# 编译选项
CFLAGS = -g -Wall -O2 -std=c11
CXXFLAGS = -g -Wall -O2 -std=c++17
CPPFLAGS = -DDEBUG -Iinclude

# 链接选项
LDFLAGS = -Llib -pthread
LDLIBS = -lm -lfoo

# 目标
all: my_program

# 编译 C 源文件
my_program: main.o utils.o
	$(CC) $(LDFLAGS) $^ $(LDLIBS) -o $@

main.o: main.c
	$(CC) $(CPPFLAGS) $(CFLAGS) -c $< -o $@

utils.o: utils.c
	$(CC) $(CPPFLAGS) $(CFLAGS) -c $< -o $@

# 清理
clean:
	rm -f *.o my_program

```

```sh

find . -maxdepth 1 -type f ! -name 'filename_' -exec rm -f {} +
```

## 静态模式规则(Static Pattern Rules)

### 语法

```makefile
targets...: target-pattern: prereq-patterns ...
    commands
```

通常使用通配符去实现目标模式和依赖模式.

​	**目标**: `foo.o bar.o all.o`

​	**目标模式**: `%.o`,匹配所有 `.o` 文件.

​	**依赖模式**: `%.c`,将 `%` 替换为匹配的主干 (例如 `foo`),生成依赖文件 `foo.c`.

​	**命令**: 编译 `.c` 文件生成 `.o` 文件.

### 手动编写规则的示例

```makefile

objects = foo.o bar.o all.o
all: $(objects)
	$(CC) $^ -o all

#手动写三次数
foo.o: foo.c
	$(CC) -c foo.c -o foo.o

bar.o: bar.c
	$(CC) -c bar.c -o bar.o

all.o: all.c
	$(CC) -c all.c -o all.o

all.c:
	echo "int main() { return 0; }" > all.c

%.c:
	touch $@

clean:
	rm -f *.c *.o all
```

### 静态模式优化后

```makefile
objects = foo.o bar.o all.o
all: $(objects)
	$(CC) $^ -o all

# 静态模式规则
$(objects): %.o: %.c
	$(CC) -c $^ -o $@

all.c:
	echo "int main() { return 0; }" > all.c

%.c:
	touch $@

clean:
	rm -f *.c *.o all
```
### 流程

1. 从 `all` 开始检查,发现需要 `foo.o bar.o all.o`.
2. 往下找到静态模式规则,与 `.o` 有关,执行它.
3. `.o` 依赖 `.c`,往下寻找 `.c`:
   - 如果 `foo.c` 或 `bar.c` 不存在,使用 `%.c` 规则生成空的 `.c` 文件.
   - 如果 `all.c` 不存在,使用 `all.c` 规则生成包含 `int main() { return 0; }` 的 `all.c` 文件.
4. 回到静态模式规则,编译 `.c` 为 `.o`.
5. 回到 `all`,链接 `.o` 为可执行文件 `all`.