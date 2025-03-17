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

2. 所有命令之前加上 `.SILENT:`都不会输出原始的命令内容

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

## \* 通配符 wildcard

`$?`表示所有比目标更新的依赖文件(即那些被修改过的 .c 文件)\*\*

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

​ 启用调试信息: `-g`

​ 启用所有警告: `-Wall -Wextra`

​ 优化级别: `-O2`

​ 指定标准 (如 C++17): `-std=c++17`

**CPPFLAGS**: 传递给 C 预处理器的额外标志.

```
CPPFLAGS = -DDEBUG -Iinclude   # 示例
```

常用选项:

​ 定义宏: `-DDEBUG`

​ 添加包含目录: `-Iinclude`

**LDFLAGS**: 传递给链接器的额外标志.

```makefile
LDFLAGS = -Llib -pthread   # 示例
```

常用选项:

​ 添加库目录: `-Llib`

​ 指定链接选项: `-pthread`

**LOADLIBES** 和 **LDLIBS**: 链接的库.

```makefile
LDLIBS = -lm -lfoo   # 示例
```

常用选项:

​ 链接数学库: `-lm`

​ 链接动态库: `-lfoo`

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

---

## 删除当前文件夹除了指定文件之外的任何文件

```sh
find . -maxdepth 1 -type f ! -name 'Makefile' -exec rm -f {} +
```

## 静态模式规则(Static Pattern Rules)

### 语法

```makefile
targets...: target-pattern: prereq-patterns ...
    commands
```

通常使用通配符去实现目标模式和依赖模式.

​ **目标**: `foo.o bar.o all.o`

​ **目标模式**: `%.o`,匹配所有 `.o` 文件.

​ **依赖模式**: `%.c`,将 `%` 替换为匹配的主干 (例如 `foo`),生成依赖文件 `foo.c`.

​ **命令**: 编译 `.c` 文件生成 `.o` 文件.

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

## 静态模式规则和过滤(Static Pattern Rules and Filter)

```Makefile
files = foo.c bar.o baz.h qux.c
c_and_h_files = $(filter %.c %.h, $(files))

```

从 files 里筛选`.c`和`.h`文件

```Makefile
$(filter %.o,$(obj_files)): %.o: %.c
	echo "target: $@ prereq: $<"

$(filter %.result,$(obj_files)): %.result: %.raw
	echo "target: $@ prereq: $<"
```

从 obj_files 中筛选出所有以 .o 结尾的文件
`: %.o: %.c`是一个规则模式,表示每个`.o`依赖`.c`文件
筛选`.result`文件 表示依赖于`.raw`文件

## 模式规则(Pattern Rules)

```makefile
%.o : %.c
	$(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@
```

将所有的 `.c` 文件编译成 `.o` 文件

双冒号:

```makefile
all: blah

blah::
	echo "hello"

blah::
	echo "hello again"

#输出
echo "hello"
hello
echo "hello again"
hello again
```

对于同一个目标定义多个规则,会运行多次

```makefile
all: blah

blah:
	echo "hello"

blah:
	echo "hello again"
#output
Makefile:7: 警告:覆盖关于目标“blah”的配方
Makefile:4: 警告:忽略关于目标“blah”的旧配方
echo "hello again"
hello again
```

单个冒号多个规则会被警告和覆盖,新的规则会覆盖旧的规则

---

## 命令与执行(Commands and execution)

### 命令回显/静默(Command Echoing/Silencing)

```makefile
all:
	@echo "This make line will not be printed"
	echo "But this will"
```

`@`后面的不会被显示,只会执行

### 双美元符号

```makefile
make_var = I am a make variable
all:
	sh_var='I am a shell variable'; echo $$sh_var

	@echo $(make_var)
```

定义变量`make_var`,第二行`echo`这个变量
`sh_var`定义在命令行中,要使用他需要在同一行`echo`,而且需要`$$`进行转义

### 错误处理

`-k` 选项:继续执行即使遇到错误:
`make -k`假设 Makefile 中有多个目标,其中一个目标的命令失败了,make 会继续执行其他目标,而不是立即停止.

-i 选项:忽略所有命令的错误:`make -i`,忽略所有命令的错误,继续执行后续的命令

在命令前添加 -:忽略单个命令的错误:

