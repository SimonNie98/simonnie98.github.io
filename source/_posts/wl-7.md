---
title: 0505-0511 工作总结
tags: []
categories: []
toc: true
comments: true
date: 2025-05-10 19:11:14
---

# 一、工程开发经验
## 1. 编译
在开发机上碰到编译代码的问题时，一定要优先考虑本地编译环境、编译流程和流水线的编译环境、流程的区别。通过diff逐步排查问题。

## 2. 运行
> 有一个出现过两次的问题（实在不应该）：在本地跑一个单测，无论是master还是自己的dev分支都死活过不了，返回状态码 548。这个时候最应该做的事情是：通过tcpdump抓包分析问题，定位问题。最后实际的问题是在于：开发机上跑了一个运行时的程序，所以单测一直接收到运行时的程序给它的响应。

这个故事要记录下来，要养成在运行时抓包分析网络请求的好习惯。

> 一些可能用到的命令:
> tcpdump -i lo any (or port XXX) -w output.pcad &
> &, fg, bg, jobs, ctrl+z
> tcpdump -r output.pcad -X -nn -v | less
> gdb ./bin_file -c /data/corefiles/xxx
> tail -f log.txt
> crontab -e 查看定时任务。




# 二、C++

1. `const` vs `constexpr` vs `consteval` vs `constinit`

2. `std::optional` 取代 `std::pair<T, bool>` 使用

3. `if constexpr`

4. `string_view` vs `string`

5. `std::variant`

6. `->*` vs `*md->pointer`

7. 不暴露给用户的函数不应该定义在`.h`文件中；

8. struct的反序列化函数，如果需要给用户使用，可以放在全局或者以静态成员函数的形式（接口返回一个对象）；序列化则可以用成员函数的方式定义；

9. char[]数组到std::string可能有一个implict type conversion.


# 三、VIM 相关的命令

1. esc模式下，`G` 跳到最后一行，小`gg` 行首。`%` 跳到花括号 `{}` 开始的地方;
