---
layout: post
title: "[转]GDB使用介绍"
date: 2012-06-19 21:06
comments: true
categories: [Linux]
---
本文转自于[CSDN][1], 仅作排版处理。

### GDB介绍

在Linux下最强大的Debug工具就是GDB了，许多IDE都集成了GDB进行调试。使用源代码级调试能够更直接的进行调试，效率明显高于输出Log信息。但目前无论是Mac下的XCode，还是Linux下的其它集成工具，对于调试库函数都是相当困难的，如果直接使用GDB这些问题就迎刃而解。我们首先来探讨一下GDB的基础知识。 

### GDB调试流程 

GDB调试依赖于编译器输出的调试信息,所以进行调试前必须确定GCC输出了调试信息。
<!--more-->
#### 生成符号文件

使用GCC编译时需要生成相应的调试信息，编译时可以使用-g选项，详细内容参GCC Manual, Section 3.9

> -g  表示将Symbol Table以系统原生格式直接生成到可执行文件中，-g选项最好不要同-O一起使用，因为代码经过优化后有时会同源代码差异很大，可能找不到指定的变量等等。

> -ggdb  表示将专门为GDB调试使用生成调试信息，它会包括很多GDB的扩展信息。

其它的Symbol Table的格式还有COFF，DWARF，Stabs等。Mac OS默认为DWARF，DWARF也是基于COFF（Common Object File Format）实现。 

输出的调试信息的多少由三个等级:

> -g[level]  默认为2

>   0  表示不生成任何调试信息

>   1  表示生成最少的调试信息，不提供局部变量及源代码行列等信息。

>   2  标准模式

>   3  较2而言，包含了宏定义等额外信息 

其它同调试相关的编译选项还有:

> -p 生成额外的代码用于输出profile信息，用于另一个工具程序使用:prof

> --coverage 用于统于代码的覆盖率。 

> --ftest-coverage 类似上面的--coverage

> -d*  用于Dump一些有用的信息，详细内容参GDB Manual. 

#### 启动GDB进行调试

下面的过程，我都以下面的工程进行解释:

目标程序: text2bin

源代码: text2bin.c

功能: 将文本文件转成二进制文件

使用方法: `./text2bin txt_file_name  [offset]`，txt_file_name为源文本文件名，offset指定忽略左侧多少字节。 

A.调试应用程序

(1)启动  

直接在命令行下输入`gdb ./text2bin`或者运行gdb后输入`file ./text2bin`都可以加载指定的应用程序。

GDB会显示加载Symbols的过程，注意如果没有出现加载text2bin的调试信息的过程，就是表明无法获取调试信息!

> Reading symbols for shared libraries ... done

> Reading symbols from /Horky/Project/WINBASE/WINBASE/TextToRaw/text2bin...

> warning: UUID mismatch detected between:

>     /Horky/Project/WINBASE/WINBASE/TextToRaw/text2bin 

(2)设置断点

除了断点外还有Watchpoints(观测点)及Catchpoints (异常捕捉点)

输入`b`或`break`加上断点位置或断点函数名，如

    b main  #在main函数入口设置断点
    b text2bin.c:50  #在源代码第50行设置断点 

如果需要查看断点信息可以使用指令:

    info breakpoints 

清除所有断点使用指令:

    clear

 清除特定的断点使用指令:

     clear text2bin.c:50
     clear main 

在调试过程可以使用`disable`及`enable`开关某个特定的断点，如`enable 2`及`disable 2`开关第2个断点，在使用`info b`查看断点时，注意其中`Enb`栏位的变化。 

对于观测点(Watchpoints)，是指在某个条件下触发的断点，如text2bin中77行:

> Buffer2[nCount++] = ConvertTextToInt(sData); 

我们要查看当nCount为10时的运行状况,我们可以通过下面的步骤完成:

a. 执行`b 77`,返回这个断点号是3

b. 执行`condition 3 nCount=10`

过程如下:

> (gdb) b 77

> Breakpoint 3 at 0x1c73: file text2bin.c, line 77.

> (gdb) condition 3 nCount=10

> (gdb) info breakpoints

> Num Type           Disp Enb Address    What

> 1   breakpoint     keep y   0x00001e1e in ConvertTextToInt at text2bin.c:124

>     breakpoint already hit 1 time

> 2   breakpoint     keep y   0x00001e25 in ConvertTextToInt at text2bin.c:125

> 3   breakpoint     keep y   0x00001c73 in main at text2bin.c:77

>     stop only if nCount = 10 

这样就可以控制当nCount为10时在77行处中断。 

