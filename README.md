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

> ``` C
> /**/
> Layers[X]=(Layerdim, Layereval)
> Abs(70-70)+Abs(70-70)+Abs(70-70)+Abs(70-48)+Abs(70-48)+Abs(70-52)+Abs(70-52)+Abs(70-52)=98
> 
> Layers[1]=(70, 98)
> Abs(24-24)+Abs(24-24)+Abs(24-24)+Abs(24-14)+Abs(24-14)+Abs(24-36)+Abs(24-36)+Abs(24-36)=56
> 
> Layers[2]=(24, 56)
> Abs(14-24)+Abs(14-24)+Abs(14-24)+Abs(14-24)+Abs(14-14)+Abs(14-40)+Abs(14-40)+Abs(14-40)=106
> 
> Layers[3]=(14, 106)
> Abs(48-70)+Abs(4-70)+Abs(48-70)+Abs(48-70)+Abs(48-48)+Abs(48-40)+Abs(48-40)+Abs(48-40)=100
> 
> Layers[4]=(48, 100)
> Abs(40-24)+Abs(40-24)+Abs(40-24)+Abs(40-24)+Abs(40-48)+Abs(40-48)+Abs(40-40)+Abs(40-40)=80
> 
> Layers[5]=(40, 80)
> Abs(52-70)+Abs(52-70)+Abs(52-70)+Abs(52-70)+Abs(52-48)+Abs(52-48)+Abs(52-52)+Abs(52-52)=80
> 
> Layers[6]=(52, 80)
> Abs(36-24)+Abs(36-24)+Abs(36-24)+Abs(36-24)+Abs(36-48)+Abs(36-48)+Abs(36-36)+Abs(36-36)=72
> 
> Layers[7]=(36, 72)
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
> /*得到 时间（开始）*/
> Get time(START);
> 
> /*通过call EXECITERATIONS() 执行迭代 &找到最优解的参数*/
> Execute iterations and find the parameters of the best solution by calling 
>   EXECITERATIONS();
>   
> /*得到 时间（结束）*/
> Get time(FINISH);
> 
> /*使用找到的参数，打包找到的最优解，报告控制器*/
> Using the parameters found, pack the best solution found, report to the console and to an output text file by calling REPORT();
> 
> /*等待用户的键入*/
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
> FUCTION INPUTBOXLIST()
> 
> /*如果存在，打开文件*/
> If exists, open the file FILENAME;
> 
> /*否则，告诉用户“无法打开 文件 XXX”*/
> Else{Tell the user "Cannot open the file FILENAME";end;}
>   
> /*读取输入文件的第一行 &设定托盘的三维变量 XX, YY, ZZ;*/
> Read the first line of the input file and set the pallet dimension variables XX, YY, ZZ;
> 
> /*读取输入文件中的每个其他行 &对应到阵列BOXLIST[]中的每个域。*/
> Read every other lines in the input file and fill each field in the array BOXLIST[].
> 
> /*变量TBN 已设定到 输入文件中 盒子的总数量啊！！TBN是the abbreviation of Total Box Number, 即TBN*/
> Now the variable TBN is already set to the total number of boxes input from the file;
> 
> /*关闭文件 FILENAME*/
> Close the file FILENAME;
> RETURN;
> ```
#


