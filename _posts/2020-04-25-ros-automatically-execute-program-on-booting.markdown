---
layout: post
title:  "리눅스 에서 부팅 시 원하는 ROS 프로그램을 자동으로 실행하는 방법"
date:   2020-04-25 21:00:00
author: injae Kim
categories: Robot_operating_system
cover:  "https://images.unsplash.com/photo-1518432031352-d6fc5c10da5a?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=967&q=80"
---

## 리눅스 에서 부팅 시 원하는 ROS 프로그램을 자동으로 실행하는 방법

---

ROS 는 다양한 운영체제와 디바이스 상 에서의 작동을 지원하기 때문에, ROS 프로그램 또한 다양한 상황과 여러가지 목적을 가지고 개발됩니다.

| <img src="https://developer.nvidia.com/sites/default/files/akamai/embedded/images/jetsonNano/JetsonNano-DevKit_Front-Top_Right_trimmed.jpg" alt="Nvidia Jetson Nano" style="zoom: 33%;" /> |
| :----------------------------------------------------------: |
| Nvidia 사의 Jetson Nano 보드<br />출처: Nvidia 공식 홈페이지 |

일반적인 데스크탑 에서 작동되는 프로그램이 아닌 위의 사진과 같은 임베디드 보드 (Nvidia Jetson Nano) 나 로봇 등 에서 작동하는 프로그램을 개발해야 할 때 ROS 는 다양한 운영체제와 기기를 지원하므로 유용하게 사용할 수 있습니다.

저의 경우 소형 로봇 내에 Jetson Nano 를 탑재하며 로봇의 영상처리와 서버와의 데이터 송수신 기능을 ROS 기반의 프로그램으로 개발했던 경험이 있습니다.

이 때 ROS 는 Window, Linux, Mac 등의 운영체제와 Jetson Nano등의 소형 보드, 데스크탑, 랩탑 등 다양한 기기들을 모두 잘 지원하므로 직접 통신 관련 기능을 개발하거나 운영체제 별 세팅을 하지 않아도 ROS 기반의 프로그램을 바로 작성할 수 있다는 장점이 있었습니다.

이렇게 로봇 등에 탑재되는 Jetson Nano와 같은 소형 보드에서 작동하는 ROS 프로그램은 부팅시 프로그래머가 원하는 ROS 프로그램이 자동으로 실행되어야 하는 경우가 자주 발생합니다.

그 이유는, ROS 프로그램이 작동해야 하는 상황이 모니터를 사용할 수 없거나 로봇등의 완제품 내부에 탑재되기 떄문입니다!

| <img src="https://www.therobotreport.com/wp-content/uploads/2018/05/Screen-Shot-2018-05-11-at-12.54.39-PM.png" style="zoom: 67%;" /> |
| :----------------------------------------------------------: |
| Boston dynamics 사의 2족, 4족 보행 로봇<br />출처: Boston dynamics 공식 홈페이지 |

위와같은 로봇에 탑재되어 실행되는 ROS 프로그램은 일반적인 SW 와 달리 모니터가 없는 상황이죠. 따라서 부팅 시 바로 프로그래머가 지정한 ROS 프로그램이 실행되어야 합니다.

**이 글에선 리눅스 에서 부팅 시 프로그래머가 원하는 ROS 프로그램을 실행하는 방법에 대해 알아보겠습니다.**

<br/>

<br/>

## 리눅스 부팅과정과 시작 프로세스

---

리눅스 부팅과정을  간단하게 살펴보면 부트로더의 OS 로드, 커널 로드 단계를 거치게 됩니다.

이후 커널로드가 완료되면 프로세스들을 실행하고 관리하기 위한 Process ID 가 1인 `Init 프로세스`가 실행됩니다.

리눅스 기반의 여러 배포판 들에선 `init` 이라는 이름의 init 시스템을 통해 Init 프로세스를 실행하였는데요, 2015년을 기준으로 여러 리눅스 배포판들이 init을 버리고 `systemd` 라는 init 시스템을 기본 init 시스템으로 채택하게 되었습니다.

`init` 에서 `systemd` 로 init 시스템이 2015년에 전환된 후 `systemd` 기반으로 부팅시 원하는 ROS 프로그램을 실행하는 방법에 대한 자료가 많이 없어서, 직접 글을 작성하게 되었습니다. 

