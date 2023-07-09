+++
title = "mac 的文件操作演示 (0)"
author = ["Isaac Quebec"]
draft = false
+++

Finder 我是不太用的. 我基本上都用命令行, 也就是 terminal (终端).

要用熟终端需要对 cd, mkdir, touch 等命令熟悉, 1 个小时足以熟练掌握. 建议安装 [z.sh](https://github.com/rupa/z) 装了它以后, 在不同目录中穿梭非常简单, 它会保存你最近访问过哪些目录, 你只需要输入几个字母即可. 比如:
![](/ox-hugo/Pasted_image_20230709223534.png)

{{< figure src="/ox-hugo/Pasted_image_20230709223544.png" >}}

找文件的话用 fzf. 首先需要安装 brew, 网上教程很多. 然后 brew install fzf 之类的就可以. 这个命令速度非常快. 如 gif:
![](/ox-hugo/2023-07-09-22-37-28.gif)

然后配合 mac 的 open 命令, 打开文件就非常轻松了.