/*EXECITERATIONS() 执行迭代 函数*/
> ``` C
> /*call 合适的函数来执行迭代*/
> FUNCTION EXECITERATIONS();
> 
> /*为什么 会是 6个变体VARIANT ???*/
> For VARIANT=1 to 6 {
>   /*VARIANT的每个值 得到托盘的不同朝向 啊！PX is the abbreviation of PalletX, PY,PZ同理*/
>   For each value of VARIANT get a different orientation of the pallet to the variable PX,PY,PZ;
>   
>   /*通过call LISTCANDITLAYERS() 罗列所有可能的候选值;*/
>   List all possible candidate values by calling LISTCANDITLAYERS();
>   
>   /*通过使用 QSORT 以递增顺序相对于 LAYEREVAL 的域 分类 阵列 LAYERS*/
>   Sort the array LAYERS in respect to its LAYEREVAL fields in increasing order by using QSORT;
>   
>   /*LAYERS[] 阵列中的每一个 层值，执行另一个迭代开始于 层值 作为开始层 的厚度*/
>   For each layer values in the LAYERS[] array, perform another iteration staring with that layer value as the starting layer thickness:
>   
>   /*LAYERLIST 变量为0，指针LAYERSINDEX为1，LAYERS[]阵列中的第一个值为开始*/
>   For LAYERSINDEX=1 to LAYERLISTLEN{
>   
>       /*LAYERS[] 阵列第一个值作为开始*/
>       Get the first value of the LAYERS[] array as the starting
>       
>           /*层厚 值*/
>           LAYERTHICKNESS value:
>           
>           /*层厚 值 = 层[层索引].层三维*/
>           LAYERTHICKNESS=LAYERS[LAYERSINDEX].LAYERDIM
>           
>       /*把所有盒子打包状态 设定为 0*/
>       Set all boxes' packed status to 0:
>       
>           /*从[1]到[TBN] BOXLIST[X].PACKST=0,设定为未打包*/
>           For X=1 to TBN do BOXLIST[X].PACKST=0;
>       do {
>           Set the variable that shows remaining unpacked potential second layer height in the current layer:
>             
>           /*设定 展示剩余未打包的 可能的第二层高度 到当前的层 的变量 LAYERINLAYER*/
>           LAYERINLAYER=0;
>
>           /*设定🚩变量 展示 当前层的打包是否完成 LAYERDONE=0;*/
>           Set the flag variable that shows packing of the current layer is finished or not: LAYERDONE=0;
>
>           /*call PACKLAYER() 函数，打包层，如果响应了内存错误，则退出程序；*/
>           Call PACKLAYER(), to pack the layer, and if a memory error is responded, exit the program;
>           
>           /*如果在当前层中有一个高度可用来打包，在当前层执行另一个层打包*/
>           If LAYERINLAYER≠ 0 do {
>               /*获取当前层可打包的高度，并作为待打包的高度*/
>               Get the height available for packing in the current layers as the layer thickness to be packed:
>               LAYERTHICKNESS=LAYERINLAYER;
>               
>               /*call PACKLAYER() 函数打包层，如果内存错误响应，则退出程序*/
>               Call PACKLAYER(),to pack the layer, and if a memory error is responded, exit the program;
>           }
>           /*决定最合适的层高 适配到托盘 剩余未打包高度*/
>           Call FINDLAYER(REMAINPY) to determine the most suitable layer height fitting in the remaining unpacked height of the pallet；
>       }while PACKING≠ 0;
>       /*如果迭代的体积利用率优于当前最优值 &不会退出迭代，保留参数：(托盘朝向，利用率，指针 指向在LAYERS[]阵列中 最初层高)*/
>       If the volume utilization of the current iteration is better than the best so far, and the iterations were not quit, keep the parameters:(Pallet orientation,utilization,and the index of the initial layer height in the LAYERS array);
>       /*如果100%打包完成，退出迭代并返回*/
>       If a hundred percent packing was found, exit doing iteration and RETURN;
>   }
> }
> ```
#


/*LISTCANDITLAYERS() 罗列所有可能的候选值 函数*/
> ``` C
> FUNCTION LISTCANDITLAYERS();
> 
> /*变量 LAYERLISTLEN 里没有对象*/
> LAYERLISTLEN=0;
> 
> /**/
> For X=1 to TBN {
> 
>   /*得到每个盒子的三维，一次一个*/
>   Get each dimension of each box, one at a time by doing:
>   For Y=1 to 3 {
>       If Y=1 do {
>           EXDIM=BOXLIST[X].DIM1;
>           DIMEN2=BOXLIST[X].DIM2;
>           DIMEN3=BOXLIST[X].DIM3;
>       }
>       If Y=2 do {
>           EXDIM=BOXLIST[X].DIM2;
>           EXDIM2=BOXLIST[X].DIM1;
>           EXDIM3=BOXLIST[X].DIM3;
>       }
>       If Y=3 do {
>           EXDIM=BOXLIST[X].DIM3;
>           EXDIM2=BOXLIST[X].DIM1;
>           EXDIM3=BOXLIST[X].DIM2;
>       }
>       
>       /*如果被检查盒子的任意维度不能 fit 托盘的对应 维度，退出循环& 继续下一个循环*/
>       If any of the dimensions of the box being examined cannot fit into the 
>               pallet's respective dimensions, exit this loop and continue with the 
>               next loop;
>       
>       /*如果EXDIM与之前检验 三维 长度相同，退出循环 &继续下一个循环*/
>       If EXDIM is the same as any of previously examined dimension lengths, 
>               exit this loop and continue with the next loop;
>       
>       /*设定EXDIM 的评估值evaluation value 为 0*/
>       Set the evaluation value of the EXDIM to 0 by doing LAYEREVAL=0;
>       
>       /*从 Z=1 到 TBN 盒子的总计数量*/
>       For Z=1 to TBN do {
>       
>           /*取得每个盒子最近的维度值到EXDIM*/ 
>           Get the closest dimension value of each box to the EXDIM
>           
>               /*查看绝对值（每个维度值与EXDIM的差值），选择最小的值& 设定变量DIMDIF来存储这个值*/
>               by looking at the absolute values of differences between each dimension and EXDIM, and selecting the smallest value;
>               and set the variable DIMDIF to this value;
>               
>           /*累加这些值*/
>           Add those values cumulatively by doing:
>               /**/
>               LAYEREVAL=LAYEREVAL+DIMDIF;
>       }
>       /*LAYERLIST阵列长度 + 1；*/
>       LAYERLISTLEN=LAYERLISTLEN+1;
>   
>       /*层[层列表长度].层评估值=LAYEREVAL；*/
>       LAYERS[LAYERLISTLEN].LAYEREVAL=LAYEREVAL;
>   
>       /*层[层列表长度].层维度=EXDIM；*/
>       LAYERS[LAYERLISTLEN].LAYERDIM=EXDIM;
>     }
> }
> RETURN;
> ``` 
#


