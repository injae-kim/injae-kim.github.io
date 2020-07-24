---
layout: post
title:  "ROS-serial 로 임베디드 보드에서 열화상 센서를 ROS 와 연동하여 사용하는 방법"
date:   2020-07-24 10:00:00
author: injae Kim
categories: Robot_operating_system
# tags:	ROS OpenCV 영상송수신
cover:  "https://images.unsplash.com/photo-1518770660439-4636190af475?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1050&q=80"
---

## ROS-serial 로 임베디드 보드에서 열화상 센서를 ROS 와 연동하여 사용하는 방법

---

안녕하세요! 간만에 ROS 관련 포스팅으로 돌아왔습니다.

오늘은 `ROS-serial` 로 임베디드 보드에서도 다양한 센서와 ROS 를 연동하여 사용하는 방법을 설명드리겠습니다.

| ![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/4511d6f1-9903-4a77-a84b-a3fa35b1e8a1/intro1.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20200724%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20200724T054106Z&X-Amz-Expires=86400&X-Amz-Signature=3fe58cfb4332ef5f139cb11ac3bae4dd57190778c3e5ca46ffcfc1914eaa0e6e&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22intro1.jpg%22) |
| :----------------------------------------------------------: |
|                   rosserial 이 필요한 이유                   |

ROS 는 다양한 디바이스와 운영체제, 개발환경에서도 범용적으로 호환되어 사용할 수 있도록 애초에 설계되고 개발되어 왔습니다.

예를들어, Windows 를 사용하는 데스크탑과 Mac, linux 운영체제를 사용하는 랩탑 사이에서도 ROS 를 사용하면 다른 종류의 디바이스, 운영체제에도 상관없이 ROS 프로그램이 호환되고 서로 데이터를 주고받는 등의 통신이 가능합니다.

이는 로봇에 들어가는 SW 를 만들때 정말 이상적인 최적의 조건이죠!

서로 다른 디바이스 들과 운영체제에서도 아무런 문제 없이 호환되는 ROS 는 이러한 장점으로 인해 최근에는 로보틱스, SLAM 등의 연구 분야에서도 많은 연구자, 개발자 분 들에게 사랑받는 플랫폼 입니다.

**하지만 ROS 는 Windows, Mac, Linux 와 같은 운영체제 위에서 작동하는데요, 만약 이러한 운영체제가 아닌 아두이노 와 같은 임베디드 보드에서 ROS 를 사용할 수 있는 방법은 없을까요?**

Ros-serial 을 사용하면 아두이노와 같은 임베디드 보드에서도 ROS 를 통해 데이터를 송수신 할 수 있습니다!

Ros-serial 과 그 사용법에 대해서 알아보겠습니다.

<br/>

<br/>

## Ros-serial 이 필요한 이유

---

| ![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/a98261f0-85c6-4c1b-8fe6-12413c4bfe11/intro2.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20200724%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20200724T060320Z&X-Amz-Expires=86400&X-Amz-Signature=a5d549691010f29365f76fdfc9276ef0254344a94edfc05794b836731148f60f&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22intro2.jpg%22) | ![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/3e45766b-40c2-4c60-840b-8ae64a8d03cf/intro3.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20200724%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20200724T060322Z&X-Amz-Expires=86400&X-Amz-Signature=5f79232c634e2544215d836a0b655f6159ecc9d4278daf4f3dcf6a6f48b30047&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22intro3.jpg%22) |
| :----------------------------------------------------------: | ------------------------------------------------------------ |
| 소형 임베디드 보드에서 xbee 통신과 Ros-serial 이용하여 ROS 시스템과 데이터 송수신을 하는 개요도 | 아두이노 등의 임베디드 보드에서도 사용가능한 Ros-serial      |

Ros-serial 은 위처럼 ROS 로 구현한 시스템과 임베디드 보드를 ROS 하나로 일관성있게 개발하고 데이터 송수신등을 연동하기 위해 사용됩니다.

일반적인 ROS 와 Ros-serial 의 가장 큰 차이점은, 구동되는 운영체제 입니다.

