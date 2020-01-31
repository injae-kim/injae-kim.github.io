---
layout: post
title:  "ROS 에서 OpenCV로 실시간 영상 송수신 구현 방법"
date:   2019-07-08 10:00:00
author: injae Kim
categories: ROS
tags:	ROS OpenCV 영상송수신
cover:  "/assets/ROS/background.jpg"
---

##  ROS 에서 OpenCV  로 실시간 영상 송수신 구현 방법

---



안녕하세요. 
`Jetson tx2` 보드와 `ROS, OpenCV, tensorflow-gpu` 등을 활용하여 임베디드 환경에서 딥러닝 기반 사물 인식 및 semantic segmentation 을 구현해보고 있는 학생 입니다.

ROS 개발을 하다 보니 ROS 상에서 작동하는 리모트 컨트롤러를 만들어야 해서 ROS 가 설치된 디바이스 간의 영상 송수신을 구현해야 되는 상황에 놓였는데, 관련된 한국어 자료나 레퍼런스가 부족하여 간단한 가이드를 작성하게 되었습니다.

`Jetson tx2` 보드에서 `usb_camera` 를 연결하여 영상을 입력 받아 ROS 가 세팅되어 있는 다른 디바이스 (노트북, 임베디드 보드, 스마트폰) 등에 실시간으로 영상을 송수신 하는 방법에 대해 알아보겠습니다

> *구현 과정에서 영상을 있는 그대로 전송하면 처리 소요 시간이 0.3초 이상으로 소요되어 실시간으로 영상 송수신을 할 수 없었습니다. 따라서 영상을 압축하여 ROS 메시지 형식으로 변환 후 전송, 수신부에서 전달받은 메시지를 압축 해제 후 영상으로 다시 변환하도록 구현하였습니다.*

---



#### 폴더 구조

```basic
|catkin_ws
|-- src
	|-- opencv
		|-- include
		|-- src
			|--  pub.cpp
			|--  sub.cpp
		|-- CMakeList.txt
		|-- package.xml
```

ROS 의 기본적인 폴더 구조인 catkin_ws 폴더 에서 `catkin_create_pkg` 명령어로 opencv 패키지를 생성해 주시면 됩니다.



#### ROS Opencv Install

```basic
$ sudo apt-get install ros-melodic-opencv*
$ sudo apt-get install ros-melodic-usb-cam
$ sudo apt-get install ros-melodic-cv-bridge
$ sudo apt-get install ros-melodic-cv-camera
```

ROS 에서 OpenCV 를 사용하기 위해 필요한 패키지를 다운로드 해 줍니다



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

*`find_package (OpenCV REQUIRED)` 를 꼭 추가해 주셔야 빌드가 성공적으로 진행 됩니다.*

catkin_package() 와 include_directories() 에서도 OpenCV 사용을 위한 의존성을 추가해 주시고 rosrun 을 위한 add_executable 을 작성해 주시면 됩니다.



여기까지가 코드 실행을 위한 사전 준비 사항입니다. 이제 바로 소스코드로 넘어가 보겠습니다.

---



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

ROS 에서 메시지를 publish 할 node 를 `opencv_pub` 으로 선언
메시지의 이름을 `camera/image` 로 설정 후 NodeHandle 을 통해 등록
usb_cam 의 영상을 입력받기 위한 `cv::VideoCapture` 선언



```c++
cap >> frame;
```

usb_cam 의 영상을 `cv:: Mat frame` 에 입력받음



```C++
// Encode image       
std::vector<uchar> encode;
std::vector<int> encode_param;

encode_param.push_back(CV_IMWRITE_JPEG_QUALITY);
encode_param.push_back(20);

cv::imencode(".jpg", frame, encode, encode_param);
```

> *영상을 송수신 하는 과정에서 영상 원본을 전송하면 1920x1080 의 고해상도 이므로 직접 테스트 해본 결과 영상 한 프레임 당 수신하는데 0.3초 이상의 소요시간이 걸려서 영상이 끊기면서 보여지게 됩니다. 따라서 opencv 의 이미지 압축 메소드인 `cv::imencode()` 를 사용하여 영상을 압축하여 송신 후 수신 부에서 영상을 압축 해제 하는 방식으로 구현하였습니다.*

압축 (encode) 된 image를 저장 할 변수 `std::vector<uchar> encode` 선언
압축에 사용할 매개변수인 `encode_param` 에 `CV_IMWRITE_JPEG_QUALITY`, `20 (압축률 %)` push
`cv::imencode()` 로 이미지 압축 실행




```c++
// Decode image
cv::Mat decode = cv::imdecode(encode, 1);
```

`cv::imencode()` 로 압축된 이미지를 decode 하려면 `cv::imdecode()` 사용
압축과 해제가 잘 되는지 decode 을 출력해 봄 으로써 확인할 수 있습니다




```c++
// Convert Decoded image to ROS std_msgs format
std_msgs::UInt8MultiArray msgArray;
msgArray.data.clear();
msgArray.data.resize(encode.size());
std::copy(encode.begin(), encode.end(), msgArray.data.begin());

// Publish ROS msg
pub.publish(msgArray);
```

