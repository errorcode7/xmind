# lldb

https://www.dllhook.com/post/51.html

## process

### launch

process launch -v DEBUG=1 
process launch --tty -- <args> 
pro la -t -- <args>
process launch --tty=/dev/ttys006 -- <args>
pro la -t/dev/ttys006 -- <args>

### attach

process attach --pid 123 
attach -p 123 

process attach --name a.out
pro at -n a.out 

process attach --name a.out --waitfor
pro at -n a.out -w

- 用途

## image

查询可执行 程序和链接库信息
target modules的别名

add          -- Add a new module to the current target's modules.
dump         -- Commands for dumping information about one or more target modules.
list         -- List current executable and dependent shared library images.
load         -- Set the load addresses for one or more sections in a target module.
lookup       -- Look up information within executable and dependent shared library images.
search-paths -- Commands for managing module search paths for a target.
show-unwind  -- Show synthesized unwind instructions for a function.

### list

image list -f -o WhatsApp

(lldb) image lookup --address 0x1ec4
(lldb) im loo -a 0x1ec4

### lookup

找地址所在的链接库以及偏移
image lookup -a $pc
image lookup --address 0x1ec4
im loo -a 0x1ec4
查找函数
有符号(函数名)
image lookup -r -n <FUNC_REGEX>
无符号
image lookup -r -s <FUNC_REGEX>

### dump

导出所有的sections
image dump sections [lib]
导出所有符号(函数名 )
image dump symtab [lib]

## print

print/p是expression的别名

(lldb) expr (int) printf ("Print nine: %d.", 4 + 5)
print (int) printf ("Print nine: %d.", 4 + 5)

p/进制 $pc

## expression

执行一个表达式，现实并返回值
作用，相当于直接插入/修改源码：
1.动态修改内存值
2.执行函数，相当于call
3.打印变量

### -O

expr -O=po
对象描述，常用于输出objectiveC的对象

### -d

打印表达式的动态类型
OBJ
expr -d 1 -- [SomeClass returnAnObject]
CPP
 expr -d 1 -- someCPPObjectPtrOrReference

### -i

执行打断点的函数，并暂停到断点处
expr -i 0 -- function_with_a_breakpoint()

### -u

执行报错的函数，并暂停到报错处
expr -u 0 -- function_which_crashes()

### 其他

## disassemble

当前帧反汇编
dis/di

### dis  -F intel

### dis -A thumbX

thumbv4t
thumbv5
thumbv5e
thumbv6
thumbv6m
thumbv7
thumbv7f
thumbv7s
thumbv7k
thumbv7m
thumbv7em

### dis -n

指定函数

### 其他

-A <arch>（--arch <arch>）架构
-C <num-lines>（--context <num-lines>）要显示的源上下文行数。

-F <disassembly-flavor>（--flavor <disassembly-flavor>）
反汇编代码风格，只有at&t和intel。
-P <plugin>（--plugin <plugin>）
反汇编插件

-a <addr-expr>（--address <addr-expr>）指定地址会，反汇编包含该地址的函数。
-s <addr-expr>（--start-address <addr-expr>）开始地址
-e <addr-expr>（--end-address <addr-expr>）结束地址

-b（--bytes）显示操作码
-c <num-lines>（--count <num-lines>）显示的指令数。

-f（--frame）当前帧开始反汇编
-l（--line）
如果有调试行表信息，则反汇编当前帧的当前源行指令，否则反汇编pc指针附近指令。

-m（--mixed）源码汇编混合现实显示。

-n <function-name>（--name <function-name>）
反汇编指定函数
-p（--pc）反汇编当前指令指针
-r（--raw）原始反汇编，不带符号信息。

## register

register read	读取所有通用寄存器值
register read --all 所有寄存器=re r -a
register read/x 以16进制显示
register read --format i指令格式
register read $x0	读取x0寄存器值
register write $x1 10	修改x1寄存器的值为10
register write rax 123 
register write pc `$pc+8`

## memory

