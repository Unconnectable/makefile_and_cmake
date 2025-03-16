CC = gcc       # 使用 gcc 作为 C 编译器
CFLAGS = -g    # 启用调试信息

# 隐式规则 #1: 通过 C 链接器隐式规则生成可执行文件 blah
# 隐式规则 #2: 通过 C 编译隐式规则生成 blah.o，因为 blah.c 存在
blah: blah.o

# 生成 blah.c 文件
blah.c:
	echo "int main() { return 0; }" > blah.c

# 清理生成的文件
clean:
	rm -f blah*