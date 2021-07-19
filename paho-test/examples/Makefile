#--------------------------------------------------------------
#           my app
#--------------------------------------------------------------

ZH_SDK_PATH   ?= $(shell pwd)

# 可执行文件的名字
TARGET = mqtt_test_exe
# 先定义编译器链接器
#CC =/opt/linux-imx6ull/linux-glibc/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc
CC= gcc
#LD =/opt/linux-imx6ull/linux-glibc/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-ar
# -I指定头文件目录
INCLUDE = -I$(ZH_SDK_PATH)/../src 
# 开启编译warning和设置优化等级
CFLAGS = -g -w $(INCLUDE)
# -l指定静态库名字
LIB =  -lm -lpthread
# 正则表达式表示目录下所有.c文件，相当于：SRCS = main.c a.c b.c
SRCS = $(wildcard ./my-mqtt-test.c )
MQTT_SRC_FILES=$(wildcard ../src/*.c)
# OBJS表示SRCS中把列表中的.c全部替换为.o，相当于：OBJS = main.o a.o b.o
OBJS = $(patsubst %c, %o, $(SRCS))
MQTT_OBJS =  $(patsubst %c, %o, $(MQTT_SRC_FILES))

# 要生成的目标文件
all: $(TARGET)
	#cp -a $(TARGET)  /mnt/hgfs/ubuntu-share/test-jkl/
	#mv  $(OBJS) $(MQTT_OBJS)  ./obj/

# 第一行依赖关系：冒号后面为依赖的文件，相当于Hello: main.o a.o b.o
# 第二行规则：$@表示目标文件，$^表示所有依赖文件，$<表示第一个依赖文件
$(TARGET): $(OBJS)  $(MQTT_OBJS)
	$(CC) -o $@ $^ $(LIB)
# 上一句目标文件依赖一大堆.o文件，这句表示所有.o都由相应名字的.c文件自动生成
%o:%c
	$(CC) -c $^ $(INCLUDE) $(CFLAGS) 

.PHONY:clean  
clean:
	rm -rf $(OBJS) $(TARGET)  $(MQTT_OBJS)
