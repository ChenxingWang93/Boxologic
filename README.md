# Boxologic
A set of C programs that calculate the best fit for boxes on a pallet, and visualize the result.  //

# Data Structure 数据结构
数据结构会影响程序性能和求解时间。对于程序来说，为快速获取数据，使用了2个不同的 ***array*** & 1个 ***double linked list*** 来完成任务


第一个是：Boxlist[]array, 保存所有打包盒的尺寸dimensions、坐标coordinates 以及其他必要数据在容器中，在这个 array 中有总计 ***12个字段***
|Element|Name|Description|
|-------|----|-----------|
|1.     |Packst|:Status of packing (0: Not packed; 1: Packed), 打包和没打包两种状态，|
|2.     |N     |:The number of boxes that have the same dimensions, 有相同尺寸的盒子的数量，|
|3.     |Dim1  |:The length of one of the three dimensions, 三个维度中的 一种 “长度”，|
|4.     |Dim2  |:The length of another ofthe three dimensions, 三个维度中的 另一种 “长度”，|
|5.     |Dim3  |:The length of the other of the three dimensions, 三个维度中的 其他的 “长度”，|
|6.     |Cox   |:X-coordinate of the location of the packed box, 打包了的盒子的 X-coordinate, 坐标，|
|7.     |Coy   |:Y-coordinate of the location of the packed box, 打包了的盒子的 Y-coordinate, 坐标，|
|8.     |Coz   |:Z-coordinate of the location of the packed box, 打包了的盒子的 Z-coordinate, 坐标，|
|9.     |Packx |:X-dimension of the orientation of the box as it has been packed, 打包了的盒子的 X-dimension，|
|10.    |Packy |:Y-dimension of the orientation of the box as it has been packed, 打包了的盒子的 Y-dimension，|
|11.    |Packz |:Z-dimension of the orientation of the box as it has been packed, 打包了的盒子的 Z-dimension，|
|12.    |Vol   |Volume of the box (Diml*Dim2*Dim3), 盒子的体积，|

- 一旦打包状态为0: Not packed; 则不会计算盒子的体积
- 一旦打包状态为1: Packed; 每个盒子会记录在 Boxlist[] array 中

|Element|Name|Description|
|-------|----|-----------|
|1.     |Layerdim|:A dimension value, 一个三维值|
|2.     |Layereval|:Evaluation weight value for the corresponding layerdim value, 给对应的 Layerdim 值 评估 重量值|

我们使用的双向链表 _**double linked list**_ 保持当前边缘层的拓扑结构在 建设中，我们保留每个gap 右下角的x &z 坐标，程序会看到这些gaps 并试图用盒子来一次性填补，并保持层的边缘均匀，每个双向链表 _**double linked list**_ 中的entry 都有这些 data field

|Element|Name|Description|
|-------|----|-----------|
|1.     |*pre|: Pointer that keeps the address of the previous entry, 用指针来保留之前入口的地址，|
|2.     |Cumx|: Keeps the x-coordinate of the gap's right corner, 保持缝隙的x-coordinate处在右上角，|
|3.     |Cumz|: Keeps the z-coordinate of the gap's right corner, 保持缝隙的x-coordinate处在右上角，|
|4.     |*pos|: Pointer that keeps the address of the following entry. 用指针来保留随后入口的地址，|


### 数值限制 Numerical Limits
数据结构模型 model 的限制通过考虑内存限制 memory limitations 平均计算内存容量 average computer memory capacity 以及现实生产中的打包问题 realistic packing problems:
- maximum number of boxes in a box set 在盒子集中 的盒子最大数量 
- number of total different dimension values 不同三维值 的总计数量
- max dimension length for either pallet or any box 托盘 或者盒子的 最大三维“长度”

### 算法流程图 Flow Chart of the Algorithm

<img width="800" alt="算法流程图1" src="https://user-images.githubusercontent.com/31954987/212693108-75f6604e-9111-41a9-8f90-50e4fb0e4bb2.png">

<img width="800" alt="算法流程图2" src="https://user-images.githubusercontent.com/31954987/212938794-480c4340-d396-4bfc-970a-a54abcaeea76.png">

