# 跟我一起写Makefile
    Makefile文件告诉make命令怎样去编译,链接程序
## Makefile 规则
    target ... : prerequisites ... 
    command
    ... 
    ...
    
    target 也就是一个目标文件，可以是 Object File，也可以是执行文件。还可以是一个标签（Label），对于标签这种特性，在后续的“伪目标”章节中会有叙述。 
    prerequisites 就是要生成那个 target 所需要的依赖。 command 也就是 make 需要执行的命令。
    prerequisites中如果有一个以上的文件比 target 文件要新的话，command 所定义的命令就会被执行。

## Makefile如何工作的
    在默认的方式下，也就是我们只输入 make 命令。那么， 
    1、make 会在当前目录下找名字叫“Makefile”或“makefile”的文件。 
    2、如果找到，它会找文件中的第一个目标文件（target）

## Makefile中使用变量
    我们声明一个变量，叫 objects 能够表示 obj 文件就行
    objects = main.o kbd.o #定义变量
    $(objects) 的方式来使用这个变量 #使用变量


## make自动推导 (make的隐晦规则)
    GNU 的 make 很强大，它可以自动推导文件以及文件依赖关系后面的命令，于是我们就没必要去在每一个[.o]文件后都写上类似的命令，make 会自动识别，并自己推导命令。 
 
    只要 make 看到一个[.o]文件，它就会自动的把[.c]文件加在依赖关系中，如果 make找到一个 whatever.o，那么 whatever.c，就会是 whatever.o 的依赖文件。
    并且 cc -c whatever.c 也会被推导出来，于是，我们的 makefile 再也不用写得这么复杂

## 清空目标文件的规则
    .PHONY : clean 
    clean : 
    -rm edit $(objects)
    .PHONY 意思表示 clean 是一个“伪目标”
    而在 rm 命令前面加了一个小减号的意思就是，也许某些文件出现问题，但不要管，继续做后面的事。

## Makefile显式规则，隐晦规则
    显式规则说明了，如何生成一个或多的的目标文件
    隐晦规则是Makefile的自动推导功能

## 引用其它的 Makefile
    在 Makefile 使用 include 关键字可以把别的 Makefile 包含进来
    include 的语法是： include <filename>
    include foo.make a.mk //引入多个makefile文件
    make 会在当前目录下首先寻找，如果当前目录下没有找到，那么，make 还会在下面的几个目录下找：
        1、如果 make 执行时，有“-I”或“--include-dir”参数，那么 make 就会在这个参数 所指定的目录下去寻找。 
        2、如果目录<prefix>/include（一般是：/usr/local/bin 或/usr/include）存在的话，make也会去找。

## make 工作时的执行步骤如下：
    1、读入所有的 Makefile。 
    2、读入被 include 的其它 Makefile。 
    3、初始化文件中的变量。 
    4、推导隐晦规则，并分析所有规则。 
    5、为所有的目标文件创建依赖关系链。 
    6、根据依赖关系，决定哪些目标要重新生成。 
    7、执行生成命令。

## Make 书写规则
    如果命令太长，你可以使用反斜框（‘\’）作为换行符。
	“#”是注释符
	
## Make文件搜索路径
	Makefile 文件中的特殊变量“VPATH”，在当前目录找不到依赖文件和目标文件的情况下，到所指定的目录中去找寻文件
	VPATH = src:../headers
	指定两个目录，“src”和“../headers”，make 会按照这个顺序进行搜索，目录由“冒号”分隔。

	make 的“vpath”关键字
	vpath %.h ../headers
	该语句表示，要求 make 在“../headers”目录下搜索所有以“.h”结尾的文件

## Make 伪目标
	使用一个特殊的标记“.PHONY”来显示地指明一个目标是“伪目标”

