# Boxologic
A set of C programs that calculate the best fit for boxes on a pallet, and visualize the result.  //

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
The first line of the boxlist file defines the X, Y, and Z dimensions of the pallet in which all the boxes are to be packed 
> 