```makefile
all: target1 target2 target3

target1:
    echo "Running target1"
    false  # 这个命令会失败

target2:
    echo "Running target2"
    -false  # 这个命令会失败,但错误会被忽略

target3:
    echo "Running target3"
```

```makefile
new_contents = "hello:\n\ttouch inside_file"
all:
	mkdir -p subdir
	printf $(new_contents) | sed -e 's/^ //' > subdir/makefile
	cd subdir && $(MAKE)

clean:
	rm -rf subdir

```

流程:

1. `new_contents`是一个字符串不会作为命令被使用
2. `all`作为默认目标被执行
3. 创建目录`subdir`,向`subdir/makefile`文件写入`new_contents`的内容
4. 进入`subdir`然后递归调用`makefile`
5. 当前目录的`makefile`内容是

```makefile
hello:
	touch inside_file
```

输出结果

```sh
make

mkdir -p subdir
printf "hello:\n\ttouch inside_file" | sed -e 's/^ //' > subdir/makefile
cd subdir && make
make[1]: 进入目录“parent_dir/subdir”
touch inside_file
make[1]: 离开目录“parent_dir/subdir”
```

## 导出、环境变量和递归 make(Export, environments, and recursive make)

### 基本术语:

环境变量(Environment Variable):操作系统中的全局变量,可以在当前 shell 会话及其子进程中使用.
1.1 在 Shell 中设置环境变量

```bash
# 设置环境变量
export MY_ENV_VAR="I am an environment variable"
# 查看环境变量
echo $MY_ENV_VAR  # 输出:I am an environment variable
```

1.2 设置`MY_ENV_VAR`后在 Makefile 中使用环境变量

```makefile
all:
    echo $$MY_ENV_VAR  # 输出:I am an environment variable
    echo $(MY_ENV_VAR) # 输出:I am an environment variable
```

全局变量(Global Variable)
定义:全局变量是指在 Makefile 中定义的变量,可以在整个 Makefile 中使用.

2.1 在 Makefile 中定义全局变量

```makefile
#这是一个Makefile的全局变量
MY_GLOBAL_VAR="I am a global variable"

all:
    echo $(MY_GLOBAL_VAR)  # 输出:I am a global variable
    echo $$MY_GLOBAL_VAR   # 输出为空,因为 MY_GLOBAL_VAR 不是环境变量

```

2.2 定义后,导出全局变量为环境变量

```makefile
MY_GLOBAL_VAR="I am a global variable"
export MY_GLOBAL_VAR

all:
    echo $(MY_GLOBAL_VAR)  # 输出:I am a global variable
    echo $$MY_GLOBAL_VAR   # 输出:I am a global variable
```

一个具体的例子

```makefile
new_contents = "hello:\n\techo \$$(cooly)"

all:
	mkdir -p subdir
	printf $(new_contents) | sed -e 's/^ //' > subdir/makefile
	@echo "---MAKEFILE CONTENTS---"
	@cd subdir && cat makefile
	@echo "---END MAKEFILE CONTENTS---"
	cd subdir && $(MAKE)

# Note that variables and exports. They are set/affected globally.
cooly = "The subdirectory can see me!"
export cooly
# This would nullify the line above: unexport cooly

clean:
	rm -rf subdir

#输出
make
mkdir -p subdir
printf "hello:\n\techo \$(cooly)" | sed -e 's/^ //' > subdir/makefile
---MAKEFILE CONTENTS---
hello:
        echo $(cooly)---END MAKEFILE CONTENTS---
cd subdir && make
make[1]: 进入目录“Parent_dir/subdir”
echo "The subdirectory can see me!"
The subdirectory can see me!
make[1]: 离开目录“Parent_dir/subdir”
```

流程:

1. `,mkdir -p subdir`
2. 向`subdir/makefile`输入内容
3. 进入`subdir` 然后`cat makefile`
4. 进入子文件夹然后递归调用`make`
5. 因为通过`export`把`cooly `导出为`env_var`所以在子文件夹可以 `echo`

## **很重要!!**

> 第三步,启动一个新的 make 进程来执行子目录中的 Makefile,执行完后会自动输入消息进入目录和离开目录.但是命令没有离开目录的操作,为什么会回到原来的目录呢？子目录中完成执行后,它会自动返回到父目录

### 语法解析

`create_ = "target_name:\n\techo \$$(var_name)"`:

1.  `\t`制表符,也就是`tab`,要和换行符连起来用满足`makefile`的格式,智能缩进一个`tab`