/*COMPLAYERLIST() 对比列表 函数 com means compare*/
> ``` C
> FUNCTION COMPLAYERLIST(i,j);
> 
> /*COMPLAYERLIST(i,j) 函数要求编译器内建函数QSORT()*/
> This function is required for the compiler built in function QSORT().
> 
> /*回传 i &j之间的差值*/
> It returns the difference between the values i & j.
> ```
#


/*PACKLAYER()打包层 函数*/
> ``` C
> /**/
> If LAYERTHICKNESS=0 do {PACKING=0;RETURN;};
> 
> /*初始化 第一个& 唯一一个到层's X &Z 值的节点 */
> Initialize the first &only node to the layer's X and Z values:
> 
>     /*废料优先.*/
>     SCRAPFIRST.CUMX=PX;SCRAPFIRST.CUMZ=0;
> Perform an infinite loop unless 'Q' is typed to quit{
> 
>     /**/
>     Check the keyboard input, if 'Q' is hit, exit the loop and RETURN;
>     
>     /**/
>     To find the gap with the least Z value in the layer call FINDSMALLESTZ();
>     
>     
>     /*情况-1*/
>     SITUATION-1:
>     /*如果缝隙 的两侧都没有盒子 则do*/
>     if there is no box on the both sides of the gap do {
>     
>         /*计算缝隙的 X &Z 维值*/
>         calculate the gap's X &Z dimensions;
>         
>         /*通过查看以下参数为缝隙找到最合适的盒子*/
>         to find the most suitable boxes to the gap found, by looking at;
>         
>             /*缝隙的x 维度：LENX*/
>             the X-dimension of the gap: LENX,
>             
>             /*缝隙的层厚：LAYERTHICKNESS*/
>             layerthickness of the gap: LAYERTHICKNESS,
>             
>             /*距离缝隙的最大可用厚度：REMAINPY*/
>             maximum available thickness to the gap: REMAINPY,
>             
>             /*距离缝隙的最大可用z 轴：LPZ*/
>             maximum available Z dimension to the gap: LPZ;
>             
>             /*call FINDBOX(LENX,LAYERTHICKNESS,REMAINPY,LPZ,LPZ)函数*/
>             call FINDBOX(LENX,LAYERTHICKNESS,REMAINPY,LPZ,LPZ);
>             
>             /*通过call CHECKFOUND() 函数 检查由FINDBOX() 函数 找到的盒子*/
>             Check on the boxes found by the FINDBOX() function by calling CHECKFOUND();
>             
>             /*如果层的打包完成，则退出循环*/
>             If the packing of the layer is finished, exit the loop;
>             
>             /*如果层的边缘是被拉平的，则到下一个循环的 第一行*/
>             If the edge of the layer is evened, go to the first line of the next loop;
>             
>             /*在链列表linked list中添加 一个新节点 显示正在被打包层的边缘的拓扑结构*/
>             Add a new node to the linked list showing the topology of the edge of the currently being packed layer after packing a new box, and set all the necessary variables and pointers properly;
>             To check the hundred percent packing condition,
>             call VOLUMECHECK();
>     }
>     
>     
>     /*情况-2*/
>     SITUATION-2:
>     /*如果缝隙的左侧 没有盒子*/
>     
>     If there is no box on the right side of the gap do {
>         /*计算缝隙的 X &Z 维值*/
>         Calculate the gap's X & Z dimensions;
>         
>         /*通过查看以下参数为缝隙找到最合适的盒子*/
>         To find the most suitable boxes to the gap found, by looking at;
>         
>             /*缝隙的x 的方向长度：LENX*/
>             the X-dimension of the gap: LENX,
>             
>             /*缝隙的层厚：LAYERTHICKNESS*/
>             layerthickness of the gap: LAYERTHICKNESS,
>             
>             /*距离缝隙的最大可用厚度：REMAINPY*/
>             maximum available thickness to the gap: REMAINPY,
>             
>             /*缝隙的z 方向长度：LENX*/
>             the Z dimension of the gap: LENZ,
>             
>             /*距离缝隙的最大可用z 轴值：LPZ*/
>             maximum available Z dimension to the gap: LPZ;
>             
>             /*call FINDBOX(LENX, LAYERTHICKNESS, REMAINPY, LENZ, LPZ) 函数 找到盒子*/
>             call FINDBOX(LENX, LAYERTHICKNESS, REMAINPY, LENZ, LPZ);
>             
>             /*通过call CHECKFOUND() 函数 检查由FINDBOX() 函数 找到的盒子*/
>             Check on the boxes found by the FINDBOX() function by calling CHECKFOUND();
>             
>             /*如果打包层结束，退出循环*/
>             If the packing of the layer is finished, exit the loop;
>             
>             /*如果层的边缘是被拉平的，则到下一个循环的 第一行*/
>             If the edge of the layer is evened, go to the first line of the next loop;
>
>             /*适当地设定所有必要的变量和指针 代表当前正在打包 层的边缘拓扑结构;*/
>             Set all the necessary variables and pointers properly to represent the current topology of the edge of the layer that is currently being packed;
>             
>             /*如果当前层的边缘 是平的，适当地设定所有必要的变量 &指针 处理不必要的节点；*/
>             If the edge of the current layer is evened, set all the necessary variables and pointers properly and dispose the unnecessary node;
>             
>             /*通过call VOLUMECHECK() 检查打包情况*/
>             To check the hundred percent packing condition, call VOLUMECHECK()
>     }
>     
>       
>     /*情况-3*/
>     SITUATION-3:
>     /*如果缝隙的右侧 没有盒子*/
>     If there is no box on the right side of the gap do {
>         /*计算缝隙的 x & z 维值*/
>         Calculate the gap's X & Z dimensions;
>
>         /*通过查看以下参数为缝隙找到最合适的盒子*/
>         To find the most suitable boxes to the gap found, by looking at;
>         
>         /*缝隙的x 的方向长度：LENX*/
>         the X-dimension of the gap: LENX,
>         
>         /*缝隙的层厚：LAYERTHICKNESS*/
>         layerthickness of the gap: LAYERTHICKNESS,
>         
>         /*距离缝隙的最大可用厚度：REMAINPY*/
>         maximum available thickness to the gap: REMAINPY,
>         
>         /*缝隙的z 方向长度：LENX*/
>         the Z dimension of the gap: LENZ,
>         
>         /*距离缝隙的最大可用z 轴值：LPZ*/
>         maximum available Z dimension to the gap: LPZ;
>         
>         /*call FINDBOX(LENX, LAYERTHICKNESS, REMAINPY, LENZ, LPZ) 函数 找到盒子*/
>         call FINDBOX(LENX, LAYERTHICKNESS, REMAINPY, LENZ, LPZ);
>         
>         /*通过call CHECKFOUND() 函数 检查由FINDBOX() 函数 找到的盒子*/
>         Check on the boxes found by the FINDBOX() function by calling CHECKFOUND();
>         
>         /*如果打包层结束，退出循环*/
>         If the packing of the layer is finished, exit the loop;
>         
>         /*适当地设定所有必要的变量和指针 代表当前正在打包 层的边缘拓扑结构;*/
>         Set all the necessary variables and pointers properly to represent the current topology of the edge of the layer that is currently being packed;
>         
>         /*如果层的边缘是被拉平的，则到下一个循环的 第一行*/
>         If the edge of the layer is evened, go to first line of the next loop;
>         
>         /*通过call VOLUMECHECK() 检查打包情况*/
>         To check the hundred percent packing condition, call VOLUMECHECK()
>     }
>     
>       
>     /*情况-4*/
>     /*缝隙两侧都有盒子*/
>     SITUATION-4: IF THERE ARE BOXES ON BOTH SIDES OF THE GAP
>     /*分 情况-4A*/
>     SUBSITUATION-4A
>     /*如果 缝隙的z轴方向 与两侧 相同*/
>     If the Z dimensions of the gap is the same on both sides {
>         /*计算缝隙的 x & z 方向值*/
>         Calculate the gap's X & Z dimensions;
>         
>         /*通过查看以下参数为缝隙找到最合适的盒子*/
>         To find the most suitable boxes to the gap found, by looking at;
>             /*缝隙的x 的方向长度：LENX*/
>             the X-dimension of the gap: LENX,
>
>             /*缝隙的层厚：LAYERTHICKNESS*/
>             layerthickness of the gap: LAYERTHICKNESS,
>             
>             /*距离缝隙的最大可用厚度：REMAINPY*/
>             maximum available thickness to the gap: REMAINPY,
>             
>             /*缝隙的z 方向长度：LENX*/
>             the Z dimension of the gap: LENZ,
>             
>             /*距离缝隙的最大可用z 轴值：LPZ*/
>             maximum available Z dimension to the gap: LPZ; 
>             
>             /*call FINDBOX(LENX, LAYERTHICKNESS, REMAINPY, LENZ, LPZ) 的函数 找到盒子*/
>             call FINDBOX(LENX, LAYERTHICKNESS, REMAINPY, LENZ, LPZ);
>             
>             /*通过call CHECKFOUND() 函数 检查由FINDBOX() 函数 找到的盒子*/
>             Check on the boxes found by the FINDBOX() function by calling CHECKFOUND(); 
>     }
>     
>       
>     /*分 情况-4B*/
>     /*如果 缝隙的z轴方向 与两侧 不同*/
>     If the Z dimension of the gap is different on both sides {
>         /*计算缝隙的 x & z 维值*/
>         Calculate the gap's X and Z dimensions;
>         
>         /*通过查看以下参数为缝隙找到最合适的盒子*/
>         To find the most suitable boxes to the gap found, by looking at;
>             /*缝隙的x 的方向长度：LENX*/
>             the X-dimension of the gap: LENX,
>
>             /*缝隙的层厚：LAYERTHICKNESS*/
>             layerthickness of the gap: LAYERTHICKNESS,
>             
>             /*距离缝隙的最大可用厚度：REMAINPY*/
>             maximum available thickness to the gap: REMAINPY,
>             
>             /*缝隙的z 方向长度：LENX*/
>             the Z dimension of the gap: LENZ,
>             
>             /*call FINDBOX(LENX, LAYERTHICKNESS, REMAINPY, LENZ, LPZ) 的函数 找到盒子*/
>             call FINDBOX(LENX, LAYERTHICKNESS, REMAINPY, LENZ, LPZ);
>             
>             /*通过call CHECKFOUND() 函数 检查由FINDBOX() 函数 找到的盒子*/
>             Check on the boxes found by the FINDBOX() function by calling CHECKFOUND();
>             
>             /*如果层的打包任务结束，则退出循环*/
>             If the packing of the layer is finished, exit the loop;
>             
>             /*如果层的边缘是被拉平的，则到下一个循环的 第一行*/
>             If the edge of the layer is evened,go to the first line of the next loop;
>             
>             /*适当地设定所有必要的变量和指针 代表当前正在打包 层的边缘拓扑结构;*/
>             Set all the necessary variables and pointers properly to represent the current topology of the edge of the layer that is currently being packed; 
>             
>             /*更新拓扑的边缘信息，如果另一个缝隙被添加到拓扑，则添加一个节点来保留这个缝隙，适当地更新其他*/
>             While updating the edge of topology information, if another gap is added to the topology, add a new node to keep this gap, and update the others properly;
>             
>             /*通过call VOLUMECHECK() 检查打包情况*/
>             To check the hundred percent packing condition, call VOLUMECHECK()
>     }
> }
>     
> ```
#


