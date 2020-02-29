# orb-slam2 notes <1> Overview

Hi,all! This is my first github blog, and here are some notes after reading orb-slam2 source code.

This blog is about the overview of orb-slam2(Homepage：http://webdiis.unizar.es/~raulmur/orbslam/)

## 1. Overview
ORB-SLAM是由Raul Mur-Artal，J. M. M. Montiel和Juan D. Tardos于2015年发表在IEEE Transactions on Robotics。项目主页网址为：http://webdiis.unizar.es/~raulmur/orbslam/。 
ORB-SLAM是一个基于特征点的实时单目SLAM系统，在大规模的、小规模的、室内室外的环境都可以运行。该系统对剧烈运动也很鲁棒，支持宽基线的闭环检测和重定位，包括全自动初始化。ORB-SLAM是一种适用于单目，立体声和RGB-D相机的通用且精确的SLAM解决方案。它能够实时计算摄像机的轨迹以及在各种环境下的稀疏3D场景重建，范围从桌子的小型手持序列到围绕多个城市街区行驶的汽车。它能够关闭大型环路，并实时地从宽基线进行全局重新定位。它包括针对平面和非平面场景的自动且强大的初始化。该系统包含了所有SLAM系统共有的模块：跟踪（Tracking）、建图（Mapping）、重定位（Relocalization）、闭环检测（Loopclosing）。由于ORB-SLAM系统是基于特征点的SLAM系统，故其能够实时计算出相机的轨线，并生成场景的稀疏三维重建结果。ORB-SLAM2在ORB-SLAM的基础上，还支持标定后的双目相机和RGB-D相机。


## 2. System Framework 
整个SLAM系统主要包括跟踪（Tracking）、局部建图（Local Mapping）和闭环检测（LoopClosing）等，在这之前，还有初始化SLAM系统的过程（System），这也构成了SLAM系统的主要的线程：
![Image text](https://github.com/Learndeligent/orb-slam2-notes/blob/master/images/Framework.png)
（1）跟踪：提取ORB特征，根据上一帧进行姿态估计，或者通过全局重定位初始化位姿，然后跟踪已经生成的局部地图来优化位姿，然后通过一些规则来选择是否生成关键帧；
（2）局部建图：这一部分主要完成局部地图构建。包括对关键帧的插入，验证最近生成的地图点并进行筛选，然后生成新的地图点，使用局部捆集调整（Local BA），最后再对插入的关键帧进行筛选，去除多余的关键帧；
（3）闭环检测：这一部分主要分为两个过程，分别是闭环探测和闭环校正。闭环检测先使用WOB进行探测，然后通过Sim3算法计算相似变换。闭环校正，主要是闭环融合和Essential Graph的图优化。

# orb-slam2 notes <2> Pipeline of system

At the begining of notes, it's important to figure out the logistic of the whole system, so i arrange the mind map of mono_tum.cc as below:

## 1. The pipeline of mono_tum
main程序里还是很简单易懂的，这里重要的是system部分初始化以及跟踪（TrackMonocular）两个步骤。
![Image text](https://github.com/Learndeligent/orb-slam2-notes/blob/master/images/mono_tum.png)

## 2. The Initialization of SLAM System
这里主要是对SLAM系统内部的一些类进行初始化，这里主要包括地图、闭环检测和可视化窗口；
![Image text](https://github.com/Learndeligent/orb-slam2-notes/blob/master/images/SLAM_ini.png)

## 3. TrackMonocular 
这里主要是构造FRAME的过程，这部分的代码主要在Frame.cpp中，参数中的差异项为ORBextractor，两者提取特征的数量不一致；在FRAME构建完成之后，就可以根据FRAME来跟踪了，即主线程Track，这里的Track分为两个方向，初始化和跟踪。下节会介绍初始化，之后会正式讲解跟踪！
![Image text](https://github.com/Learndeligent/orb-slam2-notes/blob/master/images/TrackMonocular.png)


# orb-slam2 notes <3> Initialization

this part is about Monocular initialization and the code can be found in Tracking::MonocularInitialization

## 1. The pipeline of monocular initialization
单目初始化的程序逻辑主要如下图所示，这里需要注意的是orb-slam2对于初始化的要求还是比较高的
![Image text](https://github.com/Learndeligent/orb-slam2-notes/blob/master/images/MonocularInitialization.png)

## 2. How to solve H, F, R and t?
这里的内容主要是一些理论知识，可以参考Multiview geometry这本书，代码中也会多次引用该书本上的知识：这里的重点在于掌握求H和F的方法，以及如何通过H和F来reconstruct出motion，即R和t
![Image text](https://github.com/Learndeligent/orb-slam2-notes/blob/master/images/SolveHandF.png)

## 3. Create Initial Map
![Image text](https://github.com/Learndeligent/orb-slam2-notes/blob/master/images/CreateInitialMapMonocular.png)



# orb-slam2 notes <4> Track

After initialization, the thread of tracking can be working which consist of three parts as belows:
![Image text](https://github.com/Learndeligent/orb-slam2-notes/blob/master/images/Track().png)

    ## 1. Estimation of Camera Pose
这里主要有三种方法：基于匀速模型，指的是前面两帧可以求出一个速度，假设速度是恒定的，那就可以根据上一帧的姿态和运动速度来求得当前帧的姿态，然后通过上一帧的MapPoints的投影点来缩小匹配的区域，从而来加速匹配，这也考虑到实际上相机的载体可能是匀速前进的；实际代码中若匹配点数量过少，会扩大搜索的区域；基于关键帧，当速度为空或者通过匀速模型求出来的匹配点过少时，则采用基于关键帧的位姿估计，这里采用的关键帧就是上一帧关键帧，通过BoW来加速匹配，特征点的匹配关系通过MapPoints来进行维护，优化的初值为上一帧的位姿；基于重定位，只有在上述两个方法track(匹配)的效果都太差时，Relocalization才出手，与“基于关键帧”的处理方式相像，只不过是在整个关键帧数据库里面去找和当前帧BoW接近的关键帧，然后依次对它们遍历，直到找到内点数量较多的关键帧；
![Image text](https://github.com/Learndeligent/orb-slam2-notes/blob/master/images/Ini_Tcw_3model.png)

    ## 2. Track Local Map
主要有三部分：1、更新局部地图点和局部地图关键帧；2、在所有的局部地图点中查找和当前帧的匹配点；3、优化位姿，判断跟踪局部地图是否成功；
![Image text](https://github.com/Learndeligent/orb-slam2-notes/blob/master/images/TrackLocalMap.png)

    ## 3. Create New Keyframe


# orb-slam2 notes <5> Build Local Map

The task of this thred is to insert new keyframe, update map points and key frame, and local BA
![Image text](https://github.com/Learndeligent/orb-slam2-notes/blob/master/images/LocalMapping().png)

    ## 1. Insert New KeyFrame
关键帧插入的流程是先从缓冲队列中取出关键帧，然后计算该关键帧特征点的BoW，然后给当前关键帧和那些局部地图与其匹配上的MapPoints设置关联，然后更新Covisibility Graph，即更新当前关键帧与其他关键帧的连接关系，然后将该关键帧插入地图
![Image text](https://github.com/Learndeligent/orb-slam2-notes/blob/master/images/ProcessNewKeyFrame.png)

    ## 2、Local BA
这里需要注意的优化的对象包括两个：一是与当前关键帧相连的关键帧，二是上述关键帧相连的地图点，即CreateNewMapPoints()中生成的MapPoints；

    ## 3、局部关键帧剔除和局部地图点剔除
因为每一步都会有冗余的关键帧需要剔除，这就导致有一些地图点的情况也随之改变，所以这两者是相应进行的；除此之外，局部地图点筛选的对象是最近添加的点。

# orb-slam2 notes <6> Loop Closing

It's important to introduce LoopClosing into slam system for the consideration of accumulative error, the structure of Loop Closing of orb-slam is as below:

## 1. Detect Loop 


reference:
1. http://webdiis.unizar.es/~raulmur/orbslam/
2. Raúl Mur-Artal, and Juan D. Tardós. ORB-SLAM2: an Open-Source SLAM System for Monocular, Stereo and RGB-D Cameras. ArXiv preprint arXiv:1610.06475, 2016 

