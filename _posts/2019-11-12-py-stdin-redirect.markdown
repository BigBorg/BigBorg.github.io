---
layout: 	post
title:		"python调用二进制程序-标准输入输出流重定向"
header-img:	"img/post-bg-2016.jpg"
date:		2019-11-12
author: 	"Borg"
catalog:	true
tags:
    - Python
---

# 背景介绍
[fastalign](https://github.com/clab/fast_align) 是自然语言处理领域的一个词对齐工具，需要对 fastalign 的训练、模型存储、文本/单句对齐（模型使用）等步骤封装成自动化的 web 平台。fastalign 算法本身没有 python 实现，尝试过将源代码用 python 重写，但发现与官方的二进制程序相比性能差别极大，特别是当需要对齐的数据达上千万句子时，这个性能差距会被放大地更加明显。

# 调研
仔细观察 fast_align 的源代码会发现里面有个 [force_align.py](https://github.com/clab/fast_align/blob/master/src/force_align.py) 的脚本，里面虽然没有 fast_align 算法的 python 实现，但该脚本其实是调用了二进制程序，python 读入句子，再通过标准输入流的重定向把输入传送给二进制程序，再从二进制程序的标准输出读出结果。其核心代码摘录如下：

```python
def popen_io(cmd):
    p = subprocess.Popen(cmd, stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    def consume(s):
        for _ in s:
            pass
    threading.Thread(target=consume, args=(p.stderr,)).start()
    return p

fwd_cmd = [fast_align, '-i', '-', '-d', '-T', fwd_T, '-m', fwd_m, '-f', fwd_params]
fwd_align = popen_io(fwd_cmd)
fwd_align.stdin.write('{}\n'.format(line))
fwd_line = fwd_align.stdout.readline().split('|||')[2].strip()
```
即用 subprocess.Popen 去开启二进制程序作为子进程存储为 fwd_align，通过 fwd_align 的 stdin, stdout 就可以与二进制程序交互。

# 适配 python 3
官方的 force_align.py 确实可用，但是在 python3 环境下则会出现编码错误，以及输入之后 fwd_align 一直收不到输入数据的错误。通过调查 subprocess.Popen 的文档发现有两个参数需要指定才能正常工作。

1. buffsize：
缓冲区大小，python3 下默认等于 io.DEFAULT_BUFFER_SIZE，我的机器上是8192。根据文档，python3.2 之前的默认值是0，即不缓冲，因此上面的 force_align.py 才能正常运行。而现在则需要**将 buffsize 设置为 1**，即行缓冲，写完一行后，子进程就可以取到数据，否则默认情况下一行数据未达到缓冲大小，子进程就没法取到输入。

2. universal_newlines:
universal_newlines 为 True 时 buffsize=1 才会生效，并且会以文本模式打开标准输入输出，即对标准输入输出写入写出的数据格式是 python3 中的 str 字符串，否则是以二进制流打开，写入写出的是 bytes 。

# 注意事项
用该种方式封装 fast_align 模型调用需要注意的是二进制文件读取一行后给出一行输出，该过程中间不能穿插其他输入，因此封装出来的模型是非线性安全的，同时只能处理一条数据，多线程下需要用锁对模型进行保护。还有输入注意以行为单位，需要确保输入的每句话以换行符结尾。