/*FINDLAYER() 通过检查未打包的盒子找到最合适的层厚值 函数*/
> ``` C
> Function FINDLAYER();
> /*设定 总评价值 到一个big number*/
> 
> Set the overall evaluation value to a big number:EVAL=1000000;
> /*从 X=1 到 TBN*/
> For X=1 to TBN {
> 
> /*如果 盒子数量 X 被打包 继续下一个loop*/
> If the box number X has already been packed continue with the next loop:
> 
>     /*如果 BOXLIST[X].PACKST≠0 (盒子打包状态是未打包) 继续*/
>     If BOXLIST[X].PACKST≠0 continue；
>     
> /*一次获得 一个 盒子的一个维度*/
> Get each dimension of each box, one at a time by doing:
> For Y=1 to 3 {
>     If Y=1 do {
>         EXDIM=BOXLIST[X].DIM1;
>         DIMEN2=BOXLIST[X].DIM2;
>         DIMEN3=BOXLIST[X].DIM3;
>     }
>     If Y=2 do {
>         EXDIM=BOXLIST[X].DIM2;
>         DIMEN2=BOXLIST[X].DIM1;
>         DIMEN3=BOXLIST[X].DIM3;
>     }
>     If Y=3 do {
>         EXDIM=BOXLIST[X].DIM3;
>         DIMEN2=BOXLIST[X].DIM1;
>         DIMEN3=BOXLIST[X].DIM2;
>     }
>     
>     /*如果被检查盒子的任意维度不能 fit 托盘的对应 维度，退出循环& 继续下一个循环*/
>     If any of the dimensions of the box being examined cannot fit into the pallet's respective dimensions, exit this loop and continue with the next loop;
>     
>     /*EXDIM 的评估值 evaluation value 设定为0*/
>     Set the evaluation value of the EXDIM to 0 by doing LAYEREVAL=0;
>     For z=1 to TBN do {
>     
>         /*得到每个盒子三维值距离EXDIM 的最近三维值，通过检查差值（dimension与EXDIM的）的绝对值，选择最小值；把这个值赋予 DIMDIF 这个变量*/
>         Get the closest dimension value of each box to the EXDIM by looking at the absolute values of differences between each dimension and EXDIM, and selecting the smallest value; and set the variable DIMDIF to this value.
>         
>         /*累加这些值*/
>         Add those values cumulatively by doing:
>             LAYEREVAL=LAYEREVAL+DMDIF;
>     }
>     /*如果刚刚检查的维度有较小的评价值，则保留这个维度*/
>     If the dimension that has just been examined has a smaller evaluation value, keep that dimension:
>     
>     /*如果LAYEREVAL 的值小于 big number，则把变量LAYEREVAL的值赋给 EVAL，且LAYERTHICKNESS-EXDIM*/
>     If(LAYEREVAL<EVAL)do {
>         EVAL=LAYEREVAL;
>         LAYERTHICKNESS-EXDIM;
>     }
>   }
> }
> RETURN;
> ```
#


