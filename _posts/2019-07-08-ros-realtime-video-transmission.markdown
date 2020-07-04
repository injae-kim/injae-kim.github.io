---
layout: post
title:  "ROS 에서 OpenCV로 실시간 영상 송수신 구현 방법"
date:   2019-07-08 10:00:00
author: injae Kim
categories: Robot_operating_system
# tags:	ROS OpenCV 영상송수신
cover:  "/assets/ROS/background.jpg"
---

##  ROS 에서 OpenCV  로 디바이스간 실시간 영상 송수신 구현
---

## 목적
안녕하세요. `Jetson tx2` 보드와 `ROS, OpenCV, tensorflow-gpu` 등을 활용하여 임베디드 환경에서 딥러닝 기반 `사물 인식` 및 `Semantic Segmentation` 프로젝트를 진행하고있는 학부생 입니다.

프로젝트 진행 도중, 딥러닝 기반 영상처리는 ROS 가 설치된 `Jetson tx2` 보드 내에서 실행한 후 사물이 인식된 결과 영상을 로봇을 조종하는 조종자의 노트북이나 다른 노드들 에게 무선으로 전송해야 하는 상황이 발생하였습니다.

하지만 `ROS 노드 간 실시간 무선 영상 송수신` 을 한방에 모두 지원해주는 ROS 라이브러리는 존재하지 않았습니다. 레퍼런스나 튜토리얼, 해외자료들도 거의 없다시피 한 상황이였습니다.

**그래서, 직접 구현해 보았습니다.**

ROS 에선 메시지 라는 기능으로 노드 간 무선 통신을 공식적으로 지원하고 있는데,OpenCV 를 통해 영상을 알맞는 메시지 형식으로 변환후 전송하는 방식으로 구현할 수 있었습니다. 

<br/>

---

<br/>

## ROS 세팅

#### 폴더 구조

```basic
catkin_ws
|-- src
	|-- opencv
		|-- include
		|-- src
			|--  pub.cpp
			|--  sub.cpp
		|-- CMakeList.txt
		|-- package.xml
```

ROS 의 기본적인 폴더 구조인 catkin_ws 폴더 에서 `catkin_create_pkg` 명령어로 opencv 이름의 패키지를 생성해 주시면 됩니다.

<br/>

#### ROS Opencv Install

```basic
$ sudo apt-get install ros-melodic-opencv*
$ sudo apt-get install ros-melodic-usb-cam
$ sudo apt-get install ros-melodic-cv-bridge
$ sudo apt-get install ros-melodic-cv-camera
```

ROS 에서 OpenCV 를 사용하기 위해 필요한 패키지를 다운로드 해 줍니다

<br/>

#### package.xml 세팅

```xml
<?xml version="1.0"?>
<package format="2">
  <name>opencv</name>
  <version>0.0.0</version>
  <description>The opencv package</description>

  <maintainer email="helios789@naver.com">helios789</maintainer>

  <license>MIT</license>

  <buildtool_depend>catkin</buildtool_depend>
    
  <build_depend>cv_bridge</build_depend>
  <build_depend>image_transport</build_depend>
  <build_depend>roscpp</build_depend>
  <build_depend>sensor_msgs</build_depend>
  <build_depend>std_msgs</build_depend>
    
  <build_export_depend>cv_bridge</build_export_depend>
  <build_export_depend>image_transport</build_export_depend>
  <build_export_depend>roscpp</build_export_depend>
  <build_export_depend>sensor_msgs</build_export_depend>
  <build_export_depend>std_msgs</build_export_depend>
    
  <exec_depend>cv_bridge</exec_depend>
  <exec_depend>image_transport</exec_depend>
  <exec_depend>roscpp</exec_depend>
  <exec_depend>sensor_msgs</exec_depend>
  <exec_depend>std_msgs</exec_depend>
    
  <export></export>
</package>

```

ROS 의 일반적인 package.xml 작성법과 동일합니다.
`<build_depend>`, `<build_export_depend>`, `<exec_depend>` 태그에 실행에 필요한 `cv_bridge`, `image_transport` (OpenCV 관련), `roscpp` (c++ 사용), `sensor_msgs`, `std_msgs` (ROS 메시지 사용, Node 별 송수신 구현) 을 추가해 줍니다.