## Make 静态模式
	语法:
	<targets ...>: <target-pattern>: <prereq-patterns ...> 
	<commands>
	
	targets 定义了一系列的目标文件，可以有通配符。是目标的一个集合。 
	target-parrtern 是指明了 targets 的模式，也就是的目标集模式。 
	prereq-parrterns 是目标的依赖模式，它对 target-parrtern 形成的模式再进行一次依赖目标的定义。
	
	举例:
	如果<target-parrtern>定义成“%.o”，意思是我们的<target>集合中都是以“.o”结尾的，
	而如果我们的<prereq-parrterns>定义成“%.c”，意思是对<target-parrtern>所形成的目标集进行二次定义，
	其计算方法是，取<target-parrtern>模式中的“%”（也就是去掉了[.o]这个结尾），并为其加上[.c]这个结尾，形成的新集合
	objects = foo.o bar.o 
	all: $(objects) 
	$(objects): %.o: %.c 
	$(CC) -c $(CFLAGS) $< -o $@

	展开如下：
	foo.o : foo.c 
	$(CC) -c $(CFLAGS) foo.c -o foo.o 
	bar.o : bar.c 
	$(CC) -c $(CFLAGS) bar.c -o bar.o

## Make 自动生成依赖性
	C/C++编译器都支持一个“-M”的选项，即自动找寻源文件中包含的头文件，并生成一个依赖关系。
	cc -M main.c
	
## Make 显示命令
	通常，make 会把其要执行的命令行在命令执行前输出到屏幕上。当我们用“@”字符在命令行前，那么，这个命令将不被 make 显示出来
	如果 make 执行时，带入 make 参数“-n”或“--just-print”，那么其只是显示命令，但不会执行命令，这个功能很有利于我们调试我们的 Makefile
	make 参数“-s”或“--slient”则是全面禁止命令的显示

## Make 命令执行
	如果你要让上一条命令的结果应用在下一条命令时，你应该使用分号分隔这两条命令
	cd /home/hchen; pwd

## Make 命令出错
	忽略命令的出错，我们可以在 Makefile 的命令行前加一个减号“-”
	给 make 加上“-i”或是“--ignore-errors”参数，那么，Makefile 中所有命令都会忽略错误
	如果一个规则是以“.IGNORE”作为目标的，那么这个规则中的所有命令将会忽略错误
	make 的参数的是“-k”或是“--keep-going”，这个参数的意思是，如果某规则中的命令出错了，那么就终目该规则的执行，但继续执行其它规则。

## Make 嵌套执行
	有一个子目录叫 subdir，这个目录下有个 Makefile 文件，来指明了这个目录下文件的编译规则。那么我们总控的 Makefile 可以这样书写：
	subsystem: 
	cd subdir && $(MAKE) 或者 $(MAKE) -C subdir 
	
	总控 Makefile 的变量可以传递到下级的 Makefile 中（如果你显示的声明），但是不会覆盖下层的 Makefile 中所定义的变量，除非指定了“-e”参数。
	如果你要传递变量到下级 Makefile 中，那么你可以使用这样的声明： 
		export <variable ...> 
	如果你不想让某些变量传递到下级 Makefile 中，那么你可以这样声明： 
		unexport <variable ...>

	有两个变量，一个是 SHELL，一个是 MAKEFLAGS，这两个变量不管你是否 export，其总是要传递到下层 Makefile 中
	如果你不想往下层传递参数，那么，你可以这样来： 
	subsystem: 
	cd subdir && $(MAKE) MAKEFLAGS=
	
	“-w”或是“--print-directory”会在make 的过程中输出一些信息，让你看到目前的工作目录
	make: Entering directory `/home/hchen/gnu/make'
	make: Leaving directory `/home/hchen/gnu/make'

	系统变量“MAKELEVEL”，其意思是，如果我们的 make 有一个嵌套执行的动作（参见前面的“嵌套使用 make”），这个变量会记录了我们的当前 Makefile 的调用层数

