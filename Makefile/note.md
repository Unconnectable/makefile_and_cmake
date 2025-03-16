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

