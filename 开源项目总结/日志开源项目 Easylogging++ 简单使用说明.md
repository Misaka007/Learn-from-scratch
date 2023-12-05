# 日志开源项目 Easylogging++ 简单使用说明
## 一、 安装
首先，你需要从GitHub上下载Easylogging++的源代码。
项目链接：https://github.com/abumq/easyloggingpp
## 二、配置
在你的项目中，只需要包含一个头文件`easylogging++.h`和`easylogging++.cc`即可。你可以在你的主函数中初始化 Easylogging++。
初始化：
```cpp
#include "easylogging++.h"

INITIALIZE_EASYLOGGINGPP
```
## 三、案例
```cpp
#include <iostream>
#include "easylogging++.h"

INITIALIZE_EASYLOGGINGPP

int main() {

    std::string home = std::getenv("HOME");
    // printf(home.c_str());
    std::string path =
            home + "/zone3/ES_log/esm_%datetime{%Y-%M-%d_%H:%m:%s}.log";

    el::Configurations conf;
    conf.setGlobally(el::ConfigurationType::Filename, path.c_str());
    conf.setGlobally(el::ConfigurationType::Format,
                     "%datetime{%Y-%M-%d %H:%m:%s} %level %msg");
    // Turn off printing
    // conf.setGlobally(el::ConfigurationType::ToStandardOutput, "false");
    el::Loggers::reconfigureAllLoggers(conf);

    LOG(INFO) << "header braking";
    return 0;
}
```
解析：
在用户的 HOME 目录下的 /zone3/ES_log/ 路径中创建一个名为esm_%datetime{%Y-%M-%d_%H:%m:%s}.log的日志文件；

日志的格式被设置为 %datetime{%Y-%M-%d %H:%m:%s} %level %msg，每条日志都将包含日期时间、日志级别和日志消息。

conf.setGlobally(el::ConfigurationType::ToStandardOutput, "false");，取消终端打印；

LOG(INFO) << "header braking";将记录一条信息级别的日志，日志的内容是 2023-12-04 23:30:44 INFO header braking。
## 四、使用介绍
### 1. 日志级别
Easylogging++支持多种日志级别，包括INFO, WARNING, ERROR, FATAL等。你可以通过以下方式记录不同级别的日志：
```cpp
LOG(INFO) << "This is an info log";
LOG(WARNING) << "This is a warning log";
LOG(ERROR) << "This is an error log";
LOG(FATAL) << "This is a fatal log";
```
### 2. 日志格式
你可以自定义日志的格式。例如，你可以设置日志的时间格式，日志级别的显示方式等。
```cpp
el::Configurations conf;
conf.setGlobally(el::ConfigurationType::Format, "%datetime %level %msg");
el::Loggers::reconfigureLogger("default", conf);
```
在这个示例中，%datetime表示日志的时间，%level表示日志的级别，%msg表示日志的内容。
### 3. 日志文件
你可以将日志输出到文件。你可以设置日志文件的路径，以及是否自动刷新日志文件。
```cpp
el::Configurations conf;
conf.setGlobally(el::ConfigurationType::Filename, "/path/to/log/file.log");
conf.setGlobally(el::ConfigurationType::ToFile, "true");
conf.setGlobally(el::ConfigurationType::ToStandardOutput, "false");
el::Loggers::reconfigureLogger("default", conf);
```
在这个示例中，日志会被输出到/path/to/log/file.log文件中，而不会在终端打印。