<br/>

#### CMakeLists.txt 작성

```basic
cmake_minimum_required(VERSION 2.8.3)
project(opencv)

find_package(catkin REQUIRED COMPONENTS
  cv_bridge
  image_transport
  roscpp
  sensor_msgs
  std_msgs
)

find_package (OpenCV REQUIRED)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES opencv
  CATKIN_DEPENDS cv_bridge image_transport roscpp sensor_msgs std_msgs
# DEPENDS system_lib
)

include_directories(
# include
  ${catkin_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
)

add_executable(opencv_pub src/pub.cpp)
target_link_libraries(opencv_pub ${catkin_LIBRARIES} ${OpenCV_LIBRARIES})

add_executable(opencv_sub src/sub.cpp)
target_link_libraries(opencv_sub ${catkin_LIBRARIES} ${OpenCV_LIBRARIES})
```

`find_package(cakin REQUIRE COMPONETS ${your packages...} )` 부분에서 opencv 와 Node 간 메시지 전송에 필요한 `  cv_bridge, image_transport, roscpp, sensor_msgs, std_msgs` 를 추가해 줍니다.

**`find_package (OpenCV REQUIRED)` 를 꼭 추가해 주셔야 빌드가 성공적으로 진행 됩니다.**

알맞는 OpenCV 버전을 찾을수 없다면, 디바이스에 설치된 OpenCV 버전을 확인하신 후 `find_package (OpenCV 3.3.1 REQURIED)` 처럼 해당 버전을 명시하여 작성해주시면 빌드가 정상적으로 진행됩니다.

catkin_package() 와 include_directories() 에서도 OpenCV 사용을 위한 의존성을 추가해 주시고 rosrun 을 위한 add_executable 을 작성해 주시면 됩니다.

<br/>

---

<br/>

## 소스코드