따라서 이 `systemd` 를 활용하여 우리가 원하는 ROS 프로그램을 커널로드 후 자동으로 실행시켜 보도록 하겠습니다!

이 글에서 사용한 리눅스는 `Ubuntu 16.04 LTS` 버전입니다. 2015 년 전에 릴리즈 된 리눅스 배포판의 경우 `init 시스템`을 `systemd` 대신 그대로 사용하고 있는 경우도 있으므로 설치된 리눅스 배포판의 버전과 릴리즈 시점을 확인하시기 바랍니다.

<br/>

### 리눅스의 Init 시스템 의 역사와 변화 과정

---

리눅스가 부팅되면서 커널이 로드된 후에는, 여러 프로세스들을 관리하기 위한 Process ID 가 1인 `Init 프로세스`가 실행됩니다. 

리눅스에서 `ps -ef` 명령어로 현재 실행중인 Process 들의 ID 와 PID = 1 인 init 프로세스를 확인할 수 있습니다.

```bash
$ ps -ef
UID		PID		PPID	CMD
root	1		0		init
root	2		0		[kthreadd]
root	3		2		[migration/0]
...
```

이러한 `init` 프로세스는 추후에 실행되는 프로세스를 관리하고, 종료시 할당된 자원을 회수하는 등의 중요한 기능을 담당하게 됩니다.

2015년 전 까지는 리눅스 배포판 들 에서 `init` 프로세스로 `init` 이라는 이름의 시스템을 사용하고 있었는데요, 2015년 이후 `init` 시스템이 `systemd` 라는 이름의 시스템으로 대체되게 됩니다.

기존의 `init` 시스템은 개발된 지 오래되었기도 하고, 여러 단점들을 가지고 있었습니다.

가장 큰 단점은, 부팅 시 병렬 처리를 지원하지 않는다는 것 이였습니다.

CPU 가 발전하면서, 멀티코어 CPU 가 대부분의 PC 에 장착되게 되었지만, 기존의 init 시스템은 부팅 시 에 이러한 멀티코어에서 처리할 수 있는 병렬성을 완전히 활용하지 못하고 있었습니다.

따라서, 병럴처리를 지원하고 기존의 init 시스템을 개선한 `systemd` 가 등장하게 되었습니다.

`systemd` 의 중요 특징을 간단하게 나열해보면 다음과 같습니다.

- 전체적인 시스템 구조의 개선
- 더 간단한 부팅 프로세스
- 부팅시의 병렬 처리 지원