如果在调试时，需在下面若干行代码后追加一个断点，在指定位置可以使用偏移量来指定断点，如`b +5`及`b -5`，即表示在当前行的后五行及前五行位置设置断点。 

(3)控制调试过程

在开始时需要告诉GDB目标程序是哪一个，可以用`gdb ./text2bin`，也可以在启动gdb后使用指令来指定:

    file ./text2bin

运行则使用`r[un]`指令，可以同时带上参数，如

    r ./expert.txt 7 

在调试过程中需要有一系列操作控制调试的过程: c[ontinue]/fg(fg是foreground的缩写)

从断点状态恢复程序的执行。 

    s[tep] [count] #单步执行 (step in)

    n[ext] [count] #单步执行,跳过函数 (step out)

    u[ntil] [location] #执行到某个位置，当遇到循环时可以使用此指令方便地跳转到指定的位置。

    finish #执行到当前函数结束位置为止,同时显示函数返回值

    backtrace #查看当前位置的被调用路径 

(4)监测变量及内存

简单地显示变量的值可以使用`print`指令直接输出:

    p[rint] [expression]

如输出main函数中的文件名变量`p argv[0]`，输出ConvertTextToInt函数中n的值`p n`或`p ConvertTextToInt::n` 

查看内存内容时则使用`x`指令:

    x /nfu addr  #以指定格式显示内存内容

    x addr  #显示指定地址处理内存内容

    x #显示当前数据段内容 

如以字串形显示某内存内容的指令为:

    x /sb 0xbffff80e

以数据形式显示某内存中5个字节的内容的指令为:

    x /5db 0xbffff80e 

设定local variable watch，用来在每条执行后显示某些变量的值，可以使用`display`指令来指定，如:

    display S
    display /sb 0xbffff707 

去除时使用`undisplay #`，#为display列表中的序号 

display后面所带的参数同`x`指令:

> n 表示repeat count

> f 表示格式,分为:

>   x  十六制数据

>   d  带符号之整型数据

>   u  无符号整型数据

>   o  八进制整型数据

>   t  二进制数据

>   a 以地址格式显示,包括十六进制及偏移量

>   c 以字符形式输出

>   f  小数位输出

>   s  以字串形式输出 

> u表示大小单位:

>   b  Bytes

>   h  Halfwords

>   w  Words

>   g  Giant words (eight bytes)  

(5)查看源代码

使用`l[ist]`指令就可查看源代码了，如:

    l 150 #查看当前代码的第150行

    l text2bin.c:150  #查看text2bin.c的150行

    l main  #查看main函数内容

查看时如果需要翻页直接回车，默认GDB一次显示10行，我们也可以通过`set linesize [count]`进行调整。 

B.如何调试动态库或静态库

当调试库函数时，需要透过主程序调用的形式来挂载，所以不能直接使用GDB对目标库进行调试，而是需要`attach`指定的父进程，然后再进行测试。 这里有两种情况:

(1)主程序启动时自动加载库，此时使用GDB挂载时也会自动加载相应的调试信息。

(2)主程序动态加载库，对于这种情况则需要另外使用`symbol filename`来加载特定的调试信息。 

提醒:当进行多线程调试时，一定要确保能找出真正的主线程。

断开主线程时，使用`detach`指令来完成。    

C.如何调试多线程的程序

我们在写程序时常常会有多线程的运用, 比如有些程序中读取数据及数据处理，就是通过两个线程来完成的。对多线程进行调试最大的难点在于线程的同步问题。

GDB提供了一套指令针对多线程:

    info threads #查看当前有多少线程

    thread n #切换到指定线程

    set print thread-events on/off #设定是否打印线程状态

当设置中断时，也可以专为某个线程设置，如

    b[reak] [location] thread n

即表示为n线程在location处设置断点，这样就可以进行线程别的调试。 

### 技巧

(1)在GDB中如果需要调用外部程序可以使用`shell [command]`来完成。

(2)当源代码目录被移动了，或者在另一台PC上调试，GDB不能通过Debug信息找到源代码时，可以使用`dir[ectory]`来指定搜索的目录。 

### GDB的前端程序(GDB frontend)

使用命令行是显得不方便了，所以我们可以选择一些GDB前端程序:

