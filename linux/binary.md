# 二进制分析

### readelf

elf全称为executable and linking format。

**查看so信息：**`readelf -d liboma.so`

### objdump

### ldd

**查看可执行文件/动态库的依赖：**`ldd xx.so`

### nm

**查看某个符号是否定义：**`nm liboma|grep 'HandleDatasetStart`