### 算法何谓启发式 How Does The Heuristic Work?
<img width="800" alt="image" src="https://user-images.githubusercontent.com/31954987/213875993-3a68e536-2fb1-416f-a7a4-31d07d9d7c6a.png">

> ``` C
> Variable assignments:                                                                          XX=104;YY=96;ZZ=84;
> Boxlist[X]=(Packst,   N,    Dim1,   Dim2,   Dim3,   Cox,    Coy,    Coz,    Packx,    Packy,    Packz,        Vol)
> Boxlist[1]=(     0,   2,   260.7,    897,    1.5,     0,      0,      0,        0,        0,        0,  350771.85)
> Boxlist[2]=(     0,   2,   260.7,    897,    1.5,     0,      0,      0,        0,        0,        0,  350771.85)
> Boxlist[3]=(     0,   2,   260.7, 1399.5,    1.5,     0,      0,      0,        0,        0,        0, 547274.475)
> Boxlist[4]=(     0,   2,   260.7, 1399.5,    1.5,     0,      0,      0,        0,        0,        0, 547274.475)
> Boxlist[5]=(     0,   1,   260.7,  920.7,    1.5,     0,      0,      0,        0,        0,        0, 360039.735)
> Boxlist[6]=(     0,   1,     910,   2700,    1.5,     0,      0,      0,        0,        0,        0,    3685500)
> Boxlist[7]=(     0,   1,     910,   2460,    1.5,     0,      0,      0,        0,        0,        0,    3357900)
> Boxlist[8]=(     0,   1,      60,    595,    1.5,     0,      0,      0,        0,        0,        0,      53550)
> ```

核心的打包程序函数
|FUNCTION 函数|PURPOSE 目标|
|------------|------------|
|Packlayer() |随着盒子被打包则更新linked list &Boxlist[] array|
|Findsmallestz()|用smallest z 值来决定当前层盒子的 间隙        |
|Findbox()   |找到最好 fit 当前间隙的 盒子                    |
|Analyzebox()|被Findbox()用来 分析 盒子的三维                 |
|Checkfound()|决定pack 哪一个盒子                            |
|Execiterations()|call合适的 函数 来执行 迭代                 |
|Report()    |复制best-so-far packing                      |
|Outputboxlist()|Writes 打包信息到文件                       |
|Graphunpackedout()|Writes 打包顺序                         |

Findbox 函数 使用 Analyzebox 函数 分析未打包的盒子。每个不同朝向 的未打包盒子，Analyzebox 函数参数都有:
> ``` C
> Hmx   :Maximum available X-dimension of the current gap to be filled.   //当前 要填补缝隙中 **最大**可用 X-dimension
> Hy    :Current layer thickness value.                                   //**当前**层厚值 (y轴值)
> Hmy   :Maximum available Y-dimension of the current gap to be filled.   //当前 要填补缝隙中 **最大**可用 Y-dimension
> Hz    :Z-dimension of the current gap to be filled.                     //要被填充的 **当前**层厚（z轴值）
> Hmz   :Maximum available z-dimension to the current gap to be filled.   //当前 要填补缝隙中 **最大**可用 Z-dimension
> Dim1  :X-dimension of the orientation of the box being examined         //盒子 朝向 的X-dimension
> Dim2  :Y-dimension of the orientation of the box being examined         //盒子 朝向 的Y-dimension
> Dim3  :Z-dimension of the orientation of the box being examined         //盒子 朝向 的Z-dimension
> ```
<img width="800" alt="image01" src="https://user-images.githubusercontent.com/31954987/213901830-7b9a211b-c72f-4c6b-ae07-cc522d150aa9.png">


### pseudo-codes of the functions函数伪代码
Appendix A - Pseudo-codes of the Functions 函数的伪代码