>  DDD   [GNU] (<http://www.gnu.org/software/ddd>  目前功能最为强大的GDB前端程序)
   
>  Nemiver  [GNOME]  (<http://home.gna.org/nemiver/>) 
 
>  Kdbg   [KDE] (<www.kdbg.org>)  

>  Insight  [Wirte in Tcl/Tk] (<http://sourceware.org/insight>)  

>  Emacs (不用介绍了!)

一般的IDE也带有GDB frontend程序，如XCode，KDevelop，Anjuta，Eclipse。

GDB Frontend都是通过伪终端(pseudo-terminal)的方式来实现，有兴趣可以了解一下。 

### 扩展GDB的功能

已经有人在通GDB进行代码覆盖率测试，事实上我们也可以通过类似GDB的方式读取Debug信息中的符号表来进行语法检查。有关Debug信息的存放，可以使用`objdump -x`或`readelf -a`来查看其中的不同，这有助更好的理解程序的结构。 

需求是多样的，GDB本身提供了两种方式来扩展GDB，一种为组合GDB的指令，类似宏的方式；另一种方式则是功能强大的python脚本。

(1)在GDB环境下使用define指令来定义一个指令，如

    define localv
      info scope $arg0
    end

这样我们在使用时，想查看main函数中的所有的变量，就可以通过下面的指令完成:

> (gdb) localv main 
 
> Scope for main:
  
> Symbol argc is at the address (reg 5 + 8), length 4.
  
> Symbol argv is at the address (reg 5 + 12), length 4.
  
> Symbol fpSrc is at the address (reg 5 + -44), length 4.
  
> ...... 

如果这样的指令非常好用，每次调试时都定义一次不太现实。所以GDB允许将这些操作定义在一个文本文件中，然后在GDB中使用`source [command_file]`来执行，如`source /TestData/localv.cmd`。在执行过程中GDB不会显示每个指令的执行结果,如果需要显示就在`source`加`-v`来打开。 

除了组织指令集外，还有另一种有用的自定义指令: Hooks. GDB允许用户指定在特定的GDB指令执行前后执行一段自定义指令。比如，如果希望在设置断点前后都显示当前断点状况，就可以定义两个如下指令:

> (gdb) define hook-break
  
> Type commands for definition of "hook-break". 
 
> End with a line saying just "end". 
 
> \>info b
  
> \>end
  
> (gdb) define hookpost-break
  
> Type commands for definition of "hookpost-break".
  
> End with a line saying just "end".
  
> \>info b 
 
> \>end 

然后执行`b[reak]`指令时就可以看到类似下面的输出:

> (gdb) b GetFileSize

> Num Type           Disp Enb Address    What

> 1   breakpoint     keep y   0x00001a68 in main at text2bin.c:25

> 2   breakpoint     keep y   0x00001e1e in ConvertTextToInt at text2bin.c:124

> Breakpoint 3 at 0x1d86: file text2bin.c, line 108.

> Num Type           Disp Enb Address    What

> 1   breakpoint     keep y   0x00001a68 in main at text2bin.c:25

> 2   breakpoint     keep y   0x00001e1e in ConvertTextToInt at text2bin.c:124

> 3   breakpoint     keep y   0x00001d86 in GetFileSize at text2bin.c:108 

hook及hookpost即表示在某个指令的前后。后面的指令一定要使用GDB指令的全写，如上面就不能写成`define hook-b`或`define hookpost-b`。 

如果需要更为详细的资料，请参考GDB Manual,20. Extending GDB 

(2)在GDB环境可以直接调用python，如在GDB环境下执行`python print 23`

使用Python编写脚本同上面定义指令集是类似的，可以执行指令`python`, GDB就会要求输入python脚本，并以end为结束标志。

GDB为编写Python提供了一个新的模块gdb， 在脚本中可以进行引用，其中包括了几个主要的指令:

    execute command    #command是GDB CLI(Command Line Interface)指令字串.

    get_parameter parameter  #获取一项GDB的参数,诸如上面提到的linesize.

    write string  #输出一个字串到GDB输出窗口

    flush    #Flush当前GDB输出流 

只有当编译GDB时指定了`—with-python`时，GDB才会支持`python`指令. 

### 参考文档:

(1) 使用GDB进行代码覆盖率测试

       <http://www.ibm.com/developerworks/cn/linux/l-cn-gdb/>

(2) 使用 GDB 调试 Linux 软件

       <http://www.ibm.com/developerworks/cn/linux/sdk/gdb/>

(3) 掌握 Linux 调试技术

       <http://www.ibm.com/developerworks/cn/linux/sdk/l-debug/>

(4) 用GDB调试程序

       <http://docs.chinalinuxpub.com/doc/pro/gdb.html>

(5) GDB调试精粹及使用实例

       <http://fanqiang.chinaunix.net/program/other/2006-07-14/4834.shtml>

(6) GDB的官方文档

       <http://www.gnu.org/software/gdb/documentation/>

(7) GDB指令参考 (可以打出来方便查询)

       <http://users.ece.utexas.edu/~adnan/gdb-refcard.pdf>

[1]: http://blog.csdn.net/horkychen/article/details/7676024