소스코드는 [github link](https://github.com/injae-kim/ROS-OpenCV) 에서 확인하실 수 있습니다.
<br/>

#### pub.cpp 전체 코드

```c++
#include <cv_bridge/cv_bridge.h>
#include <opencv2/opencv.hpp>
#include <image_transport/image_transport.h>
#include <iostream>
#include <vector>
#include <ros/ros.h>
#include <opencv2/highgui/highgui.hpp>
#include <std_msgs/UInt8MultiArray.h>

int main(int argc, char** argv)
{
    ros::init(argc, argv, "opencv_pub");
    
    ros::NodeHandle nh;
    ros::Publisher pub = nh.advertise<std_msgs::UInt8MultiArray>("camera/image", 1);

    cv::VideoCapture cap(0);
    cv::Mat frame;

    
    while(nh.ok())
    {
        cap >> frame;

        if(!frame.empty())
        {

            cv::imshow("frame", frame);

            // Encode, Decode image example            
            std::vector<uchar> encode;
            std::vector<int> encode_param;
            
            encode_param.push_back(CV_IMWRITE_JPEG_QUALITY);
            encode_param.push_back(20);
            
            cv::imencode(".jpg", frame, encode, encode_param);
            cv::Mat decode = cv::imdecode(encode, 1);
            cv::imshow("decode", decode);

            // Convert encoded image to ROS std_msgs format
            std_msgs::UInt8MultiArray msgArray;
            msgArray.data.clear();
            msgArray.data.resize(encode.size());
            std::copy(encode.begin(), encode.end(), msgArray.data.begin());

            // Publish msg
            pub.publish(msgArray);

            cv::waitKey(1);
          
        }

        ros::spinOnce();
    }

    return 0;
    
}
```



---



#### pub.cpp 설명

```c++
ros::init(argc, argv, "opencv_pub");

ros::NodeHandle nh;
ros::Publisher pub = nh.advertise<std_msgs::UInt8MultiArray>("camera/image", 1);

cv::VideoCapture cap(0);
cv::Mat frame;
```

ROS 에서 메시지를 publish 할 node 를 `opencv_pub` 으로 선언하였습니다.

이때 메시지 queue size 를 1로 정하였는데, queue size 가 작을수록 영상 송수신의 지연이 덜하지만 끊김현상이 자주 나타나고, queue size 를 늘리면 영상 끊김은 감소하지만 지연 시간이 증가합니다.

<br/>

```C++
// Encode image       
std::vector<uchar> encode;
std::vector<int> encode_param;

encode_param.push_back(CV_IMWRITE_JPEG_QUALITY);
encode_param.push_back(20);

cv::imencode(".jpg", frame, encode, encode_param);
```

고해상도의 원본 영상을 보내기 보다는 OpenCV 에서 지원하는 jpg 포맷의 압축 기능을 이용하여 압축하여 전송하는 방법을 택하였습니다.

실제로 원본 영상을 그대로 전송 시 와이파이의 상태에 따라서 지연이 매우 심한 상황이 발생하였는데 압축률의 조정으로 해결할 수 있었습니다.

`encode_param` 에 압축률을 지정할 수 있으며 임의로 20으로 선택하였습니다.

<br/>


```c++
// Decode image
cv::Mat decode = cv::imdecode(encode, 1);
```

`cv::imencode()` 로 압축된 이미지를 decode 하려면 `cv::imdecode()` 사용하면 됩니다.

<br/>


```c++
// Convert Decoded image to ROS std_msgs format
std_msgs::UInt8MultiArray msgArray;
msgArray.data.clear();
msgArray.data.resize(encode.size());
std::copy(encode.begin(), encode.end(), msgArray.data.begin());

// Publish ROS msg
pub.publish(msgArray);
```

`cv::imencode()` 의 결과로 압축된 이미지는 `unsigned char` 타입의 array 로 반환됩니다.
따라서 `unsigned char` 타입의 array를 ROS 메시지로 전송하기 위해선 `std_msgs::UInt8MultiArray` 메시지 형식 사용하면 됩니다.

`std_msgs::UInt8MultiArray msgArray` 에 `std::copy()` 로 압축된 이미지를 복사 후`pub.publish(msgArray)` 로 메시지를 다른 노드들에게 전송해 줍니다.

<br/>

---

<br/>

#### sub.cpp 전체 코드

```c++
#include <ros/ros.h>
#include <opencv2/opencv.hpp>
#include <image_transport/image_transport.h>
#include <opencv2/highgui/highgui.hpp>
#include <cv_bridge/cv_bridge.h>
#include <vector>
#include <std_msgs/UInt8MultiArray.h>

void imageCallback(const std_msgs::UInt8MultiArray::ConstPtr& array)
{
  try
  {
    cv::Mat frame = cv::imdecode(array->data, 1);
    cv::imshow("view", frame);
    cv::waitKey(1);
  }
  catch (cv_bridge::Exception& e)
  {
    ROS_ERROR("cannot decode image");
  }
}

int main(int argc, char **argv)
{
  ros::init(argc, argv, "opencv_sub");

  cv::namedWindow("view");
  cv::startWindowThread();

  ros::NodeHandle nh;
  ros::Subscriber sub = nh.subscribe("camera/image", 5, imageCallback);

  ros::spin();
  cv::destroyWindow("view");
}
```

<br/>

#### sub.cpp 설명

```c++
ros::init(argc, argv, "opencv_sub");

cv::namedWindow("view");
cv::startWindowThread();

ros::NodeHandle nh;
ros::Subscriber sub = nh.subscribe("camera/image", 5, imageCallback);

ros::spin();
```

영상을 수신할 `opencv_sub` 노드를 생성합니다.
영상 송신 노드의 `camera/image` 메시지를 subscribe 해주고 callback 함수로 `imageCallback` 등록합니다.

<br/>


```c++
void imageCallback(const std_msgs::UInt8MultiArray::ConstPtr& array)
{
  try
  {
    // Decode image in msg
    cv::Mat frame = cv::imdecode(array->data, 1);
    cv::imshow("view", frame);
    cv::waitKey(1);

    clock_t end = clock();
    ROS_INFO("%0.4f sec..", (double)(end-start) / 100000); 
    start = end;
  }
  catch (cv_bridge::Exception& e)
  {
    ROS_ERROR("cannot decode image");
  }
}
```

pusblisher 에서 전송한 압축된 영상을 msg 로 받아 decode 후 출력하는 `imageCallback` 함수 입니다. 

`std_msgs::UInt8MultiArray::ConstPtr& array -> data`  로 메세지의 데이터에 접근 이 가능합니다.
메시지 안에 저장된 데이터는 압축된 영상이므로 `cv::imdecode()` 로 메시지로 전달받은 압축된 영상을 압축해제 후 `cv::imshow` 로 수신된 영상을 볼 수 있습니다.

<br/>

---

<br/>

## 빌드 및 실행 결과

#### catkin_make 빌드

```bash
helios789@helios789-17ZD990-VX70K:~/catkin_ws$ cm
Base path: /home/helios789/catkin_ws
Source space: /home/helios789/catkin_ws/src
Build space: /home/helios789/catkin_ws/build
Devel space: /home/helios789/catkin_ws/devel
Install space: /home/helios789/catkin_ws/install
####
#### Running command: "make cmake_check_build_system" in "/home/helios789/catkin_ws/build"
####
####
#### Running command: "make -j8 -l8" in "/home/helios789/catkin_ws/build"
####
[ 25%] Built target talker
[ 50%] Built target listener
[ 75%] Built target opencv_pub
[100%] Built target opencv_sub
```

catkin_make 명령어로 빌드를 실행시켜 줍니다.

<br/>

#### .bashrc 수정

```bash
export ROS_MASTER_URI=http://***.***.***.***:11311
export ROS_HOSTNAME=***.***.***.***
#export ROS_MASTER_URI=http://localhost:11311
#export ROS_HOSTNAME=localhost
```

ROS 가 설치된 서로 다른 디바이스 (임베디드 보드 - 노트북 등) 에서 영상 송수신을 하려면, `pub.cpp` 가 실행되는 영상 송신 측 디바이스의 ip 를 모든 영상 수신 측 디바이스의 `ROS_MASTER_URI ` 로 설정 해 주시고 `ROS_HOSTNAME` 은 각각의 디바이스의 ip 로 설정해 주어야 합니다.

<br/>

```bash
$ roscore
$ rosrun opencv opencv_pub
$ rosrun opencv opencv_sub
```
ROS 의 Node 를 실행하기 위해 각각의 디바이스 에서 roscore 와 영상 송신을할 노드 pub, 영상 수신을 할 노드 sub 를 실행시켜 줍니다

---



#### 실행 결과

  <img src="/assets/ROS/ros-opencv.jpg" title="ros-opencv result">

상단의 터미널에선 `opencv_pub `노드를 실행하였으며 좌측이 원본영상, 우측이 압축된 영상 입니다.

하단의 터미널에선 `opencv_sub` 노드를 실행하였으며 압축된 영상을 수신후 정상적으로 압축해제하여 잘 보여주고 있는것을 확인할 수 있습니다.

다른 디바이스 간에는 와이파이 연결 상태에 따라서 다르지만, **10 FPS** 의 성능을 보이는 것을 확인할 수 있었습니다. 다만, 와이파이 연결 상태가 안좋으면 영상 수신에 지연이 발생하는 것을 확인하였습니다. 

<br/>

<br/>

  <img src="/assets/ROS/rosgraph.png" title="ros-opencv rqt_graph">

실행 결과를 `$ rqt_graph` 로 확인한 결과 입니다. 정상적으로 `pub`, `sub` 노드가 실행되어 있으며 `/camera/image ` 로 등록된 메시지가 잘 전달되고 있는 것을 확인 할 수 있습니다.

<br/>

---

<br/>

 `ROS 에서 OpenCV 를 활용한 디바이스간 실시간 무선 영상 송수신 방법` 을 소개드렸습니다.

이를 활용하여 ROS 가 설치된 디바이스 간 영상 송수신이 가능하고, 이를 드론 조종이나 RC카 조종 등 ROS 위에서 작동하는 모든 애플리케이션에 적용이 가능하리라고 생각됩니다.

진행하시는 ROS 관련 프로젝트에 조금이나마 도움이 되면 좋겠습니다!

읽어주셔서 감사합니다!

<br/>

#### Reference

[ROS-Wiki](http://wiki.ros.org/Documentation)

[ROS-opencv-tutorial](http://wiki.ros.org/vision_opencv)

[ROS-msg-tutorial](http://wiki.ros.org/msg)

[Converting between ROS images and OpenCV images (Python)](http://wiki.ros.org/cv_bridge/Tutorials/ConvertingBetweenROSImagesAndOpenCVImagesPython)