ROS 는 Windows, Linux, Mac 등의 운영체제가 탑재된 머신에서 구동되는 반면 Ros-serial 은 아두이도나 STM 시리즈 등의 운영체제가 없는 소형 임베디드 보드에서 구동됩니다.

이러한 상황에서 소형 임베디드 보드에 센서를 연결 후 센서에서 받아오는 데이터들을 ROS 로 구현한 시스템 상에 보내고 싶을 때, 소형 임베디드 보드에는 운영체제를 탑재할 수 없어 ROS 를 사용할 수 없습니다.

이때 Ros-serial 을 사용하면 운영체제, 디바이스의 종류에 상관없이 전체 시스템을 ROS 하나로 일관성 있고 깔끔하게 구현할 수 있습니다.

직접 임베디드 보드와 센서를 사용하여 Ros-serial 과 ROS 시스템을 연동하는 방법을 알아보겠습니다.

<br/>

<br/>

## Ros-serial 을 사용하기 위한 준비물

---

| ![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/bc736901-3935-43f9-95c4-440d071f519d/intro4.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20200724%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20200724T062012Z&X-Amz-Expires=86400&X-Amz-Signature=743b7ed5c0d59de03f8a931a07cff7b1f4cab52dce92e16b0b37dba80e924e6d&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22intro4.jpg%22) |
| :----------------------------------------------------------: |
| Ros-serial 을 사용하여 데이터 송수신을 해볼 임베디드 보드와 열화상 센서 |

저의 경우 **열화상 센서를 통해 원거리에서 일정 영역의 온도를 임베디드 보드로 받아온 후, 그 온도 데이터를 Ros-serial 를 통해 리눅스 랩탑에서 구동되고 있는 ROS 프로그램에 전송 후 OpenCV 로 시각화 하기 위해** Ros-serial 을 사용하였습니다.

