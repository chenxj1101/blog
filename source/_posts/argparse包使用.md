layout: post
title: argparse包用法总结
date: 2018/2/20 20:19:50
categories:
- 编程
tags:
- Python
- argparse
- 命令行

---

经常需要使用argparse这个包来写命令行，所以对用法做个记录，以免忘记

## Argparse简介

argparse是python自带的一个命令行解析包，非常适合编写可读性好的命令行程序。
要使用的话，直接导入即可，无需另外安装。

## 实例

新建一个 `parse.py`文件如下：
```python
# !/usr/bin/python
# -*- coding: UTF-8 -*-

import argparse

# get parameters
def parse_args():

    description = u"扫描给定项目路径，生成路径下数据信息"

    parse = argparse.ArgumentParser(description=description)

    help = u"输出文件名称"
    parse.add_argument('output', help=help)

    help = u"已记录好的总表，如不存在可用'-'代替"
    parse.add_argument('backup', help=help)

    help = u"需要扫描的项目目录,可输入多个"
    parse.add_argument('filepath', nargs='*', help=help)

    help = u"项目名称"
    parse.add_argument('-p','--project', help=help)

    help = u"数据类型, 默认值为: FASTQ"
    parse.add_argument('-t','--type',default='FASTQ' , help=help)

    help = u"测序平台, 默认值为: Illumina"
    parse.add_argument('-c', '--cexu', default='Illumina', help=help)

    help = u"文库大小, 数据类型:int(整数型)"
    parse.add_argument('-l', '--library',  type=int, help=help)

    args = parse.parse_args()

    return args


if __name__ == '__main__':

    args = parse_args()
    output = args.output
    backup = args.backup
    filepath = args.filepath
    project = args.project
    ...
```

运行 `python parse.py -h` 

```bash
usage: scanData.py [-h] [-p PROJECT] [-t TYPE] [-c CEXU] [-l LIBRARY]
                   output backup [filepath [filepath ...]]

扫描给定项目路径，生成路径下数据信息

positional arguments:
  output                输出文件名称
  backup                已记录好的总表，如不存在可用'-'代替
  filepath              需要扫描的项目目录,可输入多个

optional arguments:
  -h, --help            show this help message and exit
  -p PROJECT, --project PROJECT
                        项目名称
  -t TYPE, --type TYPE  数据类型, 默认值为: FASTQ
  -c CEXU, --cexu CEXU  测序平台, 默认值为: Illumina
  -l LIBRARY, --library LIBRARY
                        文库大小, 数据类型:int(整数型)
```

可以看到帮助信息是由argparse包自动生成的。且帮助信息是可以自己定义的。

## 定位参数

在上述程序中，`output`, `backup`, `filpath`这3个参数在帮助信息中是`positional arguments`，中文意思就是`定位参数`，用法是不用带`-`就能使用。顾名思义，这些参数的位置是固定的，前后顺序是无法更换的，且这些参数是必选的，不全的话就会报错。

```bash
python scanData.py test.txt
usage: scanData.py [-h] [-p PROJECT] [-t TYPE] [-c CEXU] [-l LIBRARY]
                   output backup [filepath [filepath ...]]
scanData.py: error: too few arguments
```

## 可选参数

上述程序中的其他参数，在帮助信息中显示是`optional arguments`， 意思是可选参数。可以使用`-`来指定短参数，也可以使用`--`来表示长参数。可选参数是`非必选`的，可用可不用。而且顺序是可以随意更换的。

## 参数设置

在`argparse`中可以对参数的`数据类型、默认值、可选值、帮助信息`等进行设置。

```python
# !/usr/bin/python
# -*- coding: UTF-8 -*-

import argparse

parse = argparse.ArgumentParser()

help = u"文库大小, 数据类型:int(整数型)"
parse.add_argument('-l', '--library',  type=int, default=150, choice=[150,200], help=help)
args = parse.parse_args()

```

在`add_argument`方法中，`type`指定数据类型，`default`指定默认值，`choice`指定可选值， `help`指定自定义帮助信息。`nargs`可以指定参数的数量，值可以为正整数，也可以为`*`表示`任意个数，可以为0个`， 还可以为`+`，代表`1或1以上的个数`。

在上述的`parse.py`中，`filepath`这个参数的个数就是不固定的，可以传入多个参数。参数解析之后，得到的是一个数组。


## 文档

[Argparse文档](https://docs.python.org/3/library/argparse.html)