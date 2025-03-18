# 感谢 Job Vranish (https://spin.atomicobject.com/2016/08/26/makefile-c-projects/)
TARGET_EXEC := final_program

BUILD_DIR := ./build
SRC_DIRS := ./src

# 查找所有需要编译的 C 和 C++ 文件
# 注意 * 表达式周围的单引号,否则,Shell 会错误地扩展这些表达式,但我们希望直接将 * 传递给 find 命令,
SRCS := $(shell find $(SRC_DIRS) -name '*.cpp' -or -name '*.c' -or -name '*.s')

# 在每个 src 文件前添加 BUILD_DIR,并在其后添加 .o
# 例如,./your_dir/hello.cpp 会变成 ./build/./your_dir/hello.cpp.o
OBJS := $(SRCS:%=$(BUILD_DIR)/%.o)

# 字符串替换(不带 % 的后缀版本),
# 例如,./build/hello.cpp.o 会变成 ./build/hello.cpp.d
DEPS := $(OBJS:.o=.d)

# 需要将 ./src 中的每个文件夹传递给 GCC,以便它能够找到头文件
INC_DIRS := $(shell find $(SRC_DIRS) -type d)
# 在 INC_DIRS 前添加前缀,例如,moduleA 会变成 -ImoduleA,GCC 理解这个 -I 标志
INC_FLAGS := $(addprefix -I,$(INC_DIRS))

# -MMD 和 -MP 标志一起为我们生成 Makefile！
# 这些文件将使用 .d 作为输出,而不是 .o,
CPPFLAGS := $(INC_FLAGS) -MMD -MP

# 最终的构建步骤,
$(BUILD_DIR)/$(TARGET_EXEC): $(OBJS)
	$(CXX) $(OBJS) -o $@ $(LDFLAGS)

# C 源代码的构建步骤
$(BUILD_DIR)/%.c.o: %.c
	mkdir -p $(dir $@)
	$(CC) $(CPPFLAGS) $(CFLAGS) -c $< -o $@

# C++ 源代码的构建步骤
$(BUILD_DIR)/%.cpp.o: %.cpp
	mkdir -p $(dir $@)
	$(CXX) $(CPPFLAGS) $(CXXFLAGS) -c $< -o $@


.PHONY: clean
clean:
	rm -r $(BUILD_DIR)

# 包含 .d 文件,前面的 - 用于抑制缺少 Makefile 的错误,
# 最初,所有的 .d 文件都会缺失,我们不希望显示这些错误,
-include $(DEPS)