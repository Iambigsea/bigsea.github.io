
## 什么是OpenGL ES？
    OpenGL（全写Open Graphics Library）是指定义了一个跨编程语言、跨平台的编程接口规格的专业的图形程序接口。它用于三维图像（二维的亦可），是一个功能强大，调用方便的底层图形库。
OpenGL在不同的平台上有不同的实现，但是它定义好了专业的程序接口，不同的平台都是遵照该接口来进行实现的，思想完全相同，方法名也是一致的，所以使用时也基本一致，只需要根据不同的语言环境稍有不同而已。OpenGL这套3D图形API从1992年发布的1.0版本到目前最新2014年发布的4.5版本，在众多平台上多有着广泛的使用。
    
    OpenGL ES (OpenGL for Embedded Systems) 是 OpenGL 三维图形 API 的子集，针对手机、PDA和游戏主机等嵌入式设备而设计。
    
OpenGL ES相对于OpenGL来说，减少了许多不是必须的方法和数据类型，去掉了不必须的功能，对代价大的功能做了限制，比OpenGL更为轻量。在OpenGL ES的世界里，没有四边形、多边形，无论多复杂的图形都是由点、线和三角形组成的，也去除了glBegin/glEnd等方法。
## OpenGL ES可以做什么？
OpenGL ES是手机、PDA和游戏主机等嵌入式设备三维（二维也包括）图形处理的API，当然是用来在嵌入式设备上的图形处理了，OpenGL ES 强大的渲染能力使其成为我们在嵌入式设备上进行图形处理的优良选择。我们经常使用的场景有：<br>

* 图片处理。比如图片色调转换、美颜等。
* 摄像头预览效果处理。比如美颜相机、恶搞相机等。
* 视频处理。摄像头预览效果处理可以，这个自然也不在话下了。
* 3D游戏。比如神庙逃亡、都市赛车等。

## OpenGL ES版本及Android支持情况
OpenGL ES当前主要版本有1.0/1.1/2.0/3.0/3.1。这些版本的主要情况如下：

* OpenGL ES1.0是基于OpenGL 1.3的，OpenGL ES1.1是基于OpenGL 1.5的。Ａndroid 1.0和更高的版本支持这个API规范。OpenGL ES 1.x是针对固定硬件管线的。
* OpenGL ES2.0是基于OpenGL 2.0的，不兼容OpenGL ES 1.x。Android 2.2(API 8)和更高的版本支持这个API规范。OpenGL ES 2.x是针对可编程硬件管线的。
* OpenGL ES3.0的技术特性几乎完全来自OpenGL 3.x的，向下兼容OpenGL ES 2.x。Android 4.3(API 18)及更高的版本支持这个API规范。
* OpenGL ES3.1基本上可以属于OpenGL 4.x的子集，向下兼容OpenGL ES3.0/2.0。Android 5.0（API 21）和更高的版本支持这个API规范。
## OpenGL ES 2.0中基本概念
学习OpenGL ES 2.0需要知道OpenGL ES 2.0相关的一些概念及知识。 
在上段中提到了OpenGL ES 2.0相对1.x全新的两个重要东西——顶点着色器和片元着色器。
#### 顶点着色器
    着色器（Shader）是在GPU上运行的小程序。从名称可以看出，可通过处理它们来处理顶点。此程序使用OpenGL ES SL语言来编写。它是一个描述顶点或像素特性的简单程序。