/*FINDBOX() 找到最好fit当前缝隙的盒子 函数*/
> ``` C
> /*参数：HMX:缝隙的x方向最大值；HY:缝隙的y方向值；HMY:缝隙的y方向最大值；HZ:缝隙的z方向值；HMZ:缝隙的z方向最大值*/
> FUNCTION FINDBOX(HMX: Maximum X dimension of the gap;
>     HY: Y dimension of the gap;HMY: Maximum Y dimension of the gap;
>     HZ: Z dimension of the gap;HMZ: Maximum Z dimension of the gap);
>     
> /*对于能fit in 当前层厚的盒类型*/
> For the box type fitting in the current layerthickness:
>     BFX=;BFY=;BFZ=;
>     
> /*对于不能fit in 当前层厚的盒类型*/
> For the box type that cannot fit in the current layerthickness, but the closest one:
>     BFX=;BFY=;BFZ=;
>     
> /*只检验不同的盒子*/
> For Y=1 to TBN with step BOXLIST[Y].N do {(Examines only different boxes)
>     /*如果盒子被检验到 被打包，继续下一个循环*/
>     If the box that is being examined has been packed before, continue with the next loop;
>     
>     /*某类箱子中未装箱的箱子索引*/
>     X=The index of the box that has not been packed before among a certain type of box;
>     
>     /*分析所有6种被检验盒子可能的朝向*/
>     Analyze all six possible orientations of the box being examined:
>     
>     /*HMX,HY,HMY,HZ,HMZ,DIM1,DIM2,DIM3 HMX:缝隙的x方向最大值; HY:缝隙的y方向值; HMY:缝隙的y方向最大值; HZ:缝隙的z方向值; HMZ:缝隙的z方向最大值; */
>     ANALYZEBOX(HMX, HY, HMY, HZ, HMZ, BOXLIST[X].DIM1,BOXLIST[X].DIM2,BOXLIST[X].DIM3);
>     ANALYZEBOX(HMX, HY, HMY, HZ, HMZ, BOXLIST[X].DIM1,BOXLIST[X].DIM3,BOXLIST[X].DIM2);
>     ANALYZEBOX(HMX, HY, HMY, HZ, HMZ, BOXLIST[X].DIM2,BOXLIST[X].DIM1,BOXLIST[X].DIM3);
>     ANALYZEBOX(HMX, HY, HMY, HZ, HMZ, BOXLIST[X].DIM2,BOXLIST[X].DIM3,BOXLIST[X].DIM1);
>     ANALYZEBOX(HMX, HY, HMY, HZ, HMZ, BOXLIST[X].DIM3,BOXLIST[X].DIM1,BOXLIST[X].DIM2);
>     ANALYZEBOX(HMX, HY, HMY, HZ, HMZ, BOXLIST[X].DIM3,BOXLIST[X].DIM2,BOXLIST[X].DIM1);
> }
> RETURN;
> ```
#