## Make 定义命令包
	如果 Makefile 中出现一些相同命令序列，那么我们可以为这些相同的命令序列定义一个变量。定义这种命令序列的语法以“define”开始，以“endef”结束.
	define run-yacc 
	yacc $(firstword $^) 
	mv y.tab.c $@ 
	endef

	foo.c : foo.y 
	$(run-yacc)
	“run-yacc”是这个命令包的名字，其不要和 Makefile 中的变量重名。在“define”和“endef”中的两行就是命令序列。


## Make 使用变量
	变量在声明时需要给予初值，而在使用时，需要给在变量名前加上“$”符号，但最好用小括号“（）”或是大括号“{}”把变量给包括起来。
	如果你要使用真实的“$”字符，那么你需要用“$$”来表示。

	
## 注意 Make变量的递归定义
	CFLAGS = $(include_dirs) -O 
	include_dirs = -Ifoo -Ibar
	当“CFLAGS”在命令中被展开时，会是“-Ifoo -Ibar -O”。但这种形式也有不好的地方，那就是递归定义

	可以使用 make 中的另一种用变量来定义变量的方法。
	这种方法使用的是“:=”操作符，前面的变量不能使用后面的变量，只能使用前面已定义好了的变量

	FOO ?= bar
	其含义是，如果 FOO 没有被定义过，那么变量 FOO 的值就是“bar”，如果 FOO 先前被定义过，那么这条语将什么也不做

	“+=”操作符给变量追加值
	

## Make 变量高级用法
	变量值的替换。我们可以替换变量中的共有的部分，其格式是“$(var:a=b)”或是“${var:a=b}”，其意思是，把变量“var”中所有以“a”字串“结尾”的“a”替换成“b”字串	

	foo := a.o b.o c.o 
	bar := $(foo:.o=.c) 
	这个示例中，我们先定义了一个“$(foo)”变量，而第二行的意思是把“$(foo)”中所有以“.o”字串“结尾”全部替换成“.c”，所以我们的“$(bar)”的值就是“a.c b.c c.c”。

## Make override 指示符 
	如果有变量是通常 make的命令行参数设置的，那么 Makefile中对这个变量的赋值会被忽略。
	如果你想在 Makefile 中设置这类参数的值，那么，你可以使用“override”指示符。 override <variable> = <value>

## Make 多行变量
	使用 define 关键字设置变量的值可以有换行，这有利于定义一系列的命令
	define 指示符后面跟的是变量的名字，而重起一行定义变量的值，定义是以 endef 关键字结束。
	变量的值可以包含函数、命令、文字，或是其它变量
	define two-lines 
	echo foo 
	echo $(bar) 
	endef

## Make 环境变量
	make 运行时的系统环境变量可以在 make 开始运行时被载入到 Makefile 文件中，
	但是如果 Makefile 中已定义了这个变量，或是这个变量由 make 命令行带入，那么系统的环境变量的值将被覆盖。
	如果 make 指定了“-e”参数，那么，系统环境变量将覆盖 Makefile 中定义的变量

## Make 目标变量
	目标变量(“Target-specific Variable”)作用范围只在这条规则以及连带规则中，所以其值也只在作用范围内有效。而不会影响规则链以外的全局变量的值
	语法
	<target ...> : <variable-assignment>
	<variable-assignment>可以是前面讲过的各种赋值表达式，如“=”、“:=”、“+=” 或是“？=”
	我们设置了这样一个变量，这个变量会作用到由这个目标所引发的所有的规则中去
	prog : CFLAGS = -g 
	prog : prog.o foo.o bar.o 
	$(CC) $(CFLAGS) prog.o foo.o bar.o
	，不管全局的$(CFLAGS)的值是什么，在 prog 目标，以及其所引发的所有规则中（prog.o foo.o bar.o 的规则），$(CFLAGS)的值都是“-g”

## Make 条件判断  

“$@”表示目标的集合  
“$<”表示依赖的文件  
“$^”  
“$$$$”意为一个随机编号  