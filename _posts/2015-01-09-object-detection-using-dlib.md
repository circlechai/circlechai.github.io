---
layout: post
title: 物体检测总结
categories: [machine learning]
---

##D-lib安装
D-lib库代码下载地址：https://github.com/davisking/dlib

##配置D-lib
D-lib不需要依赖别的库。在VS2008中配置，只需要把d-lib-xx.xx添加到包含文件中：
 
##运行自带的例子。 
1. D-lib自带了很多很好且非常实用的例子，要运行D-lib的例子，安装cmake，然后安装：
```shell
cd dlib/test
mkdir build
cd build
cmake ..
cmake --build . --config Release
./dtest --runall
```
进行单元测试。
2. 编译D-lib自带的例子
cd examples
mkdir build
cd build
cmake ..
cmake --build . --config Release
在build目录下，找到Reasese目录，进入所在目录，可以看到生成的exe可执行文件。

##物体检测
###使用自带的可执行文件检测物体

D-lib自带了人脸检测器，如果要检测别的物体，先要训练出对应的检测器。

1.	对训练数据集图像进行标注
D-lib自带了图像标注工作，对应在tools目录下，非常的好用。在使用该工具前，先用cmake对原代码进行编译：

```shell
cd dclib/tools/imglab
mkdir build
cd build
cmake ..
cmake --build . --config Release
```

生成exe可执行文件，然后分下面步骤进行标注。

- 生成XML文件，在该文件中会将训练图片文件名包含在里面

```shell
Win: imglab -c mydataset.xml C:\Users\willard\Desktop\images

Linux: ./imglab -c mydataset.xml /tmp/images
```

- 标注图像

```shell
Win: imglab C:\Users\willard\Desktop\images\mydataset.xml
Linux: ./imglab mydataset.xml
```

运行上面命令后，会弹出下面标注对话框

一旦标注完成，点击菜单保存即可。你还可以通过下面的命令：
imglab mydataset.xml
进行验证。

2.	完成标注后，我们可以用例子中的train_object_detector.exe训练检测器：
```shell
train_object_detector.exe –tv C:\Users\willard\Desktop\images\mydataset.xml
```
迭代收敛后，可以看到检测器在训练集本身的presicion、recall、average presicion是多少。同时在imglab.exe目录下，会生成object_detector.svm检测模板。

模板训练好后，同样可以使用train_object_detector.exe对图片物体进行检测了：
```shell
train_object_detector.exe （路径+）图片
```

##通过修改源码编译后再进行物体检测
D-lib自带的例子很多都是通过paser通过shell传递参数的，这样做在使用时确实很方便，不过我们有时我们希望直接在程序中传递参数。具体怎么修改，这里不列出，主要讲一下需要注意的地方。
###载入图片
D-lib中的例子train_object_dectector用debug编译方式编译后，然后去检测图片，会报“。。。JPEG and PNG files”的错误，这里在编译的时候，如果我们要检测的图片是jpg格式的话，我们需要把libjpeg里面的文件全部加入到所建的项目里面，另外在源码的最前面，要定义define DLIB_JPEG_SUPPORT，同时在source.cpp里面，在`#ifdef DLIB_JPEG_SUPPORT`的前面加上`#define DLIB_JPEG_SUPPORT`，这样程序就可以准确运行了。

2.	使用opencv载入图片
D-lib支持Opencv，所以可以将D-lib和OpenCV结合起来使用。D-lib对OpenCV的支持是2.1以后的版本的，所以如果使用的是2.1以后的版本的话，不需要对D-lib下OpenCV所在的文件夹下的文件进行修改了。如果使用的是2.1（比如我现在使用的是OpenCV2.1），需要把文件夹下各个文件中包含的OpenCV头文件注释掉，即：
```shell
//Dlib dafault
//#include <opencv2/core/core.hpp>
//#include <opencv2/core/types_c.h>
并将OpenCV2.1的头文件包含进来，代码如下：
// OpenCV 2.1.0
#include<cv.h>                 
#include<cvaux.h>
#include<highgui.h>
#include<cxcore.h>
```
修改完后，在载入图片的时候，便不再需要依赖D-lib的libjpeg载入图片功能了，这样在每次建立新项目的时候不再需要将libjpeg里面的文件全部加入到所建的项目里面。只需将all文件夹下的source.cpp添加到项目里去即可。

