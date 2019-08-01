---
layout: post
title:  "Logging in Python"
categories: python
tags: python logging
---
# 何时logging

目标|工具
-|-
命令行显示普通的控制台输出 | `print()`
报告正常操作中发生的事件 | `logging.info()`(对于详细的输出可用`logging.debug()`)
运行时发生问题时给出警告 | `warnings.warn()`（在库代码中使用，说明问题可被避免，应该修改客户程序以消除警告）或`logging.warning()`（客户程序无能为力，但该事件仍需要提醒）
报告一个运行时错误 | 抛出异常
不抛出异常，抑制错误的报告（如一个需要长期运行的服务器进程的错误处理）|`logging.error()`,`logging.exception()`或`logging.critical()`

# logging的级别
级别|场合
-|-
`DEBUG`|只在debug时需要的详细信息
`INFO`|确认程序正常进行
`WARNING`|预期外的事件发生，或作为未来某些事件的预兆（如硬盘空间不足），但程序仍然可以正常运行
`ERROR`|严重的问题，会导致程序无法执行某些功能
`CRITICAL`|可能导致程序崩溃的问题

python的默认logging级别是`WARNING`，即`INFO`和`DEBUG`不会显示。
# 基础配置
## 通过代码配置
```
logging.basicConfig(filename='example.log',filemode='w', level=logging.DEBUG)
```
可设置log的输出文件和级别。模式默认为'a'
### 其它参数
参数|解释
-|-
`format='%(levelname)s:%(message)s'`|输出格式,`%(asctime)s`可显示时间
`datefmt='%m/%d/%Y %I:%M:%S %p'`|`%(asctime)s`显示的日期格式
## 通过命令行配置
```
python foobar.py --log=INFO
```
设置log级别
输入验证：
```
numeric_level=getattr(logging, loglevel.upper(), None)
if not isinstance(numeric_level, int):
    raise ValueError('Invalid log level: %s' % loglevel)
logging.basicConfig(level=numeric_level, ...)
```
# 高级配置
## 组件
组件|说明
-|-
`Logger`|提供程序直接使用的接口
`Handler`|将`Logger`产生的log发送至目的地
`Filter`|提供细粒度的方式控制log的输出
`Formatter`|控制最终输出的log的布局
`LogRecord`|在上述4个组件中传递的Log实例
## Logger
- 每个Logger实例拥有一个名字，以层次化的方式命名（用`.`隔开）
```
logger = logging.getLogger(__name__)
```
可获取用模块命名的logger  
- 直接调用logging.debug()等方法使用的Logger名字为'root'
- 一个Logger产生的LogRecord会自动传递给它祖先的Handler
- Logger的级别未指定时，继承父Logger的级别
- `logger.exception()`会同时输出一个stack trace
- `logger.log()`需要显示指定级别，可以指定自定义级别
- `addFilter()`和`removeFilter()`
- `addHandler()`和`removeHandler()`
- `setLevel()`
## Handler
- 可以指定Log的目的地，包括HTTP GET/POST，电子邮件，socket，Linux的syslog，Windows NT的event log等。
- `setLevel()`指定发送的Log级别
- `setFormatter()`指定格式
- `addFilter()`和`removeFilter()`
## Formatter
- 初始化：`logging.Formater(fmt='',datefmt='')`
