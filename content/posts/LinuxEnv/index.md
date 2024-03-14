---
title: Linux环境变量与进程环境列表
dir: posts/LinuxEnv
share: true
tags:
  - linux
date: 2024-03-13T19:30:13+08:00
draft: false
---
## 一、概述

每个进程都有一份环境列表，即在用户空间内存维护的一组环境变量。
* 调用 `fork()` 创建的新进程，会继承父进程的环境副本，这也为父子进程间通信提供了一种机制。
* 调用 `exec()` 替换当前正在运行的程序时，新程序要么继承老程序的环境，要么在 `exec()` 调用的参数中指定新环境并加以接收。

环境变量最常用的用途之一是在 shell 程序中。通过自身环境中放置变量值，将环境变量传递给其创建的进程。
大多数 shell 使用 `export` 命令向环境中添加变量值。

```shell
SHELL=/bin/bash
# 上述命令仅仅是创建变量
export SHELL
# Pur variable into shell process's environment
# 在 Bourne shell 及其衍生 shell (bash Korn) 可使用下列语法仅向执行的程序添加一个变量而不改变父 shell
NMAE=jackson program
# 可以在 program 前放置多对
```

## 二、 在 shell 中访问

`printenv` 命令会打印当前进程的环境列表。
通过 Linux 专有的 `/proc/PID/environ` 文件可以检查任一进程的环境列表。

## 三、在程序中访问


**读取环境变量**

1. 在 C 语言程序中，可以使用全局变量 `cahr **environ` 访问。C 运行启动代码定义了该变量并以环境列表为其赋值。
2. 在 `main()` 函数声明中定义第三个参数来访问环境列表。该参数的作用域仅在 `main()`函数内，除了该局限性，该特性也不在 `SUSv3` 中。
```C
int main(int argc, char *argv[], char* envp[])
{
	return 0;
}
```
3. 使用库 API `cahr* getenv(const char* name);`。
	* 不要修改 `getenv()` 返回的字符串，这是因为大多数实现，该字符串实际上是环境的一部分，即与环境由相同的地址。


**添加环境变量**

```C
include <stdlib.h>
int putenv(cahr *string);
// Returns 0 on success, or nonzero on error
```

string 是 name=value 形式的字符串。
注意调用该字符串后，该字符串就成为了环境的一部分，简而言之，环境中保存的并不是该 string 的副本，而就是其本身。此后不应修改该 stirng 所在内存区域。


**修改环境变量**

```C
int setenv(const char* name, const char* value, int overwrite);
// Returns 0 on success, or -1 on error
```

`setenv()` 复制其参数到环境中，与 `putenv` 恰恰相反。

**删除环境变量**

```C
int unsetenv(const char* name);
int clearenv(void);

// 也可以通过将 environ 变量赋值为 NULL 来达到清除的目的。
```