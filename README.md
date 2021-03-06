# limlog ![](https://www.travis-ci.org/zxhio/limlog.svg?branch=master)

快速、轻量的 **C++11** 日志库。[查看细节分析](https://www.cnblogs.com/shuqin/p/12103952.html)

### 安装
limlog 代码文件较少，可以直接自行编译，也可以将 `Log.h` 和生成的 `liblimlog.a` 放到项目所在目录。

```sh
$ git clone https://github.com/zxhio/limlog.git
$ cd limlog
$ mkdir build && cd build
$ cmake [-DLIMLOG_NO_FILE_FUNC_LINE=ON] ..
$ make
```

### 平台 (x86)
- Linux (WSL)
- Windows
- OS X

### 日志格式

每条日志格式及日志文件名如下：
```c
test_log_file.20200102.1.log

20200102 13:49:31.669000 3 DEBUG  true - LogTest.cpp:log_1_same_element_x6():242
20200102 13:49:31.669000 3 DEBUG  c - LogTest.cpp:log_1_same_element_x6():245
20200102 13:49:31.670000 3 DEBUG  c@string - LogTest.cpp:log_1_same_element_x6():248
20200102 13:49:31.670000 3 DEBUG  std::string - LogTest.cpp:log_1_same_element_x6():252
```

若是日志行不需要后缀如 `LogTest.cpp:log_1_same_element_x6():242`, 可以添加编译宏 `NO_FILE_FUNC_LINE` 来控制。

### 使用
用法同 `std::cout`

```cpp
#include "Log.h"
#include <string>

int main() {
    setLogFile("./test_log_file");
    setLogLevel(limlog::LogLevel::DEBUG);
    setRollSize(64); // 64MB

    std::string str("std::string");
    LOG_DEBUG << 'c' << 65535 << -9223372036854775808 << 3.14159 << "c@string" << str;

    return 0;
}
```

### 性能

测试的机器为 i7 9700k@8 Linux 4.4(WSL), MSVC 跑了几个用例耗时大概是WSL一半左右，这里后面具体测一下。

测试用例如下

**-** 表示一个该日志包含的元素，**-4** 表示4个，**-+4096** 表示除了写入一个该元素外，再追加 4096 字节长度的该元素。[具体用例](./LogTest.cpp)

| 序号/类型 | 长度(byte) | bool | char | int16_t | uint16_t | int32_t | uint32_t | int64_t | uint64_t | double | c@string | std::string |
| :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| 1 | 82(495/6) | - | - | | | - | | | | - | - | - |
| 2 | 99(596/6) | -4 | -4 | | | -4 | | | | -4 | -4 | -4 |
| 3 | 167(1006/6)  | -16 | -16 | | | -16 | | | | -16 | -16 | -16 |
| 4 | 180 | | - | - | - | - | - | - | - | - | - | - |
| 5 | 243 | | - | - | - | - | - | - | - | - | - + 64 | - |
| 6 | 243 | | - | - | - | - | - | - | - | - | - | - + 64 |
| 7 | 435 | | - | - | - | - | - | - | - | - | - + 256 | - |
| 8 | 435 | | - | - | - | - | - | - | - | - | - | - + 256 |
| 9 | 1202 | | - | - | - | - | - | - | - | - | - + 1024 | - |
| 10 | 1202 | | - | - | - | - | - | - | - | - | - | - + 1024 |
| 11 | 4275 | | - | - | - | - | - | - | - | - | - + 4096 | - |
| 12 | 4275 | | - | - | - | - | - | - | - | - | - | - + 4096 |

单位：微秒/条。

| 线程/序号 | 单个用例条数 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 |
| :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| 1  | 10w | 1.23 | 1.50 | 2.55 | 2.66 | 3.45 | 3.44 | 5.93 | 5.66 | 1.30 | 1.09 | 1.45 | 1.32 |
| 2 | 10w | 1.20 | 1.46 | 2.45 | 2.51 | 3.29 | 3.27 | 5.14 | 5.15 | 1.19 | 1.16 | 12.79 | 65.54 |
| 4 | 10w | 1.13 | 1.36 | 1.70 | 2.00 | 1.42 | 1.12 | 1.12 | 1.12 | 1.21 | 33.56 | 127.66 | 131.84 |
| 8 | 10w | 1.22 | 1.34 | 1.96 | 1.74 | 16.41 | 6.67 | 26.08 | 28.47 | 68.16 | 75.51 | 260.09 | 262.99 |
| 1 | 100w | 1.24 | 1.51 | 2.59 | 2.58 | 3.45 | 3.45 | 5.88 | 5.87 | 2.16 | 8.79 | 31.82 | 31.40 |
| 2 | 100w | 1.21 | 1.52 | 2.55 | 2.86 | 3.27 | 3.26 | 5.03 | 5.17 | 17.17 | 17.65 | 63.49 | 63.20 |
| 4 | 100w | 1.14 | 2.99 | 5.51 | 5.36 | 6.92 | 7.31 | 13.21 | 12.88 | 35.16 | 36.24 | 128.66 | 130.22 |
| 8 | 100w | 3.17 | 6.37 | 11.03 | 10.89 | 14.73 | 14.73 | 28.66 | 23.08 | 71.57 | 72.51 | 263.82 | 273.63 |

由于所有的测试用例都是一次跑完的，单个用例的条数设置超过 1000w 后产生的日志大小超过 110G，在某些的情况下，序号 **1** 能跑到 0.77 us 左右，这个现象我也不清楚怎么偏差这么大的，在使用SSD进行压测后，日志长度越大的情况下提升的幅度越大，在序号1的情况下也能达到 0.9 us，目前的后端设计还是有问题的，这里需要改进。

问题：因为CPU的核数为8，在测试线程开8个（实际运行线程为9）的时候，速度出现了大幅度的下降。在使用任务管理器观察性能的时候，8线程磁盘写入每15秒左右速度减半，这个过程持续3秒，猜测也是后端写入的问题。

### TODO
1. 优化后端 buffer，在多个线程同时写入的时候，后端日志写入明显拖累了整体的性能。
2. 优化浮点数格式化为字符串的处理及对字符串写入后端的处理。

### 参考
1. [Iyengar111/NanoLog](https://github.com/Iyengar111/NanoLog), Low Latency C++11 Logging Library.
2. [PlatformLab/NanoLog](https://github.com/PlatformLab/NanoLog), Nanolog is an extremely performant nanosecond scale logging system for C++ that exposes a simple printf-like API.
3. [kfifo](https://github.com/torvalds/linux/blob/master/lib/kfifo.c), 环形生产者消费者队列.
4. [memory barries](https://github.com/torvalds/linux/blob/master/Documentation/memory-barriers.txt), 内核的内存屏障.
5. [itoa-benchmark](https://github.com/miloyip/itoa-benchmark), 几种对整形数字转换成字符串的实现和性能比较，limlog 选择了简单又快速的查表方式。