/*ANALYZEBOX(HMX, HY, HMY, HZ, HMZ, DIM1, DIM2, DIM3) 函数*/
> ``` C
> FUNCTION ANALYZEBOX(HMX, HY, HMY, HZ, HMZ, DIM1, DIM2, DIM3);
> /*如果给定盒子的所有维度 fit 缝隙中的最大空间*/
> (If all dimensions of the given box fit the maximum space in the gap:)
> /**/
> If(DIM1<=HMX and DIM2<=HMY and DIM3<=HMZ) do {
>     /*如果盒子的 当前朝向的 y轴方向 fits 缝隙的层厚*/
>     (If the y-dimension of the current orientation of the box fits to the gap's layer thickness:)
> 
>     /*DIM2<=缝隙的Y方向值*/
>     If(DIM2<=HY)do {
>         /*如果当前盒子 相比较前一个选择的 更好fit y-维度 保留当前 盒子的索引 在变量BOXI 中*/
>         If the current box is a better fit in respect to its y-dimension compared to the one selected before, keep the index of the current box in the variable BOXI;
> 
>         /*如果当前盒子 相比较前一个选择的 有相同y-维度，相比较前一个选择的 当前盒子更好 fit x 维度 保留 当前盒子的索引 在变量 BBOXI 中*/
> If the current box has the same y-dimension as the y-dimension of the selected one before, and the current box is a better fit in respect to its x-dimension compared to the selected one before, keep the index of the current box in the variable BBOXI;
> 
>         /*如果当前盒子 相比较前一个选择的 有相同的y and x-dimensions，相比较前一个选择的 当前的盒子更好fit z-dimension 保留当前 盒子的索引 在变量 BBOXI 中*/
>         If the current box has the same y and x-dimensions as the y and x-dimensions of the selected one before, and the current box is a better fit in respect to its z-dimension compared to the one selected before, keep the index of the current box in the variable BBOXI;
> 
>     }
>     /*如果盒子当前朝向的y-dimension 大于 缝隙的层厚值*/
>     (If the y-dimension of the current orientation of the box is bigger than the gap's layer thickness:)
>     
>     /*>缝隙的Y方向值*/
>     If(DIM2>HY) do {
>         /*如果当前盒子 相比较前一个选择的 更好fit y-dimension，保留当前 盒子的索引 在变量 BBOXI 中*/
>         If the current box is a better fit in respect to its y-dimension compared to the one selected before, keep the index of the current box in the variable BBOXI;
>         
>         /*如果当前盒子 相比较前一个选择的 有相同y-维度，相比较前一个选择的 当前盒子更好 fit x 维度 保留 当前盒子的索引 在变量 BBOXI 中*/
>         If the current box has the same y-dimension as the y-dimension of the selected one before, and the current box is a better fit in respect to its x-dimension compared to the selected one before, keep the index of the current box in the variable BBOXI;
>         
>         /*如果当前盒子 相比较前一个选择的 有相同的y and x-dimensions，相比较前一个选择的 当前的盒子更好fit z-dimension 保留当前 盒子的索引 在变量 BBOXI 中*/
>         If the current box has the same y and x-dimensions as the y and x- dimensions of the selected one before, and the current box is a better fit in respect to its z-dimension compared to the one selected before, keep the index of the current box in the variable BBOXI;
>         
>     }
> }
> RETURN;
> ```
#