> ``` C
> MAIN
> /*初始化变量*/
> FUNCTION INITIALIZE();
> 
> /*从输入文件中读取数据 的函数*/
> FUNCTION INPUTBOXLIST();
> 
> /*创建 候选Layers[]阵列 的函数*/
> FUNCTION LISTCANDITLAYERS();
> 
> /*计算层列表 的函数*/
> FUNCTION COMPLAYERLIST();
> 
> /*PACKLAYER层厚 的函数*/
> FUNCTION PACKLAYER();
> 
> /*通过检查未打包的盒子找到最合适的层厚值 的函数*/
> FUNCTION FINDLAYER();
> 
> /*找到最好fit当前间隙的盒子 的函数*/
> FUNCTION FINDBOX();
> 
> /*分析盒子三维 ANALYZEBOX 的函数*/
> FUNCTION ANALYZEBOX(HMX,HY,HMY,HZ,HMZ,DIM1,DIM2,DIM3);
> 
> /*用smallest z 值来决定当前层的 盒子间隙 的函数*/
> FUNCTION FINDSMALLEST();
> 
> /*决定pack 哪一个盒子 的函数*/
> FUNCTION CHECKFOUND();
> 
> /*检查盒子体积 的函数*/
> FUNCTION VOLUMECHECK();
> 
> /**/
> OUTPUTBOXLIST();
> 
> /*Writes 打包信息到文件 的函数*/
> FUNCTION REPORT();
> ```
#

/*MAIN 函数*/
> ``` C
> MAIN
> 
> /*通过call INITIALIZE() 函数 执行初始化*/
> Perform initialization by calling INITIALIZE();
> 
> /**/
> Get time(START);
> 
> /*通过call EXECITERATIONS() 执行迭代 &找到最优解的参数*/
> Execute iterations and find the parameters of the best solution by calling 
>   EXECITERATIONS();
>   
> /**/
> Get time(FINISH);
> 
> /*使用找到的参数，打包找到的最优解，报告控制器*/
> Using the parameters found, pack the best solution found, report to the console
>   and to an output text file by calling REPORT();
> Wait until a keystroke entered by the user;
> End;
> ```
#

/*INITIALIZE() 初始化 函数*/
> ``` C
> FUNCTION INITIALIZE()
> /*抓取文件名*/
> Get the input FILENAME from the user; 
> 
> /*call 函数INPUTBOXLIS() 得到来自用户输入文件的 pallet &box 集数据*/
> Get the pallet and box set data entered by the user from the input file by calling the function INPUTBOXLIS();
> calling the function INPUTBOXLIST();  
> 
> /*计算托盘体积*/
> Calculate the volume of the pallet; 
> 
> /*计算盒子总体积*/
> Calculate the total volume of all boxes;
> 
> /*创建一个节点& call 它废料优先。每一个双向链接列表 节点保持 每个当前正在打包的层 的缝隙的 X & Z坐标*/
> Create a node and call it SCRABFIRST. Each of these double linked list nodes keeps X and Z coordinates of each gap in the layer currently being packed.
> 
> /*废料优先.PRE; 废料优先.POS*/
> SCRABFIRST.PRE=NULL;SCRABFIRST.POS=NULL;
> 
> /*初始化目前保持最好的变量 &参数*/
> Initialize variables those keep the best so far and its parameters.
> ```
#

/*INPUTBOXLIST() 从输入文件中读取盒列表数据 函数*/
> ``` C
> /*INPUTBOXLIST() 函数*/
> ```
#

/*EXECITERATIONS() 执行迭代 函数*/
> ``` C
> 
> ```
#

/*LISTCANDITLAYERS() 罗列所有可能的候选值 函数*/
> ``` C
> 
> ``` 
#

/*COMPLAYERLIST() 对比列表 函数 com means compare*/
> ``` C
> FUNCTION COMPLAYERLIST(i,j);
> /*COMPLAYERLIST(i,j) 函数要求编译器内建函数QSORT()*/
> This function is required for the compiler built in function QSORT().
> /*回传 i &j之间的差值*/
> It returns the difference between the values i & j.
> ```
#

/*PACKLAYER()打包层 函数*/
> ``` C
> 
> ```
#

/*FINDLAYER() 通过检查未打包的盒子找到最合适的层厚值 函数*/
> ``` C
> 
> ```
#

/*FINDBOX() 找到最好fit当前缝隙的盒子 函数*/
> ``` C
> 
> ```
#

/*ANALYZEBOX(HMX, HY, HMY, HZ, HMZ, DIM1, DIM2, DIM3) 函数*/
> ``` C
> 
> ```
#

