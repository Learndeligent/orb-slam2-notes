# orb-slam2 notes <1>

Hi,all! This is my first github blog, and here are some notes after reading orb-slam2 source code.

This blog is about the overview of orb-slam2(Homepage：http://webdiis.unizar.es/~raulmur/orbslam/)

## 1. Overview
ORB-SLAM是由Raul Mur-Artal，J. M. M. Montiel和Juan D. Tardos于2015年发表在IEEE Transactions on Robotics。项目主页网址为：http://webdiis.unizar.es/~raulmur/orbslam/。 
ORB-SLAM是一个基于特征点的实时单目SLAM系统，在大规模的、小规模的、室内室外的环境都可以运行。该系统对剧烈运动也很鲁棒，支持宽基线的闭环检测和重定位，包括全自动初始化。ORB-SLAM是一种适用于单目，立体声和RGB-D相机的通用且精确的SLAM解决方案。它能够实时计算摄像机的轨迹以及在各种环境下的稀疏3D场景重建，范围从桌子的小型手持序列到围绕多个城市街区行驶的汽车。它能够关闭大型环路，并实时地从宽基线进行全局重新定位。它包括针对平面和非平面场景的自动且强大的初始化。该系统包含了所有SLAM系统共有的模块：跟踪（Tracking）、建图（Mapping）、重定位（Relocalization）、闭环检测（Loopclosing）。由于ORB-SLAM系统是基于特征点的SLAM系统，故其能够实时计算出相机的轨线，并生成场景的稀疏三维重建结果。ORB-SLAM2在ORB-SLAM的基础上，还支持标定后的双目相机和RGB-D相机。

## 2. System Framework 
整个SLAM系统主要包括跟踪（Tracking）、局部建图（Local Mapping）和闭环检测（LoopClosing）等，在这之前，还有初始化SLAM系统的过程（System），这也构成了SLAM系统的主要的线程：
![Image text](https://github.com/Learndeligent/orb-slam2-notes/blob/master/Framework.jpg)
（1）跟踪：提取ORB特征，根据上一帧进行姿态估计，或者通过全局重定位初始化位姿，然后跟踪已经生成的局部地图来优化位姿，然后通过一些规则来选择是否生成关键帧；
（2）局部建图：这一部分主要完成局部地图构建。包括对关键帧的插入，验证最近生成的地图点并进行筛选，然后生成新的地图点，使用局部捆集调整（Local BA），最后再对插入的关键帧进行筛选，去除多余的关键帧；
（3）闭环检测：这一部分主要分为两个过程，分别是闭环探测和闭环校正。闭环检测先使用WOB进行探测，然后通过Sim3算法计算相似变换。闭环校正，主要是闭环融合和Essential Graph的图优化。

# orb-slam2 notes <2>

At the begining of notes, it's important to figure out the logistic of the whole system, so i arrange the mind map of mono_tum.cc as below:

## 1. The pipeline of mono_tum
main程序里还是很简单易懂的，这里重要的是system部分初始化以及跟踪（TrackMonocular）两个步骤。


## 2. The Initialization of SLAM System
这里主要是对SLAM系统内部的一些类进行初始化，这里主要包括地图、闭环检测和可视化窗口；

## 3. TrackMonocular 
这里主要是构造FRAME的过程，这部分的代码主要在Frame.cpp中，参数中的差异项为ORBextractor，两者提取特征的数量不一致；在FRAME构建完成之后，就可以根据FRAME来跟踪了，即主线程Track，这里的Track分为两个方向，初始化和跟踪。下节会介绍初始化，之后会正式讲解跟踪！

reference:
1. http://webdiis.unizar.es/~raulmur/orbslam/
2. Raúl Mur-Artal, and Juan D. Tardós. ORB-SLAM2: an Open-Source SLAM System for Monocular, Stereo and RGB-D Cameras. ArXiv preprint arXiv:1610.06475, 2016 