/*FINDSMALLESTZ() 用smallest z 值决定当前层 盒子间隙 函数*/
> ``` C
> /*用smallest z 值来决定当前层的 盒子间隙*/
> FUNCTION FINDSMALLESTZ();
> 
> /*取得链接列表的第一个节点 代表当前层的边缘拓扑*/
> Get the first node of the linked list representing the edge topology of the current layer:
>     /*把废料优先值赋给 废料编号*/
>     SCRAPMEMB=SCRAPFIRST;
> /*把值赋给会以smallest z-value 保留节点 的变量*/
> Assign it to the variable which will keep the node with a smallest z-value:
> 
>     /*把废料编号值赋给 最小z值*/
>     SMALLESTZ=SCRAPMEMB;
> /*当 废料编号.位置≠NULL*/
> While SCRAPMEMB.POS≠NULL do{
>     If(SCRAPMEMB.POS).CUMZ<SMALLESTZ.CUMZ then
>         SMALLESTZ=SCRAPMEMB.POS;
> }
> RETURN;
> ```
#

/*CHECKFOUND() 决定pack 哪个盒子 函数*/
> ``` C
> /*决定pack 哪一个盒子*/
> FUNCTION CHECKFOUND();
> 
> /*如果一个 盒子 fit in 当前层 厚度被找到，保留它的索引& 朝向用来打包*/
> (If a box fitting in the current layer thickness has been found, keep its index and orientation for packing:)
> 
> /**/
> If BOXI≠0 then do {CBOXI=BOXI; CBOXX=BOXX; CBOXY=BOXY; CBOXZ=BOXZ};
> Else {
>       /*如果一个 y维值 大于当前层厚 的盒子被找到 &当前层的边缘比选中的更平 则选择盒子设定为 LAYERINLAYER*/
>       If a box with a bigger y-dimension than the current layer thickness has been found and the edge of the current layer is even then select that box and set LAYERINLAYER
>       
>       /*在当前层 的第二层的打包 的变量 更新 LAYERTHICKNESS = BBOXY*/
>       variable for a second layer packing in the current layer and update the LAYERTHICKNESS = BBOXY;
>       Else {
>             /*如果 当前 层的边缘 没有缝隙，层的打包则完成 LAYERDONE=1*/
>             If there is no gap in the edge of the current layer, packing of the layer is done:LAYERDONE=1;
>             
>             /*则，因为没有可以 fitting 的盒子到当前 选择的缝隙，跳过缝隙，通过安排& 更新必要的节点和变量来跳过& 填平缝隙*/
>             Else:Since there is no fitting box to the currently selected gap, skip that gap and even it by arranging and updating the necessary nodes and variables;
>       }
> }
> RETURN;
> ```
#

