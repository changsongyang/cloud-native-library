---
title: "第二章：修改内核很困难"
weight: 2
type: book
---

由于 eBPF 允许在 Linux 内核中运行自定义代码，在解释 eBPF 之前我需要确保你对内核的作用有所了解。然后我们将讨论为什么在修改内核行为这件事情上，eBPF 改变了游戏规则。

## Linux 内核

Linux 内核是应用程序和它们所运行的硬件之间的软件层。应用程序运行在被称为**用户空间**的非特权层，它不能直接访问硬件。相反，应用程序使用系统调用（syscall）接口发出请求，要求内核代表它行事。这种硬件访问可能涉及到文件的读写，发送或接收网络流量，或者只是访问内存。内核还负责协调并发进程，使许多应用程序可以同时运行。

应用程序开发者通常不直接使用系统调用接口，因为编程语言给了我们更高级别的抽象和**标准库**，开发者更容易掌握这些接口。因此，很多人都不知道在程序运行时内核做了什么。如果你想了解内核调用频率，你可以使用 strace 工具来显示程序所做的所有系统调用。这里有一个例子，用 cat 从文件中读取 hello 这个词并将其写到屏幕上涉及到 100 多个系统调用：

```bash
liz@liz-ebpf-demo-1:~$ strace -c cat liz.txt
hello
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- -------------
0.00    0.000000	0         5	  read
0.00    0.000000	0         1	  write
0.00    0.000000	0        21	  close
0.00    0.000000	0        20	  fstat
0.00    0.000000	0        23	  mmap
0.00    0.000000	0         4	  mprotect
0.00    0.000000	0         2	  munmap
0.00    0.000000	0         3	  brk
0.00    0.000000	0         4	  pread64
0.00    0.000000	0         1	1 access
0.00    0.000000	0         1	  execve
0.00    0.000000	0         2	1 arch_prctl
0.00    0.000000	0         1	  fadvise64
0.00    0.000000	0        19	  openat
------ ----------- ----------- --------- --------- -------------
100.00    0.000000              107     2 total
```

由于应用程序在很大程度上依赖于内核，这意味着如果我们能够观测到应用程序与内核的交互，我们就可以了解到很多关于它的行为方式。例如，如果你能够截获打开文件的系统调用，你就可以准确地看到任何应用程序访问了哪些文件。但是，怎么才能做到这种拦截呢？让我们考虑一下，如果我们想修改内核，添加新的代码，在系统调用时创建某种输出，会涉及到什么问题。

## 向内核添加新功能

Linux 内核很复杂，在写这篇文章的时候有大约 3000 万行代码 [^1]。对任何代码库进行修改都需要对现有的代码有一定的熟悉，所以除非你已经是一个内核开发者，否则这很可能是一个挑战。

但你将面临的挑战并不是纯粹的技术问题。Linux 是一个通用的操作系统，在不同的环境和情况下使用。这意味着，如果你想对内核进行修改，这并不是简单地写出能用的代码。它必须被社区（更确切地说，是被 Linux 的创造者和主要开发者 Linus Torvalds）接受，你的改变将是为了大家的更大利益。而这并不是必然的——提交的内核补丁只有三分之一被接受 [^2]。

假如，你已经想出了一个好方法来拦截打开文件的系统调用。经过几个月的讨论和一些艰苦的开发工作，让我们想象一下，这个变化被接受到内核中。很好！但是，要到什么时候它才会出现在每个人的机器上呢？

每隔两三个月就会有一个新的 Linux 内核版本，但是即使一个变化已经进入了其中一个版本，它仍然需要一段时间才能在大多数人的生产环境中使用。这是因为我们大多数人并不直接使用 Linux 内核——我们使用像 Debian、Red Hat、Alpine、Ubuntu 等 Linux 发行版，它们将 Linux 内核的一个版本与其他各种组件打包在一起。你可能会发现，你最喜欢的发行版使用的是几年前的内核版本。

例如，很多企业用户都采用红帽 ® Enterprise Linux®（RHEL）。在撰写本文时，目前的版本是 RHEL8.5，发行日期为 2021 年 11 月。这使用的是基于 4.18 版本的内核。这个内核是在 2018 年 8 月发布的。

