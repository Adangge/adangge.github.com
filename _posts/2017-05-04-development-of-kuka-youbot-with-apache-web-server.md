---
layout: post
title: KUKA YouBot 实体机开发及 Apache Web Server 应用
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

'''
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
'''

其中，JV[0]是手臂连接base的最大的joint，JV[4]是手臂顶端连接gripper的joint。在测试角度的时候一定要注意每个joint的有效范围，如果出错在后台都会有显示，所以要注意后台信息。以下代码显示的是gripper的打开，其实就是设置gripper的距离值，当然这个值也有有效范围，后台一样会显示给值是否法。

'''
void moveGripperopen(){

     brics_actuator::JointPositions msg;
     
     msg = createGripperPositonCommand(0.0115);
     gripperPublisher.publish(msg);
     
     ros::Duration(2).sleep();
}
'''