/*VOLUMECHECK() 检查打包情况 函数*/
> ``` C
> /*函数 VOLUMECHECK() 检查打包情况*/
> FUNCTION VOLUMECHECK();
> Mark the current box as packed: BOXLIST[CBOXI].PACKED=1;
> 
> /*只要被打包就保留当前盒的朝向*/
> Keep the orientation of the current box as it is packed:
>     BOXLIST[CBOXI].PACKX=CBOXX;
>     BOXLIST[CBOXI].PACKY=CBOXY;
>     BOXLIST[CBOXI].PACKZ=CBOXZ;
> /*更新总体打包体积*/
> Update the total packed volume:
>     PACKEDVOLUME=PACKEDVOLUME+BOXLIST[CBOXI].VOL;
> /*更新打包盒的数量*/
> Update the number of boxes packed:PACKEDNUMBOX=PACKEDNUMBOX+1;
> /*迭代 函数 完成后执行 best so far 打包*/
> (If performing the best so far packing after being done with the iterations:)
> If PACKINGBEST=1 do
> ```
#

/*GRAPHUNPACKEDOUT() 检查打包情况 函数*/
> ``` C
> /*如果这个函数被called 是 for 可视化数据，write 必要信息到文件 “VISUAL”*/
> If this function is called for a visualization data out, write the necessary information to the file "VISUAL";
> 
> /*否则 合并未打包盒信息 到报告文件末端*/
> Else merge the unpacked box information to the end of the report file;
> RETURN;
> ```
#

/*OUTPUTBOXLIST() 写入打包信息到文件 函数*/
> ``` C
> OUTPUTBOXLIST();
> 
> /*由使用者输入 转换坐标系统每个盒子的朝向，从最优解到 托盘朝向*/
> Transform the coordinate system and orientation of every box from the best solution format to the pallet orientation entered by the user in the input text file by looking at the value of the variable BESTVARIANT;
> 
> /*转换 的盒子信息（打包后的坐标& 三维）写入 报告REPORT文件*/
> Write the transformed box information(coordinates and the dimensions as is has been packed) to the REPORT file;
> 
> RETURN;
> ```
#

/*REPORT() 函数*/
> ``` C
> FUNCTION REPORT();
> /*设定必要变量 开始找到正确的 最佳打包*/
> Set the necessary variables to start the best packing found properly;
> 
> /*根据 BESTVARIANT 值，决定托盘朝向*/
> According to the BESTVARIANT value, determine the orientation of the pallet;
> 
> /*通知其他函数 正在执行 最优打包*/
> To tell other functions that the best packing found is being performed:
>     PACKINGBEST=1;
>     
> /*将 最优解 写入 可视化数据文件 头部信息*/
> Write the header information about the best solution found to the visualization data file
>     "VISUAL";
>     
> /*将 最优解 写入 报告数据文件 头部信息*/
> Write the header information about the best solution found to the report data file;
> 
> /*通过 call 函数 LISTCANDITLAYERS() 罗列所有 可能的候选值 */
> List all possible candidate values by calling LISTCANDITLAYERS();
> 
> /*通过函数QSORT 升序 分类 阵列LAYERS 关于其的 LAYEREVAL 字段 */
> Sort the array LAYERS in respect to its LAYEREVAL fields in increasing order by using QSORT;
> 
> /*将所有盒子 打包状态 设定为 0 For X=1 to TBN do BOXLIST[X].PACKST=0;*/
> Set all boxes's packed status to 0: For X=1 to TBN do BOXLIST[X].PACKST=0;
> 
> /*执行*/
> do {
>     /*设定 变量 展示 剩余未打包 的第二层高度 到当前高度 LAYERINLAYER=0;*/
>     Set the variable that shows remaining unpacked potential second layer height in the current layer: LAYERINLAYER=0;
>     
>     /*设定 🚩变量 展示 当前层的打包是否完结：LAYERDONE=0;*/
>     Set the flag variable that shows packing of the current layer is finished or not:
>         LAYERDONE=0;
>         
>     /*call PACKLAYER() 函数 来打包层，如果内存错误响应，退出程序;*/
>     Call PACKLAYER(), to pack the layer, and if a memory error is responded, exit the program;
>     
>     /*如果有 一个高度 可用来 packing 当前层，在当前层执行另一个 层打包的动作*/
>     If there is a height available for packing in the current layer, perform another layer packing in the current layer:
>     
>     /**/
>     If LAYERINLAYER*0 do {
>         /**/
>         Get the height available for packing in the current layer as the layer thickness to be packed: LAYERTHICKNESS=LAYERINLAYER;
>         
>         /*Call 函数PACKLAYER() 打包层，如果内存错误响应，退出程序*/
>         Call PACKLAYER(), to pack the layer, and if a memory error is responded, exit the program;
>     }
>     Call FINDLAYER(REMAINPY) to determine the most suitable layer height fitting in the remaining unpacked height of the pallet;
> } While PACKINGS;
> /*取得开始时间 结束时间 的差值*/
> Get the difference of the start time and the finish time;
> 
> /*关闭 可视化数据文件 & 报告文件；*/
> Close both the visualization data file and the report file;
> 
> /*写入 有关打包 的信息到console*/
> Write all the information about packing to the console;
> RETURN;
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