이 [링크](https://www.tecmint.com/systemd-replaces-init-in-linux/) 에서 init 과 systemd 의 차이점과 특징을 조금 더 자세히 살펴볼 수 있습니다.

<br/>

<br/>

## `Systemd` 를 사용한 부팅 시 프로그램 자동 실행 등록 방법

---

`systemd` 는 오픈소스로 개발되고 있으며 이 [깃 저장소 링크](https://github.com/systemd/systemd) 에서 코드를 확인할 수 있습니다.

`systemd` 에 대한 [공식문서](https://systemd.io/) 나 [위키](https://ko.wikipedia.org/wiki/Systemd) 를 참고하시면 더 자세한 설명과 API 등을 확인할 수 있습니다.

`systemd` 에 부팅시 원하는 프로그램을 실행하도록 등록하는 방법은 다음과 같습니다.

<br/>

### `Systemd` 에 부팅시 자동실행할 프로그램에 대한 실행 스크립트 작성

---

`/etc/systemd/system/` 경로에 자동 실행할 프로그램에 대한 실행 스크립트를 작성해 줍니다.

이때 실행 스크립트는 `.service` 확장자로 지정해주면 됩니다.

저의 경우 부팅 시 `systemd` 가 `/home/sheco/injae.sh` 스크립트를 실행하기를 원하는 상황이므로 다음과같은 `service` 파일을 작성해줍니다.

```bash
# /etc/systemd/system/ros_init.service
[Unit]
Description=Ros Init Daemon

[Service]
Type=simple
ExecStart=/home/sheco/injae.sh	# 부팅 시 실행하고 싶은 스크립트의 경로

[Install]
WantedBy=multi-user.target
```

`systemd` 가 부팅 시 실행할 `.service` 파일은 크게 `Unit`, `Service`, `Install` 로 구성되는데 각각의 항목에는 여러 옵션들을 추가할 수 있습니다.

<br/>

```bash
# service 파일의 다양한 옵션들 예시
[Unit]
Description=Ros Init Daemon 	# 추후 서비스 상태 확인 시 서비스 설명란에 출력됨
Requires=*.service 	# 이 서비스를 실행하기 위해 필요한 서비스 의존성
Before=*.service 	# 이 서비스 실행 후에 실행되어야 할 서비스
After=*.service 	# 이 서비스 실행 전에 실행되어야 할 서비스

[Service]
Type=simple # simple, forking, oneshot, notify, dbus 등 프로그램의 타입 명시를 원하는 경우
ExecStart=/home/sheco/injae.sh	# 부팅 시 실행하고 싶은 스크립트의 경로

[Install]
WantedBy=multi-user.target
```

`[Unit]` 의 `Before`, `After` 를 잘 사용하면 다수의 서비스들 끼리 원하는 순서대로 실행되도록 조정할 수 있습니다.

<br/>

### 실행할 ROS 프로그램 관련 스크립트 작성

---

`systemd` 에 등록한 `ros_init.service` 에서 실행할 파일을  `ExecStart=/home/sheco/injae.sh` 로 정해주었으므로, 해당경로에 해당이름의 스크립트 파일을 생성해 줍니다.

스크립트 파일의 내용은 일반적으로 ros 프로그램을 실행할 때 사용자가 터미널에 작성하는 내용과 일치합니다.

```bash
#! /bin/bash
# 실행하고자 하는 ros 버전에 맞는 setup.bash 실행
source /opt/ros/melodic/setup.bash
source home/sheco/catkin_ws/devel/setup.bash

# ros 실행해 필요한 마스터, 호스트 노드 IP 주소 설정
export ROS_MASTER_URI=http://10.42.0.1:11311
export ROS_HOSTNAME=10.42.0.1

# roscore 실행
sleep 3
cd /home/sheco/catkin_ws
roscore

# 원하는 ros 노드 실행, 이때 rosrun ros_node & 로 background 로 실행시켜야 함
sleep 5
rosrun opencv opencv_publish_node &
```

<br/>

ros 실행 시 설치되어 있는 버전에 맞는 setup.bash 를 먼저 실행해주어야 합니다. 저는 ros melodic 버전을 사용하였습니다.

그 후 ros 실행에 필요한 마스터, 호스트 노드의 IP 주소를 미리 확인하여 export 해주어야 합니다.

저는 이때 일반 와이파이가 아닌 Jetson Nano 보드의 자체 핫스팟을 사용하였으므로 핫스팟의 IP 주소인 10.42.0.1 로 IP 주소를 설정해주었습니다.

핫스팟이 아닌 공용 와이파이를 사용한다면 IP 동적할당 등 으로 부팅 시 마다 IP 주소가 바뀔수 도 있으니, 이런 상황에는 자신의 IP 주소를 받아오는 처리를 추가적으로 해주어야 합니다.

그 후 `roscore` 를 실행한 후, 잠시 기다린 후에 미리 컴파일 해 둔 ros 프로그램을 실행시킵니다.

이떄 주의할 점은, `rosrun ros_node &` 로 & 를 사용하여 background 로 ros 프로그램을 실행시켜 주어야 터미널이 종료되도 ros 프로그램이 계속 실행되게 됩니다.

<br/>

### `Systemd` 에 작성한 자동 실행 `service` 파일 등록

---

거의 다 왔습니다! 여기까지 우리는 `systemd` 에 자동실행 `service` 파일과 실행시키고자 하는 ros프로그램의 자동실행 스크립트 를 작성하였습니다.

남은건 `systemd` 에 부팅시 자동실행할 `service` 파일을 등록하는 것 입니다.

터미널을 실행시켜 다음 명령어를 통해 `systemd` 에 작성한 `service` 파일을 등록시켜 줍시다.

```bash
systemctl daemon-reload
systemctl enable ros_init.service	# 부팅시 이 서비스를 자동시작 함
systemctl start ros_init.service	# 서비스 시작
```

<br/>

`systemctl` 명령어를 통해 `systemd` 에 원하는 `serivce` 를 등록하거나 상태를 확인하는 등의 다양한 기능을 사용할 수 있습니다.

```bash
systemctl status service_name.service	# 서비스 상태 확인
systemctl start service_name.service	# 서비스 시작
systemctl restart service_name.service	# 서비스 재시작
systemctl stop service_name.service		# 서비스 중지
systemctl enable service_name.service	# 부팅시 이 서비스 자동 실행
systemctl disable service_name.service	# 부팅시 이 서비스 자동 실행 해제
systemctl list-units --type=service		# 실행중인 서비스 목록 확인
```

<br/>

`systemctl list-units --type=service` 로 현재 실행중인 서비스 목록을 확인하거나 내가 등록한 서비스가 잘 실행되고 있는지, 실행이 실패했다면 그 이유가 무엇인지를 로그를 통해 확인할 수 있습니다.

![](https://injae-kim.github.io/assets/ROS/2020-04-25-ros-automatically-execute-program-on-booting/service-list.JPG)

위의 사진처럼 제가 작성한 서비스 파일이 부팅 시 자동실행 되도록 등록되어 있는것을 확인할 수 있습니다.

<br/>

![](https://injae-kim.github.io/assets/ROS/2020-04-25-ros-automatically-execute-program-on-booting/service-status.JPG)

위처럼 서비스가 자동실행되는 과정에서 정상적으로 실행되었는지, 만약 실패했다면 그 이유는 무엇인지 에 대하여 확인할 수 도 있습니다.

<br/>

<br/>

## 결론

---

부팅 시 원하는 ROS 프로그램을 자동으로 실행하기 위해, 리눅스에서 프로세스들을 관리하는 `systemd` 를 활용하여 내가 원하는 프로그램의 자동 실행 스크립트를 등록하는 방법을 알아보았습니다.

실제로 Jetson Nano 와 같은 소형 보드에서 ROS 프로그램을 설치하는 경우, 매번 모니터와 키보드를 활용할 수 있는 상황만 존재하는 것 도 아니며 로봇등에 탑재되는 경우 외부에서 실험등을 진행할 때 에는 모니터와 키보드를 아예 사용할 수 없으므로 `아.. 부팅시에 내가 작성한 ROS 프로그램을 자동으로 실행시켜야 겠다!` 라는 생각이 자연스럽게 드는 것 같습니다.

리눅스의 `init` 시스템이 왜 기존의 `init` 을 잘 쓰고 있었는데 `systemd`로 대체되었을까? 에 대한 의문도 이 글을 작성하면서 알게되었습니다.

많은 도움이 되었길 바랍니다!

읽어주셔서 감사합니다.

<br/>

<br/>

## Reference

---

- [우분투 18.04 자동실행, 서비스 등록]([https://nasanx2001.tistory.com/entry/%EC%9A%B0%EB%B6%84%ED%88%AC-1804-%EC%9E%90%EB%8F%99%EC%8B%A4%ED%96%89-%EC%84%9C%EB%B9%84%EC%8A%A4%EB%93%B1%EB%A1%9D](https://nasanx2001.tistory.com/entry/우분투-1804-자동실행-서비스등록))
- [리눅스 systemctl 명령어](https://www.manualfactory.net/10507)
- [Ubuntu 16.04 system service 등록하기]([https://pinedance.github.io/blog/2017/09/12/Ubuntu-16.04-system-service-%EB%93%B1%EB%A1%9D%ED%95%98%EA%B8%B0](https://pinedance.github.io/blog/2017/09/12/Ubuntu-16.04-system-service-등록하기))

- [systemd git repo](https://github.com/systemd/systemd)

- [systemd document](https://systemd.io/)
- [wiki - systemd](https://ko.wikipedia.org/wiki/Systemd)
- [wiki - linux init process]([https://ko.wikipedia.org/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4_%EC%8B%9C%EC%9E%91_%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4](https://ko.wikipedia.org/wiki/리눅스_시작_프로세스))
- [systemd - init process 를 대체하다](https://m.blog.naver.com/PostView.nhn?blogId=xogml_blog&logNo=130141967915&proxyReferer=https:%2F%2Fwww.google.com%2F)
- [The Story Behind ‘init’ and ‘systemd’: Why ‘init’ Needed to be Replaced with ‘systemd’ in Linux](https://www.tecmint.com/systemd-replaces-init-in-linux/)
- [서버 프로세스를 관리하는 올바른 방법](http://www.codeok.net/서버 프로세스를 관리하는 올바른 방법?action=backlinks)
- [Rethinking PID 1](http://0pointer.net/blog/projects/systemd.html)