/*FINDSMALLESTZ() 用smallest z 值决定当前层 盒子间隙 函数*/
> ``` C
> 
> ```
#

/*CHECKFOUND() 决定pack 哪个盒子 函数*/
> ``` C
> 
> ```
#

/*VOLUMECHECK() 检查打包情况 函数*/
> ``` C
> 
> ```
#

/*GRAPHUNPACKEDOUT() 检查打包情况 函数*/
> ``` C
> 
> ```
#

/*OUTPUTBOXLIST() 写入打包信息到文件 函数*/
> ``` C
> 
> ```
#

/*REPORT() 函数*/
> ``` C
> 
> ```
#

``` C
Layers[X]=(Layerdim, Layereval)
Layers[l]=(70, 98)
Layers[2]=(24, 56)
Layers[3]=(14,106)
Layers[4]=(48,100)
Layers[5]=(40, 80)
Layers[6]=(52, 80)
Layers[7]=(36, 72)
```

# Future Plans
This project uses a fairly old codebase as a launching-off point. The plan is to modernize it a bit, and then add functionality by allowing the user to specify more than one container to be packed, as well as perhaps libraryifying the main code so that you can wrap your own code around it more easily. Keep watching this space for details as they emerge.
//本项目采用足够老的 codebase 作为发起点。目前的计划是让它变得更 摩登 一些，后续添加功能允许多于一个 容器被打包，包括或许libraryifying 主要代码 这样就能，关注更多细节。

# Caveat Emptor  //买主警告⚠️
This codebase is currently undergoing fairly sweeping changes. While we'll try our best to always have compile-ready, working code here, no guarantees of any sort are made. Currently the visualizer only runs using the windows-only code& binary supplied by [the project this one was forked from](https://github.com/ChenxingWang93/3d-bin-pack). in this fork is slated to be rewritten using OpenGL, and will be cross-platform.  //被预定 用OpenGL 重写

# Usage //使用
> ``` C
> USAGE:
>   boxologic <option>
>  
> OPTIONS:
>   [ -f|--inputfile  ] <boxlist text file>   : Perform bin packing analysis
>   [ -v]--version  ]                         : Print software version
>   [ -h]--help ]                             : Print the help screen
> ```

# Boxlist Format //盒子列表 形式
The first line of the boxlist file defines the X, Y, and Z dimensions of the pallet in which all the boxes are to be packed. Each successive line begins with a box number, then has the x, y and z dimensions of the box, and finally how many of this box are to be packed onto a pallet. A series of sample boxlist was [provided in the doc directory](). // 提供一系列测试用的 boxlist 

and *** a REPORT ***
> ``` C
> ---------------------------------------------------------------------------------------------
>                                         *** REPORT ***  
> ---------------------------------------------------------------------------------------------
> ELAPSED TIME //花费时间                                   : **X** sec
> TOTAL NUMBER OF ITERATIONS DONE //总计完成的迭代次数        : **X**
> BEST SOLUTION FOUND AT ITERATION //迭代过程中找到的最优解   : **X** OF VARIANT: **X**
> TOTAL NUMBER OF BOXES //盒子数量                          : **X**
> PACKED NUMBER OF BOXES //打包起的盒子数量                  : **X**
> TOTAL VOLUME OF ALL BOXES //所有盒子的体积                 : **X**
> PALLET VOLUME //托盘的体积                                : **X**
> BEST SOLUTION'S VOLUME UTILIZATION //体积利用率           : **X** OUT OF **X**
> PERCENTAGE OF PALLET VOLUME USED //托盘体积使用率          : 100.000000 %
> PERCENTAGE OF PACKED BOXES (VOLUME) //打包盒（体积）比例    : 100.000000 %
> WHILE PALLET ORIENTATION //托盘朝向                       : X=; Y=; Z=
> ---------------------------------------------------------------------------------------------
>   NO: PACKSTA DIMEN-1  DMEN-2  DIMEN-3   COOR-X   COOR-Y   COOR-Z   PACKEDX  PACKEDY  PACKEDZ
> ---------------------------------------------------------------------------------------------
> *** LIST OF UNPACKED BOXES ***
> ```  
