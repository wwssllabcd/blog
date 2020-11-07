---
layout: post
title: "一個簡單的 Linux Kernel Module"
date: 2012-11-13 19:14
comments: true
tags: [How-To, Lunux kernel]
---

以下會建立一個非常簡單的 linuxk kernel，只會包含兩個 funciton -- 即 init 與 exit
分別在 module 載入及退出的時候會呼叫到

建立 hello.c
--------------------
這邊不免俗的使用 hello module，先建立 hello_init 與 hello_exit，並且印出訊息  

當使用 insmod hello.ko 時，會自動的呼叫到 hello_init 這個 function，
而使用 rmmod 時，也會呼叫到 hello_exit，
也就是說，如果順利載入的話，會在 dmesg 裡面看到這些訊息。

<!--more-->

{% codeblock hello .c %}

	#include <linux/init.h>
	#include <linux/module.h>
	MODULE_LICENSE("Dual BSD/GPL");

	static int hello_init(void)
	{
		printk(KERN_INFO "Hello kernel\n");
		return 0;
	}

	static void hello_exit(void)
	{
		printk(KERN_INFO "Goodbye\n");
	}

	module_init(hello_init);
	module_exit(hello_exit);

{% endcodeblock %}


建立 MakeFile
------------------
此段列出 hello.c 的 Makefile，只介紹用到的指令，在最後也會補充一些 Makefile 的其他指令

{% codeblock Makefile %}

	#
	# Makefile for kernel test
	#
	PWD         := $(shell pwd) 
	KVERSION    := $(shell uname -r)
	KERNEL_DIR   = /usr/src/linux-headers-$(KVERSION)/

	MODULE_NAME  = hello
	obj-m       := $(MODULE_NAME).o   

	all:
		make -C $(KERNEL_DIR) M=$(PWD) modules
	clean:
		make -C $(KERNEL_DIR) M=$(PWD) clean

{% endcodeblock %}

這邊解釋一下 Makefile 的內容  

`PWD := $(shell pwd)`：取得目前目錄  
`KVERSION := $(shell uname -r)`：取得 kernel 版本號  
`KERNEL_DIR = /usr/src/linux-headers-$(KVERSION)/`：因為編譯 kernel 需要 include kernel 目錄，所以這邊也要定義  
`obj-m`：表示需要編繹成模組的目標檔案名集合，編譯的方式為編譯成區塊，這裡要注意的是，他同時也是代表被編譯的檔案名稱，
如果要被編譯的是 Hello.c，那這邊就要填 `obj-m := hello.o`，此外還有 obj-y，代表編譯進去內核。

編譯指令如下
	make -C $(KERNEL_DIR) M=$(PWD) modules

`-C:`：跟 make 講 這次 kernel module include 的目錄在哪  
`$(KERNEL_DIR)`：Makefile 中可以使用變數，一般變數大寫，在引用變數時，採用小括弧擴起變數名前加（$）符號來用。  
`M=$(PWD)`：描述那個目錄要被編譯 (早期的指令是SUBDIRS)，M=$(PWD)，個人猜測再作遞迴 make 的時候，會需要回到原始目錄。
這是定義在 kernel 的 make file 中，要詳細內容可以去看 kernel 的 make file  

編譯及載入 module
------------------
執行 make 來編譯 hello.ko 之後，使用 insmod 來載入我們的 module
	sudo insmod hello.ko

使用 dmesg 來察看
	[35451.985643] Hello, kernel

代表我們成功的把 module 載入
這邊列出幾項觀察 moudle 的指令
	insmod: 載入mod
	lsmod: 列出mod , 如lsmod |grep hello
	rmmod: 移除mod

Make 語法簡介
------------------
:= 語法  
指定變數的語法，make 會先把整個檔案展開，找出該變數最後一個被指定的值並且 assign 給他
也就是說
	x := foo
	y := $(x)
	x := foobar
則 Y 的結果為 foobar

?= 語法  
指定變數的語法，如果變數已經被指定過，則不會再被指定

	
QA
-------------
出現以下的 Error
	Make File Error : missing separator. Stop.
	Makefile:9: *** missing separator. Stop.
	
檢查一下make file 是否混入了空白，一定要用Tab
記得 gcc 前面要用TAB ，否則 make 會 fail 
	
	