(memory read --size 4 --format x --count 4 0xbffff3c0
me r -s4 -fx -c4 0xbffff3c0
lx -s4 -fx -c4 0xbffff3c0

memory read/4xw 0xbffff3c0
x/4xw 0xbffff3c0
memory read --gdb-format 4xw 0xbffff3c0
读取512字节
memory read --outfile /tmp/mem.txt --count 512 0xbffff3c0
me r -o/tmp/mem.txt -c512 0xbffff3c0
x/512bx -o/tmp/mem.txt 0xbffff3c0

读取起始到终止地址
memory read --outfile /tmp/mem.bin --binary 0x1000 0x2000
me r -o /tmp/mem.bin -b 0x1000 0x2000

以下仅macOS可用
获取指定的堆内存
command script import lldb.macosx.heap
process launch --environment MallocStackLogging=1 -- [ARGS]
malloc_info --stack-history 0x10010d680

Find all heap blocks that contain a pointer specified by an expression EXPR
ptr_refs EXPR

Find all heap blocks that contain a C string anywhere in the block
cstr_refs CSTRING

## breakpoint

### set

按行
breakpoint set --file foo.c --line 12
breakpoint set -f foo.c -l 12

按函数名，缩写br s -n func 
breakpoint set --name foo
breakpoint set -n foo [-n foo1]

C++函数
Class:mfunc
OBJ函数：
breakpoint set --name "-[NSString stringWithFormat:]"
b -[NSString stringWithFormat:] 

为C++函数设置断点，使用method
breakpoint set --method foo
breakpoint set -M foo

OC里面selector的断点
breakpoint set --seletor count
breakpoint set -S count
对image(连接库)设置断点 ：
breakpoint set --shlib foo.dylib --name foo
breakpoint set -s foo.dylib -n foo
可以重复使用 --shlib来标记多个公共库

- condtion

  条件断点
  breakpoint set --name foo --condition '(int)strcmp(y,"hello") == 0'
  br s -n foo -c '(int)strcmp(y,"hello") == 0'

- regular-expression

  表达式
  函数断点
  breakpoint set --func-regex regular-expression 
  br s -r regular-expression 
  源码断点
  breakpoint set --source-pattern regular-expression --file SourceFile
  br s -p regular-expression -f file 
  指定文件/行断点
  settings set target.inline-breakpoint-strategy always
  (lldb) br s -f foo.c -l 12

### list

br l

### delete

br del 1

## watchpoint

观察点，用于确定某个地方写入的值

### set

变量被写入
watchpoint set variable global_var
wa s v global_var

内存地址被写入，my_ptr起始位置
watchpoint set expression -- my_ptr
wa s e -- my_ptr
默认范围是一个指针大小，-x byte_size可指定范围

条件观察点
watch set var global_v
watchpoint modify -c '(global_v==5)'

### list

watchpoint list
watch l

### del

## frame

栈帧，可指定frame做一些操作
bt查看当前线程的全部当前栈帧
当前帧输入：
up，上一层帧=frame select --relative=1
up 2，上2层=fr s -r2
down，下一层帧=frame select --relative=-1/fr s -r-1

### variable

当前栈帧变量，含参数
frame variable/fr v
不含参数
frame variable --no-args /fr v -a 
输出变量名
frame variable bar/fr v bar/p bar 
格式化输出
frame variable --format x bar
fr v  -f x bar

### select

### diagnose

### info

## thread

### list 

thread list

### select

thread select 1
t 1

### info

线程 信息，默认当前线程
thread info

### backtrace

-c：设置打印堆栈的帧数(frame)
-s：设置从哪个帧(frame)开始打印
-e：是否显示额外的回溯
选中线程作为当前线程，可直接bt

### step-in 

### step-over

### step-inst-in

### step-inst-over 

### step-out 

### step-scripted

### jump

### plan

### continue

### until

### return

返回当前栈，可以带个返回值
thread return <RETURN EXPRESSION>

- 用途

## gui

切换到简单图形界面模式
调试的时候比较有用
h，查看快捷键
esc,退出

## command

alias   -- Define a custom command in terms of an existing command.  Expects 'raw' input (see 'help raw-input'.)
delete  -- Delete one or more custom commands defined by 'command regex'.
history -- Dump the history of commands in this session.
                 Commands in the history list can be run again using "!<INDEX>".   "!-<OFFSET>" will re-run the command that
                 is <OFFSET> commands from the end of the list (counting the current command).
regex   -- Define a custom command in terms of existing commands by matching regular expressions.
script  -- Commands for managing custom commands implemented by interpreter scripts.
ource  -- Read and execute LLDB commands from the file <filename>.
unalias -- Delete one or more custom commands defined by 'command alias'

### alias

#command alias short  expresion
command alias cmd command
command alias history  command history
#查询关键字相关的帮助apropos keyword
command alias man apropos
#二进制镜像
command alias lib target modules  list -o -f

#调试
command alias ret thread return
command alias out  thread step-out
command alias so  thread step-out
#断点
command alias bl breakpoint list
command alias bd breakpoint delete
command alias jmp thread jump -r 
#下面不能生效
command alias jpa thread jump -r -a
command alias jpo thread jump -r -b
command alias jpl thread jump -r -l

### script

#返回python shell
script

script print "Here is some text"

- import

  command script import lldb.macosx.heap

### source

### regex

通过匹配正则定义一个已经存在的命令表达式

## format

很多地方需要指定输出格式
help format
"default"
'B' or "boolean"
'b' or "binary"
'y' or "bytes"
'Y' or "bytes with ASCII"
'c' or "character"
'C' or "printable character"
'F' or "complex float"
's' or "c-string"
'd' or "decimal"
'E' or "enumeration"
'x' or "hex"
'X' or "uppercase hex"
'f' or "float"
'o' or "octal"
'O' or "OSType"
'U' or "unicode16"
"unicode32"
'u' or "unsigned decimal"
'p' or
"pointer"
"char[]"
"int8_t[]"
"uint8_t[]"
"int16_t[]"
"uint16_t[]"
"int32_t[]"
"uint32_t[]"
"int64_t[]"
"uint64_t[]"
"float16[]"
"float32[]"
"float64[]"
"uint128_t[]"
'I' or "complex integer"
'a' or "character array"
'A' or "address"
"hex float"
'i' or "instruction"
'v' or "void"

## gdb-remote

#Attach to a remote gdb protocol server running on system "eorgadd", port 8000.
#eorgadd就是ip

gdb-remote eorgadd:8000 

#本地
gdb-remote 8000 

#Attach to a Darwin kernel in kdp mode on system "eorgadd".
kdp-remote eorgadd

## display

使用display命令设置一个表达式后，它将在每次单步后，紧接着输出被设置的表达式的的值

display 表达式
undisplay 序号n
例子： 
(lldb) display $R0
(lldb) undisplay 1

## target

create    -- Create a target using the argument as the main executable.
delete    -- Delete one or more targets by target index.
list      -- List all current targets in the current debug session.
modules   -- Commands for accessing information for one or more target modules.
select    -- Select a target as the current target by target index.
stop-hook -- Commands for operating on debugger target stop-hooks.
symbols   -- Commands for adding and managing debug symbol files.
variable  -- Read global variables for the current target, before or while running a process.

### stop-hook 

每执行一步要显示什么东西

- add

  --one-liner 指定一行断点命令
  #进入函数，指定内容将被显示
  target stop-hook add --one-liner "frame variable argc argv" 
  ta st a -o "fr v argc argv" 
  
  target stop-hook add --name main --one-liner "frame variable argc argv"
  ta st a -n main -o "fr v argc argv" 
  
  target stop-hook add --classname MyClass --one-liner "frame variable *this" 
  
  #-o 暂停会执行指令
  target stop-hook add -o "frame variable"
  ta st a -c MyClass -o "fr v *this"

- create

- delete
- disable
- enable
- list

### variable

全局或在静态变量
target variable

### symbols

添加符号文件
target symbols add

## settings

lldb设置，可用lldbinit配置，初始化调试环境
进程设置，如环境变量等

### list

查看有哪些设置

### set

settings set target.run-args 1 2 3 
settings set target.env-vars DEBUG=1
set se target.env-vars DEBUG=1 
env DEBUG=1 

#设置动态类型打印为默认
settings set target.prefer-dynamic run-target 
#反汇编风格
settings set target.x86-disassembly-flavor intel

指定编程语言
target.language
重新映射源码目录，原来编译指定的目录与新目录
settings set target.source-map /buildbot/path /my/path

#设置调试的时候查看的汇编行数
settings set stop-disassembly-count  32
stop-line-count-after   5
stop-line-count-before   5

### show

settings show target.run-args

### remove

简写为rem
settings remove target.env-vars DEBUG
set rem target.env-vars DEBUG

## 执行/调试

thread return	不再执行往下代码，直接从当前调用栈返回一个值

### run/r 

lldb “可执行路径”，然后
run
ctrl-c
continue/c下一个断点

### step/s 

over in/步入
等同thread step-in

### next/n 

step over/下一步
等同thread step-over

### si 

等同thread step-inst

### ni 

等同thread step-inst-over

### finish 

Step out
跳出当前栈
等同thread step-out

###  thread return

thread return <RETURN EXPRESSION>

## 缩写对应

输入:h,可以查看常用命令及别名
alias

breakpoint :br/b
list:li
delete:del
disable:dis
enable:ena
backtrace:bt 调用栈
run :r
expression -O:po

## 概念

PC/program counter (instruction pointer/EIP)

dSYM符号文件

函数内存中的地址=二进制装载地址+函数在二进制中的偏移
iamge list -o -f
-o 表示二进制文件在虚拟内存中的偏移

### ptrace

https://www.jb51.net/article/143425.htm
防止调试

*XMind: ZEN - Trial Version*