```makefile
target:dependece
	command

#否则就多了一个空格
target:dependece
	 command
```

2. 使用`\$`对`$`进行转义字符 然后解析变量

`printf "hello:\n\techo \$(cooly)" | sed -e 's/^ //' > subdir/makefile`

1. `printf`是输出命令, 输出一个字符串`"hello:\n\techo \$(cooly)"`
2. `|`管道
3. `sed -e 's/^ //'`:`sed`是流编辑器,表示对文本处理.`-e`表示后面跟着一个编辑命令.
4. `s/匹配模式/替换内容/标志`:对应到这里`'s/^ //'`:匹配模式`^`是正则表达式元字符,表示行的开头
   ` sed -e 's/^ // '` 匹配行开头后紧跟的一个空格字符

### `make`的参数

`make clean run test`:按照顺序依次`clean` `run` `test`

---

# 变量(Variables Pt. 2)

不同的定义方式

1. `=`是递归定义,会在后面展开追加的定义
2. `:=`是直接定义
3. `?=`是如果没有定义那就定义,否则不修改当前的定义

```makefile
one = one ${later_var}
two = two ${later_var} ${later_var}
three := three ${later_var} ${later_var}${later_var}
four = x_4
four ?= x_5
five ?= x_5
later_var= later

all:
	@echo $(one)
	@echo $(two)
	@echo $(three)
	@echo $(four)
	@echo $(five)


#output
one later
two later later
three
x_4
x_5
```

递归定义

```makefile

one = hello
one = ${one}

all:
	echo $(one)
#output
Makefile:3: *** Recursive variable 'one' references itself (eventually).  Stop.
```

这里一直递归定义`one`,会停止运行

一个合理的递归

```makefile
one = hello
one := ${one} there

all:
	echo $(one)

#output
echo hello there
hello there
```

只会递归一次把`one`从 hello 变成 hello there

行首的字符会被删除,行尾会,比如保留`" "`空格

```makefile
with_spaces = hello
after = $(with_spaces)there

nullstring =
space = $(nullstring)

all:
	echo "$(after)"
	echo start"$(space)"end
#output
echo "hello     there"
hello     there
echo start" "end
start end
```

同时使用

```makefile
foo := start
foo += more

all:
	echo $(foo)
```

命令行参数和覆盖(Command line arguments and override)

```makefile
override option_one = did_override
option_two = not_override
all:
	echo $(option_one)
	echo $(option_two)

#make 指定参数option_one = hi
make option_one=hi
#output
did_override
not_override
```

## 命令列表和定义(List of commands and define)

### 目标特定的变量(Target-specific variables)

在`all`这这个特定的目标里面设置把`one`设置为`cool`,其他情况`one`还是`one`

```makefile
all: one = cool

all:
	echo one is defined: $(one)

other:
	echo one is nothing: $(one)
```

命令

1.  `make` 等于默认`make all` \
    输出\
    `echo one is defined: cool`\
    `one is defined: cool`
2.  `make other`\
     输出\
     `echo one is defined: nothing`\
     `one is defined: nothing`

### 特定模式的变量(Pattern-specific variables)

```makefile

%.c: one = cool

blah.c:
echo one is defined: $(one)

other:
echo one is nothing: $(one)

```

---

## Makefiles 的条件部分(Conditional part of Makefiles)

### 条件 if/else

```makefile
foo = ok

all:
ifeq ($(foo), ok)
	echo "foo equals ok"
else
	echo "nope"
endif
```

### 检查一个变量是否为空

```makefile
nullstring =
foo = $(nullstring) # end of line; there is a space here

all:
ifeq ($(strip $(foo)),)
	echo "foo is empty after being stripped"
endif
ifeq ($(nullstring),)
	echo "nullstring doesn't even have spaces"
endif
```

检查一个变量是否已定义

```makefile
bar =
foo = $(bar)

all:
ifdef foo
	echo "foo is defined"
endif
ifndef bar
	echo "but bar is not"
endif
```

### Makelags

检查`make`后面的参数是否出现某一个

```makefile
all:
# Search for the "-i" flag. MAKEFLAGS is just a list of single characters, one per flag. So look for "i" in this case.
ifneq (,$(findstring i, $(MAKEFLAGS)))
	echo "i was passed to MAKEFLAGS"
endif
```