| ![](https://cdn.sparkfun.com//assets/parts/1/3/1/1/3/SparkFun_MLX90640_Thermal_Imaging_Camera-Demo.gif) |
| :----------------------------------------------------------: |
| MLX90640 열화상 센서를 사용하여 온도값을 실시간으로 받아온 후 시각화 한 결과 |

이 글의 최종 목표는 위처럼 MLX90640 열화상 센서를 사용하여 온도값을 실시간으로 받아온 후 ROS-serial 을 통해 ROS 가 구동되고 있는 랩탑에 전송하여 시각화 하는 것 입니다!

사용된 임베디드 보드는 **NUCLEO-F746ZG** 로 약 3만원 정도에 판매되고 있으며 제조사는 STM, Core Architecture 은 Arm Cortex m7 입니다.

열화상 센서는 [MLX90640](https://www.sparkfun.com/products/14843) 으로 약 7 ~ 8만원 정도에 판매되고 있으며 110도의 시야각, 32 x 24 픽셀의 영역에 대한 온도 정보를 받아올 수 있는 저가형 열화상 센서 입니다.

주의할 점은 Ros-serial 이 지원하는 임베디드 보드를 사용해야 하는 점 입니다. 모든 임베디드 보드에서 Ros-serial 을 사용할 수 있는 것은 아닙니다!

- [Ros 위키 - Rosserial 공식 문서](http://wiki.ros.org/rosserial) 에서 Ros-serial 을 사용할 수 있는 임베디드 보드의 종류를 살펴볼 수 있습니다.

간략하게 살펴보자면 아두이노의 우노, 메가 이상 급의 보드나 Arm Cortex M 계열 이상, STM32 MCU 등의 보드에서 Ros-serial 을 사용할 수 있습니다.

<br/>

Ros-serial 은 다양한 개발환경을 통해 사용 가능한데요, 개발 팀 내에서 개발자 여러명이 함께 개발하기에 가장 좋은 Ros-serial 개발 환경은 `mbed` 인 것 같습니다.

`mbed` 는 임베디드 SW 개발을 위한 웹 IDE 입니다. 개발환경이 통째로 웹 IDE 상에 있으므로 클락 타임 설정 등 임베디드 하드웨어 관련 설정을 따로 해야하는 다른 로컬 개발환경과 달리 웹 상에서 개발자 들 끼리 서로의 개발환경을 공유할 수 있어 팀 내에서 사용하기에 가장 적합한 선택지 라고 생각합니다. 개발환경 세팅을 맞추는데 시간을 뺏기는건 아까우니까요!

또한 `mbed` 는 공식적으로 Ros-serial 을 지원하고 있다는 장점이 있습니다.

`mbed` 상에서 Ros-serial 을 사용하는 방법을 알아보겠습니다.

<br/>

<br/>

## mbed 웹 IDE 에서 Ros-serial 을 사용하는 방법

---

[mbed](https://os.mbed.com/) 에 접속 후 웹 IDE 를 실행해 줍시다.

| ![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/7c1a7039-bd66-456c-a7d3-0e421b8c28c6/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20200724%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20200724T064041Z&X-Amz-Expires=86400&X-Amz-Signature=033c46a9069f7a4dddaa125858816d3711a8216ae0e739168eee5c6dc465d90e&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22) |
| :----------------------------------------------------------: |
|            mbed 에서 사용할 임베디드 보드를 추가             |

새로운 워크스페이스를 만들게 되면, 사용할 임베디드 보드를 선택할 수 있습니다.

<br/>

| ![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/35946e47-3696-4a79-912b-5be9f5d77440/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20200724%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20200724T064043Z&X-Amz-Expires=86400&X-Amz-Signature=2b30f87ab4115d86b7f5973dd28496be7ec8fdb4a1e162d11917d7acebf5bbab&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22) |
| :----------------------------------------------------------: |
|    mbed 에서 공식적으로 지원하는 수 많은 임베디드 보드들     |

mbed 에서는 이미 아두이노나 Arm cortex 계열의 칩을 탑재하고 있는 수 많은 임베디드 보드를 지원하고 있습니다.

[mbed 에서 지원하는 임베디드 보드 종류](https://os.mbed.com/platforms/) 에서 내 임베디드 보드를 mbed 에서 사용 가능 한 지 확인할 수 있습니다.

<br/>

사용할 임베디드 보드를 선택했다면, MLX90640 열화상 센서에서 온도 데이터값을 받아오기 위한 MLX90640 라이브러리를 import 해 줍시다. 

| ![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/e971978f-1cfa-420a-973d-859506143feb/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20200724%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20200724T065054Z&X-Amz-Expires=86400&X-Amz-Signature=635a53152dcfa92a61b5de6a6cca78090633ea4b67f39ccec5652241fee88147&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22) |
| :----------------------------------------------------------: |
|           MLX 90640 열화상 센서 라이브러리 import            |

[mlx90640 공식 라이브러리 git 링크](https://github.com/melexis/mlx90640-library) 에서 mlx90640 라이브러리의 소스코드를 확인할 수 있습니다.

<br/>

| ![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/27e2581a-a5a8-46ac-b574-e03c91e431f0/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20200724%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20200724T065220Z&X-Amz-Expires=86400&X-Amz-Signature=b3cb0ead03e9f4903055359cfe0a2b450809768917515a3ecfdcc12a8184baf6&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22) |
| :----------------------------------------------------------: |
|        mbed 에서 사용할 Ros-serial 라이브러리 import         |

마지막으로, mbed 에서 Ros-serial 을 사용하기 위해 라이브러리 에서 ros 를 검색 후 알맞는 버전의 ros 라이브러리를 import 해 줍니다.

전 `ROS kinetic` 버전을 사용하고 있으므로, `ros_lib_kinetic` 을 import 하였습니다.

이제 Ros-serial 을 mbed 에서 사용할 준비가 끝났습니다! 바로 소스코드로 넘어가보겠습니다.

이 글에서 사용된 모든 소스코드는 [injae-kim/mlx90640-rosserial Git repo](https://github.com/injae-kim/mlx90640-rosserial) 에서 확인하실 수 있습니다.

<br/>

<br/>

## 열화상 센서의 온도 값을 임베디드 보드로 받아오고 Ros-serial 로 전송하는 소스코드 구현

---

| ![img](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/9289b984-870a-4b2c-a3d4-c4dc11cf308b/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20200724%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20200724T065724Z&X-Amz-Expires=86400&X-Amz-Signature=1f9e9e5539df5e59849a8abc318f6c6db44320eb7ca5a00c17a7cb1016c79180&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22) |
| :----------------------------------------------------------: |
| mbed 에서 ros-serial, 열화상 센서 관련 라이브러리가 import 완료된 후의 폴더 구조 |

지금까지 잘 따라오셨다면, 우리의 mbed 폴더 구조는 위처럼 되어있습니다.

MLX90640 열화상 센서와 임베디드 보드는 I2C 통신을 기반으로 온도값 데이터를 송수신하게 되어있는데요, 열화상 센서의 I2C 통신 관련 기본 세팅은 다음과 같은 코드로 할 수 있습니다.

```c++
MLX90640_I2CInit();
MLX90640_I2CFreqSet(1000);           // Set I2C Frequency 1000kHz
MLX90640_SetRefreshRate(0x33, 0x06); // Set rate to 16Hz effective - Works at 800kHz
```

<br/>

```c++
// mlx90640-library / MLX90640_I2C_Driver.cpp
I2C i2c(D14, D15);
```

여기서 주의할 점은 사용하는 임베디드 보드의 데이터 시트를 참조후 I2C 통신에 사용되는 핀 번호를 위처럼 설정해 주어야 합니다. 이는 임베디드 보드마다 다르므로 꼭 데이터 시트를 참조하시길 바랍니다.

<br/>

또한 Ros-serial 에서 통신 주기 관련 기본 값이 낮게 설정되어 있으므로, 이를 상향조정 해 주어야 합니다.

`ros_lib_kinetic / MbedHardWare.h` 에서 baud rate 를 기본값 9600 이 아닌 115200 으로 상향조정 해줍니다.

```c++
// ros_lib_kinetic / MbedHardware.h
class MbedHardware {
  public:
    // baud rate 를 기본값 9600 에서 115200 으로 상향 조정
    MbedHardware(PinName tx, PinName rx, long baud = 115200)
      :iostream(tx, rx){
      baud_ = baud;
      t.start();
    }

    MbedHardware()
      :iostream(USBTX, USBRX) {
        baud_ = 115200;	    // baud rate 를 기본값 9600 에서 115200 으로 상향 조정
        t.start();
    }   
}
```

<br/>

`ros_lib_kinetic / ros / node__handle.h` 에서 INPUT, OUTPUT buffer size 를 충분한 크기로 늘려줍니다.

```c++
// ros_lib_kinetic / ros / node__handle.h
namespace ros {
  using rosserial_msgs::TopicInfo;

  /* Node Handle */
  template<class Hardware,
           int MAX_SUBSCRIBERS=25,
           int MAX_PUBLISHERS=25,
    		// INPUT, OUTPUT buffer size 를 기본값 512 byte 에서 충분한 크기로 늘려줌
           int INPUT_SIZE=5000,
           int OUTPUT_SIZE=5000>
  class NodeHandle_ : public NodeHandleBase_
  {
    protected:
      Hardware hardware_;
  }

```

이제 MLX90640 열화상 센서 에서 온도값 데이터를 임베디드 보드로 받아온 후 Ros-serial 을 통해 전송하기 위한 준비는 끝났습니다!

<br/>

MLX90640 열화상 센서에서 온도값을 받아오는 코드는 다음과 같습니다.

```c++
static float mlx90640To[768]; // 32 x 24 의 온도값을 저장할 배열
const int bufferSize = 192; // 32 x 24 의 온도값 데이터를 4 부분으로 나누어 전송하기 위한 buffer size

void getTemperature() {   
    // mlx90640 센서에 저장되어 있는 온도 값 데이터를 받아옴
    for (int x = 0 ; x < 2 ; x++)  {
        // 센서의 데이터 프레임을 저장할 배열
        uint16_t mlx90640Frame[834];
        
        // mlx90640 의 라이브러리에서 지원하는 데이터 프레임 로드 함수 사용
        int status = MLX90640_GetFrameData(0x33, mlx90640Frame);
        float vdd = MLX90640_GetVdd(mlx90640Frame, &mlx90640);
        float Ta = MLX90640_GetTa(mlx90640Frame, &mlx90640);
    
        float tr = Ta - TA_SHIFT; //Reflected temperature based on the sensor ambient temperature
        float emissivity = 0.95;
    	
        // 읽어온 데이터 프레임에서 온도값 만 mlx90640To 배열에 복사
        MLX90640_CalculateTo(mlx90640Frame, &mlx90640, emissivity, tr, mlx90640To);
    }   
    
    // mlx90640To 배열에 저장된 온도값을 ROS-serial 의 ROS 메시지 형식으로 저장
    for(int i = 0; i < bufferSize; i++){
        msgTempratureArray1.data[i] = mlx90640To[i + (bufferSize * 0)];
        msgTempratureArray2.data[i] = mlx90640To[i + (bufferSize * 1)];
        msgTempratureArray3.data[i] = mlx90640To[i + (bufferSize * 2)];
        msgTempratureArray4.data[i] = mlx90640To[i + (bufferSize * 3)];
    }     
}
```

MLX90640 의 라이브러리의 튜토리얼 코드에 열화상 센서의 데이터 프레임을 불러오고 온도값만 따로 추출하는 코드가 잘 작성되어 있어 참고하였습니다.

온도 값 데이터를 배열에 저장한 후 ROS 메시지로 변환하여 저장해줍니다. ROS 메시지로 변환한 온도값 데이터를 ROS 시스템에 전송하는 코드는 다음과 같습니다.

```c++
void msgPublish()
{
    IRcam1.publish(&msgTempratureArray1);
    wait_ms(80);

    IRcam2.publish(&msgTempratureArray2);
    wait_ms(80);
    
    IRcam3.publish(&msgTempratureArray3);
    wait_ms(80);
    
    IRcam4.publish(&msgTempratureArray4);
    wait_ms(80);
}
```

일반적인 ROS 의 node 에서 ROS 메시지를 publish 하는 것 과 동일하게 메시지를 publish 해주면 됩니다.

다만 메시지를 연속적으로 publish 하면 통신상에 문제가 발생하는 경우가 있어 메시지 publish 사이에 짧은 시간간격을 넣어주었습니다.

<br/>

**여기서 주의할 점은, ROS-serial 을 사용하여 통신할 때 한번에 보낼 수 있는 데이터의 크기가 한정되어 있다는 점 입니다.**

| ![img](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/38ed6e94-0edc-4b51-90b0-56fd6783bae1/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20200724%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20200724T072837Z&X-Amz-Expires=86400&X-Amz-Signature=cca8357142ebfbc4db7273967a492d65d032a59a19187590abd4697b695cba0b&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22) |
| :----------------------------------------------------------: |
| ROS-serial 공식 문서에서도 확인할 수 있는 송수신 데이터 크기의 제한 |

ROS-serial 이 컴퓨팅 파워가 한정된 임베디드 보드에서 사용되기 때문에, 한번에 송수신 할 수 있는 데이터의 크기에도 제한이 있습니다. 보통 512 bytes 로 제한됩니다.

또한 아두이노 에서는 64 bit 부동 소수점 자료형도 지원하지 않습니다.

ROS-serial 을 통해 데이터 송수신을 구현하면서 이러한 데이터 송수신 크기의 제한에 대해서 저도 처음엔 몰랐어서 많이 해맸습니다. 항상 공식문서를 꼼꼼히 읽도록 합시다!

우리가 전송하려는 열화상 센서의 데이터는 32 x 24 배열이며 한 프레임에 총 768 개의 int형 데이터가 포함되어 있습니다.

따라서 한 프레임에 512 byte 를 초과하게 되죠! 따라서 소스코드 상에서 이를 4개의 부분으로 나누어서 따로따로 ROS-serial 을 통해 온도값 데이터를 전송해주는 방식으로 데이터 크기 제한 문제를 해결하였습니다.

<br/>

```c++
// 온도 값을 담고있는 프레임을 4등분 하여 저장할 메시지, node 4개 선언
std_msgs::Float32MultiArray msgTempratureArray1;
std_msgs::Float32MultiArray msgTempratureArray2;
std_msgs::Float32MultiArray msgTempratureArray3;
std_msgs::Float32MultiArray msgTempratureArray4;

ros::Publisher IRcam1("IRcam1", &msgTempratureArray1);
ros::Publisher IRcam2("IRcam2", &msgTempratureArray2);
ros::Publisher IRcam3("IRcam3", &msgTempratureArray3);
ros::Publisher IRcam4("IRcam4", &msgTempratureArray4);

// void getTemperature() 안에서 온도 값을 4등분 하여 ROS-serial 메시지에 저장
for(int i = 0; i < bufferSize; i++)
{
    msgTempratureArray1.data[i] = mlx90640To[i + (bufferSize * 0)];
    msgTempratureArray2.data[i] = mlx90640To[i + (bufferSize * 1)];
    msgTempratureArray3.data[i] = mlx90640To[i + (bufferSize * 2)];
    msgTempratureArray4.data[i] = mlx90640To[i + (bufferSize * 3)];
}
```

위처럼 소스코드에서 온도값이 저장된 배열을 4등분 하여 ROS 메시지에 저장한 후 따로 전송하는 것 을 확인할 수 있습니다.

전체 소스코드는 [injae-kim/mlx90640-rosserial Git repo](https://github.com/injae-kim/mlx90640-rosserial) 에서 확인하실 수 있습니다.

<br/>

<br/>

## ROS 시스템에서 ROS-serial 을 통해 전송된 데이터를 수신하는 방법

---

위에서 구현하였던 MLX90640 열화상 센서와 임베디드 보드, ROS-serial 을 통해 전송된 온도값 데이터를 우분투 16.04, ROS kinetic 버전이 설치된 랩탑에서 수신하는 방법을 알아보겠습니다.

```c++
#include "ros/ros.h"
#include "cv_bridge/cv_bridge.h"
#include "opencv2/highgui/highgui.hpp"
#include "opencv2/imgproc/imgproc.hpp"
#include <sensor_msgs/Image.h>
#include <sensor_msgs/image_encodings.h>
#include <std_msgs/Float32MultiArray.h>
#include <std_msgs/String.h>

using namespace cv;

const int MSG_ARRAY_ROW = 6;
const int MSG_ARRAY_COL = 32;  
const int MSG_ARRAY_LENGTH = 192;

// 온도값을 저장할 배열
Mat tempMap(24, 32, CV_8UC1);
// 온도값을 RGB 로 시각화하여 보여주기 위한 배열
Mat heatMap(24, 32, CV_8UC3);

// 온도값에 따라서 heatmap 적용 후 시각화
void makeHeatMap(){
    for(int row = 0; row < 24; row++){
        for(int col = 0; col < 32; col++){
            heatMap.at<Vec3b>(row, col)[0] = 0;
            heatMap.at<Vec3b>(row, col)[1] = tempMap.at<unsigned char>(row, col) * 3;
            heatMap.at<Vec3b>(row, col)[2] = 255;
        }
    }

}

// 임베디드 보드에서 ROS-serial 을 통해 전송한 ROS 메시지가 전송시 호출되는 callback 함수
void msgCallback1(const std_msgs::Float32MultiArray::ConstPtr& msg{

    // ROS-serial 을 통해 전송된 온도값을 teampMap 배열에 저장함
    for(int i = 0; i < MSG_ARRAY_LENGTH; i ++) {
        int row = (i / MSG_ARRAY_COL) + MSG_ARRAY_ROW * 0;
        int col = i % MSG_ARRAY_COL;

        tempMap.at<unsigned char>(row, col) = msg->data[i] * 4;
    }
	
    // 온도값 시각화 후 OpenCV 를 통해 보여줌
    makeHeatMap();
    imshow( "HeatMap", heatMap );

    waitKey(1);
}


int main(int argc, char **argv)
{
    ros::init(argc, argv, "openCV_sub");
    ros::NodeHandle nh;

    namedWindow("HeatMap", CV_WINDOW_KEEPRATIO);

    // ROS-serial 에서 데이터를 4등분 하여 보내므로, 이를 수신받을 subscriber 또한 4개 선언
    ros::Subscriber sub_IRcamera1 = nh.subscribe("IRcam1", 100, msgCallback1);
    ros::Subscriber sub_IRcamera2 = nh.subscribe("IRcam2", 100, msgCallback2);
    ros::Subscriber sub_IRcamera3 = nh.subscribe("IRcam3", 100, msgCallback3);
    ros::Subscriber sub_IRcamera4 = nh.subscribe("IRcam4", 100, msgCallback4);

    ros::spin();
    return 0;
}
```

ROS-serial 에서 온도값 데이터를 4등분으로 나누어 전송하므로, 이를 수신받을 subscriber 또한 4개를 선언해 줍니다.

ROS-serial 을 통해 전송된 메시지가 도착하면 각각의 callback 함수가 호출되며 온도값을 저장후 OpenCV 를 통해 시각화 하여 보여줍니다.

위에서 사용된 전체 소스코드는 [Git link](https://github.com/injae-kim/Sheco/blob/master/src/openCV/src/openCV_IRcamSub.cpp) 에서 확인하실 수 있습니다.

ROS 에서 OpenCV 를 사용하는 방법은 제 블로그의 [포스트](https://injae-kim.github.io/robot_operating_system/2019/07/08/ros-realtime-video-transmission.html) 를 참고하시면 됩니다.

<br/>

| ![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/b3767715-9619-4ab2-a766-8922c8c92202/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20200724%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20200724T074904Z&X-Amz-Expires=86400&X-Amz-Signature=15a1035386209d28d62fac88d178415b6ea3686d2d9a2d8a19667a39f172cbee&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22) | ![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/4d61924e-daa4-4f3b-ad94-2f7ff3eb0049/1.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20200724%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20200724T074951Z&X-Amz-Expires=86400&X-Amz-Signature=567cc6ec09562d580764ea59b4a88e0ec6380d47d640d07ee244ab78720c3d91&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%221.png%22) |
| :----------------------------------------------------------: | ------------------------------------------------------------ |
|                   뜨거운 물을 담은 텀블러                    | MLX90640 열화상 센서로 뜨거운 물을 담은 텀블러를 촬영한 후 온도 값 데이터를 시각화 한 결과 |

위의 과정을 통해, MLX90640 열화상센서에서 온도값을 임베디드 보드에 받아와 ROS-serial 을 사용하여 ROS 가 구동되고 있는 랩탑에 전송한 후 랩탑에서 ROS 와 OpenCV 를 통해 수신받은 온도값 데이터를 시각화 한 결과는 위와 같습니다.

MLX90640 열화상센서는 저가형 온도 센서로 온도 분해능이 높지 않지만, 간단한 사이드 프로젝트 등 에는 충분히 잘 활용할 수 있는 원거리 온도 센서 입니다.

<br/>

<br/>

## 결론

---

지금까지 ROS-serial 을 사용하여 임베디드 보드에서 데이터를 ROS 시스템에 송수신 하는 방법에 대해서 구체적으로 알아보았습니다.

ROS-serial 에 대한 자료가 많지 않고, 이 글에서 처럼 열화상 센서와 보드를 연동하여 ROS 시스템과 데이터를 실시간으로 송수신하는 방법에 대한 자료는 더더욱 없어서 이 글을 작성하게 되었습니다.

ROS-serial 의 송수신 데이터 크기 제한, 임베디드 보드에 대한 지식 부족, 센서 제조사에서 제공하는 라이브러리를 사용하는 방법에 대한 자료 부족 등 개발 과정에서 난관이 많았던 것 같습니다.

하지만 결국 작동하는 프로그램을 보며 뿌듯했습니다!

ROS 와 ROS-serial 을 사용하는 많은 분 께 이 글이 도움이 되길 바랍니다.

읽어주셔서 감사합니다.

<br/>

<br/>

## Reference

---

- [ROS 공식 위키 - Rosserial 관련 문서](http://wiki.ros.org/rosserial)
- [Rosserial MBED setup tutorial]([http://wiki.ros.org/rosserial_mbed/Tutorials/rosserial_mbed%20Setup](http://wiki.ros.org/rosserial_mbed/Tutorials/rosserial_mbed Setup))
- [mlx90640 열화상 센서 공식 문서](https://github.com/sparkfun/SparkFun_MLX90640_Arduino_Example)

<br/>

<br/>