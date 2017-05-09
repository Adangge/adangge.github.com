---
layout: post
title: KUKA YouBot实体机开发及Apache Web Server应用
date: 2017-05-04
tags: [KUKA YouBot, Apache Web Server, 教程]
---


这是我的第一篇博文，诣在分享KUKA YouBot的开发经验，同时也是对在学习过程中帮助过我的朋友的一个反馈！

这篇博文的主题有两个:
1. 在YouBot实体机上完成手臂的指定运动，将工作台上的一个小方块从左边移动至右边，暂且称之为"SmartMove";
2. 通过Apache Web Server完成前端开发，在网页上控制YouBot的运动。同时，基于这个WebServer可以开发Android App实现对YouBot的控制。这篇博文不介绍App的开发，有兴趣的朋友可以私信我！

### SmartMove
磨刀不误砍柴工，对于第一次学习ROS开发的朋友，以下基础是有必要掌握的:
1. 通过CatKin创建工作目录和Package， 详见 [link to Catkin!](http://wiki.ros.org/catkin/Tutorials).
2. 建立自己的GitHub账号，学习如何使用git命令。这方面网上有很都资料，大家很容易学会！
3. 安装ROS Wrapper，详见 [link to ROS Wrapper!](http://www.youbot-store.com/wiki/index.php/ROS_Wrapper)。这个Tutorial中包括一个键盘控制YouBot Base的程序,将来在仿真中也会有类似应用。同时，rostopic是非常有用的一个命令，往后会经常用到，例如查看各个joint的角度和速度，详见 [link to rostopic!](http://wiki.ros.org/rostopic)。但是用这个命令无法查看ROBOT电池的电压和HDD的温度信息。这个需要用到一个PYTHON程序，有兴趣的朋友可以私信我！
4. 运行HelloWorld Demo, 详见 [link to HelloWorld Demo!](http://www.youbot-store.com/wiki/index.php/C%2B%2B_Hello_World_example).

以上是很基础的知识，但是非常重要，特别是1和2，掌握扎实，能够事半功倍!

完成了上述4点以后，必要的API和开发基础大家都会有了。我的YouBot是12.04Ubuntu和ROS hydro，现在介绍"SmartMove"程序。这个程序是在HelloWorld Demo基础上改的，主要是手臂的运动。这里一共有20个手臂的动作，除了gripper的张合每一个动作对应5个jointvalues,这个值就是手臂joint的弧度值。同时，速度的控制其实就是控制每个动作之间的间隔时间 'ros::Duration(3).sleep()'。以下代码显示的是一个手臂动作:

```javascript

void moveArmstart(){
    
     brics_actuator::JointPositions msg;
     std::vector<double> jointvalues(5);
     
     jointvalues[0] = 5.84014;
     jointvalues[1] = 1.846735;
     jointvalues[2] = -1.8950;
     jointvalues[3] = 3.02356;
     jointvalues[4] = 2.95;
     
     msg = createArmPositionCommand(jointvalues);
     armPublisher.publish(msg);
     
     ros::Duration(3).sleep();
}
```

其中，JV[0]是手臂连接base的最大的joint，JV[4]是手臂顶端连接gripper的joint。在测试角度的时候一定要注意每个joint的有效范围，如果出错在后台都会有显示，所以要注意后台信息。以下代码显示的是gripper的打开，其实就是设置gripper的距离值，当然这个值也有有效范围，后台一样会显示给值是否法。

```javascript

void moveGripperopen(){

     brics_actuator::JointPositions msg;
     
     msg = createGripperPositonCommand(0.0115);
     gripperPublisher.publish(msg);
     
     ros::Duration(2).sleep();
}
```

理解上述代码以后，大家可以开始自己编写手臂运动。我的完整代码详见 [Apache-server_YouBot_Object_Displacer，main.cpp](https://github.com/Adangge/Apache-Server_YouBot_Object_Displacer/blob/master/main.cpp).

总得来说，对手臂运动包括底盘运动的开发是很基础的，在没有其他传感器的参与下，除了加入前端控制，YouBot的开发相当有限！接下来介绍运用Apache Web Server开发一个网页服务器来控制YouBot完成SmartMove。


### Apache Web Server
首先还是介绍下必要的基础知识:
1. 在Ubuntu下安装并配置Apache Web Server，这里介绍两篇博文，讲的还是非常清楚的，大家做下来应该可以清楚Apache Web Server是什么。[Ubuntu下安装配置Apache http server](blog.csdn.net/ichuzhen/article/details/8217577), [手把手教你在ubuntu上安装apache和mysql和php](blog.csdn.net/guaikai/article/details/6905781).
2. 编写一个Shell Script，并通过Apache在Html Page上加载，介绍一篇英文博文，[Demo:How to invoke a Shell Script on a HTML Page being served by Apache on Linux Machine](https://kuldeeparya.wordpress.com/2014/07/20/demo-how-to-invoke-a-shell-script-on-a-html-page-being-served-by-apache-on-linux-machine/).

准备工作做完之后，其实整个事情就完成一大半了。不过具体的配置和出现的问题是因人而异的，大家耐心做肯定可以完成！现在介绍两个关键的程:
1. **test_script.sh** 这个文件包括IP配置和所有你想要实现的ROS程序。它存在的目录位置通常有两种，**usr/lib/cgi-bin/test_script.sh**和**var/www/cgi-bin**，这取决于个人配置。我的代码详见[Apache-server_YouBot_Object_Displacer, test_script.sh](https://github.com/Adangge/Apache-Server_YouBot_Object_Displacer/blob/master/test_script.sh)。这里关键是要注意修改IP和URI，同时运行ROS程序前要source相关路径，否则会报错！
2. **youbot.html** 这个文件调用了test_script.sh文件，并定义了一个button和一些导言，这些在网页上都会显现。它的位置通常都是在**/var/www/html/youbot.html**。我的代码详见[Apache-server_YouBot_Object_Displacer, youBot.html](https://github.com/Adangge/Apache-Server_YouBot_Object_Displacer/blob/master/youBot.html)。

代码很简单，相信大家都可以理解！只不过是刚开始时不熟悉Apache Web Server,大家从逻辑上可能会不大理解它在Linux下如何和YouBot产生联系。这也是为什么我不建议大家直接 git clone 所有文件的原因，从头开始一步步理解所有程序才会有很大收获。从效率上来讲，这样做似乎比较费时，其实是事半功倍的。只不过很多人想要打好基础，但苦于不知从何入手或者找不到有效的资料而走了不少弯路。

写到这里这篇博文基本上就结束了！最后附上通过WebServer控制YouBot完成SmartMove的视频！
<object width="425" height="344" data='https://youtu.be/P2M-hOoy3Ig'>
</object>
