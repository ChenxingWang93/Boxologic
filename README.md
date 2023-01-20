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


### pseudo-codes of the functions函数伪代码

<img width="800" alt="算法流程图1" src="https://user-images.githubusercontent.com/31954987/212693108-75f6604e-9111-41a9-8f90-50e4fb0e4bb2.png">

<img width="800" alt="算法流程图2" src="https://user-images.githubusercontent.com/31954987/212938794-480c4340-d396-4bfc-970a-a54abcaeea76.png">

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