如 [图 2-1](#figure-f-2-1) 中的漫画所示，将新功能从想法阶段转化为生产环境中的 Linux 内核，需要数年时间 [^3]。

{{<figure src="../images/f-2-1.jpg" title="图 2-1. 向内核添加功能（Isovalent 公司的 Vadim Shchekoldin 绘制的漫画）" alt="图 2-1" id="f-2-1" >}}

## 内核模块

如果你不想等上好几年才把你的改动写进内核，还有一个选择。Linux 内核可以接受内核模块（module），这些模块可以根据需要加载和卸载。如果你想改变或扩展内核行为，编写一个模块是理所当然的。在我们打开文件的系统调用的例子中，你可以写一个内核模块来实现。

这里最大的挑战是，这仍然是全面的内核编程。用户在使用内核模块时历来非常谨慎，原因很简单：如果内核代码崩溃了，就会导致机器和上面运行的所有东西瘫痪。用户如何确保内核模块可以安全运行呢？

“安全运行”并不仅仅意味着不崩溃——用户想知道内核模块从安全角度来看是否安全。是否包括攻击者可以利用的漏洞？我们是否相信模块的作者不会在其中加入恶意代码？因为内核是特权代码，它可以访问机器上的一切，包括所有的数据，所以内核中的恶意代码将是一个令人担忧的严重问题。这也适用于内核模块。

考虑到内核的安全性，这就是为什么 Linux 发行商需要这么长时间来发布新版本的一个重要原因。如果其他人已经在各种情况下运行了数月或数年的内核版本，那些漏洞可能已经被修复。发行版的维护者可以有一些信心，他们提供给用户 / 客户的内核是经过加固的，也就是说，可以安全运行。

eBPF 提供了一个非常不同的安全方法：**eBPF 验证器（verifier）**，它确保一个 eBPF 程序只有在安全运行的情况下才被加载。

## eBPF 验证和安全

由于 eBPF 允许我们在内核中运行任意代码，需要有一种机制来确保它的安全运行，不会使用户的机器崩溃，也不会损害他们的数据。这个机制就是 eBPF 验证器。

验证器对 eBPF 程序进行分析，以确保无论输入什么，它都会在一定数量的指令内安全地终止。例如，如果一个程序解除对一个指针的定义，验证器要求该程序首先检查指针，以确保它不是空的（null）。解除对指针的引用意味着 "查找这个地址的值"，而空值或零值不是一个有效的查找地址。如果你在一个应用程序中解引用一个空指针，该应用程序就会崩溃；而在内核中解引用一个空指针则会使整个机器崩溃，所以避免这种情况至关重要。

验证也确保了 eBPF 程序只能访问其应该访问的内存。例如，有一个 eBPF 程序在网络堆栈中触发，并通过内核的 **套接字缓冲区（socket buffer）**，其中包括正在传输的数据。有一些特殊的辅助函数，如 `bpf_skb_load_bytes()`，这个 eBPF 程序可以调用，从套接字缓冲区读取字节数据。另一个由系统调用触发的 eBPF 程序，没有可用的套接字缓冲区，将不允许使用这个辅助函数。验证器还确保程序只读取套接字缓冲区内的数据字节——它不允许访问任意的内存。这里的目的是确保 eBPF 程序是安全的。

当然，仍然有可能编写一个恶意的 eBPF 程序。如果你可以出于合法的原因观测数据，你也可以出于非法的原因观测它。要注意只从可验证的来源加载可信的 eBPF 程序，并且只将管理 eBPF 工具的权限授予你信任的拥有 root 权限的人。

## eBPF 程序的动态加载

eBPF 程序可以动态地加载到内核中和从内核中删除。不管是什么原因导致该事件的发生，一旦它们被附加到一个事件上就会被该事件所触发。例如，如果你将一个程序附加到打开文件的系统调用，那么只要任何进程试图打开一个文件，它就会被触发。当程序被加载时，该进程是否已经在运行，这并不重要。

这也是使用 eBPF 的可观测性或安全工具的巨大优势之一——即刻获得了对机器上发生的一切事件的可见性。

此外，如 [图 2-2](#figure-f-2-2) 所示，人们可以通过 eBPF 非常快速地创建新的内核功能，而不要求其他 Linux 用户都接受同样的变更。

{{<figure src="../images/f-2-2.jpg" title="图 2-2. 用 eBPF 添加内核功能（漫画：Vadim Shchekoldin Isovalent）" alt="图 2-2" id="f-2-2" >}}

现在你已经看到了 eBPF 是如何允许对内核进行动态的、自定义的修改的，让我们来看看如果你想写一个 eBPF 程序会涉及哪些内容。

## 参考

[^1]: Linux 5.12 有大约 2880 万行代码，Phoronix（2021 年 3 月）。
[^2]:  Yujuan Jiang 等人，《我的补丁能用了吗？要多久？》（论文，2013 年）。根据这篇研究论文，33% 的补丁将在 3 - 6 个月后被接受。
[^3]:  值得庆幸的是，现有功能的安全补丁会更快地被提供。