`cv::imencode()` 의 결과로 압축된 이미지는 `unsigned char` 타입의 array 로 return 됨
`unsigned char` 타입의 array를 ROS 메시지로 전송하기 위해선 `std_msgs::UInt8MultiArray` 메시지 형식 사용
메시지 `std_msgs::UInt8MultiArray msgArray` 에 `std::copy()` 로 압축된 이미지를 복사 후`pub.publish(msgArray)` 로 메시지 publish



---



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



#### sub.cpp 설명

```c++
ros::init(argc, argv, "opencv_sub");

cv::namedWindow("view");
cv::startWindowThread();

ros::NodeHandle nh;
ros::Subscriber sub = nh.subscribe("camera/image", 5, imageCallback);

ros::spin();
```

`opencv_sub` 으로 노드 생성
`camera/image` publisher 를 subscribe, callback 함수로 `imageCallback` 등록




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

pusblisher 에서 전송한 압축된 영상을 msg 로 받아 decode 후 출력하는 `imageCallback` 함수`std_msgs::UInt8MultiArray::ConstPtr& array` -> `data` 로 메세지의 데이터에 접근 가능
`cv::imdecode()` 로 메시지로 전달받은 압축된 영상을 압축해제 후 `cv::imshow` 로 영상 출력



---



#### 실행 결과

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

catkin_make 명령어로 빌드를 실행시켜 줍니다. 성공적으로 잘 빌드 되네요



#### .bashrc 수정

```bash
export ROS_MASTER_URI=http://***.***.***.***:11311
export ROS_HOSTNAME=***.***.***.***
#export ROS_MASTER_URI=http://localhost:11311
#export ROS_HOSTNAME=localhost
```

만약 ROS 가 설치된 서로 다른 디바이스 (임베디드 보드 - 노트북) 에서 영상 송수신을 하려면, `pub.cpp` 가 실행되는 디바이스의 ip 를 모든 디바이스의 `ROS_MASTER_URI ` 로 설정 해 주시고 `ROS_HOSTNAME` 은 각각의 디바이스의 ip 로 설정해 주시면 됩니다.



```bash
$ roscore
```
ROS 의 Node 를 실행하기 위해 각각의 디바이스 에서 roscore 를 실행시켜 줍니다



```bash
$ rosrun opencv opencv_pub
```

영상을 송신할 노드에서는 opencv_pub 을 실행시켜 줍니다


```bash
$ rosrun opencv opencv_sub
```

영상을 수신할 노드에서는 opencv_sub  을 실행시켜 줍니다



#### 실행 결과

  <img src="/assets/ROS/ros-opencv.jpg" title="ros-opencv result">

상단의 터미널에선 `opencv_pub `노드를 실행하였으며 좌측이 원본영상, 우측이 압축된 영상 입니다. 압축된 영상을 실제로 확인해보시면 해상도가 상당히 떨어지는 것 을 알 수 있습니다.

하단의 터미널에선 `opencv_sub` 노드를 실행하였으며 압축된 영상을 정상적으로 해제하여 잘 보여주고 있는것을 확인할 수 있습니다.

압축된 영상을 송신하는 `opencv_sub` 노드에서 **20 FPS** 정도의 성능을 보이며 영상이 부드럽게 잘 전송되는 것을 확인할 수 있습니다.

다른 디바이스 간에는 와이파이 연결 상태에 따라서 다르지만, **10 FPS** 의 무난한 성능을 보이는 것을 확인할 수 있었습니다. 다만, 와이파이 연결 상태가 안좋으면 영상 수신에 지연이 발생하는 것을 확인하였습니다. 

---

  <img src="/assets/ROS/rosgraph.png" title="ros-opencv rqt_graph">

실행 결과를 `$ rqt_graph` 로 확인한 결과 입니다. 정상적으로 `pub`, `sub` 노드가 실행되어 있으며 `/camera/image ` 로 등록된 메시지가 잘 전달되고 있는 것을 확인 할 수 있습니다.



---



여기까지 ROS에서 OpenCV 를 활용한 여러개의 디바이스 간 실시간 영상 송수신 방법에 대해서 알아보았습니다.

저는 리모트 컨트롤러 구현을 하려고 했지만 `임베디드 보드를 이용한 CCTV`, `OpenCV 의 다양한 메소드들 을 이용한 객체 검출 및 추적`, `사물 인식 과 얼굴 인식`, `tensorflow 를 사용한 딥러닝`  등 아주 다양한 방면에서 ROS 를 활용할 수 있는 방법중 하나가 OpenCV 를 사용한 실시간 영상 송수신인 것 같습니다.

부족한 점이나 잘못된 점, 오류 등은 `helios789@naver.com` 으로 메일 주시면 수정하겠습니다.
조금이나마 ROS 를 사용하시는 분 들께 도움이 됐으면 합니다.
읽어주셔서 감사합니다!



#### Reference

[ROS-Wiki](http://wiki.ros.org/Documentation)

[ROS-opencv-tutorial](http://wiki.ros.org/vision_opencv)

[ROS-msg-tutorial](http://wiki.ros.org/msg)

[Converting between ROS images and OpenCV images (Python)](http://wiki.ros.org/cv_bridge/Tutorials/ConvertingBetweenROSImagesAndOpenCVImagesPython)