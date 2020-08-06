---
layout: post
title:  "싱글톤 패턴이 필요한 이유와 실제 서비스에 적용까지"
date:   2020-08-06 10:00:00
author: injae Kim
categories: Dev
# tags:	
cover:  "https://images.unsplash.com/photo-1480694313141-fce5e697ee25?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1050&q=80"
---

## 싱글톤 패턴이 필요한 이유와 실제 서비스에 적용까지

---

안녕하세요!

이 글에선 싱글톤 패턴이 필요한 이유와 구현 방법, 실제 서비스에 적용해 본 사례에 대해 공유하고자 합니다.

싱글톤 패턴은 **소프트웨어 디자인 패턴** 중 하나 인데요, 이 글을 통해 싱글톤 패턴이 필요한 이유에 대한 이해와 더 나아가 여러분들이 디자인 패턴에 대해서도 관심이 생기셨으면 하는 마음에 글을 작성하게 되었습니다.

싱글톤 패턴에 대해서 바로 알아보겠습니다!

<br/>

<br/>

## 쉽게 이해하는 싱글톤 패턴이 필요한 이유

---

| ![](https://injae-kim.github.io/assets/Dev/2020-08-06-singleton-pattern-usage/슬라이드1.JPG) |
| :----------------------------------------------------------: |
|   사무실에서 프린트 1대 를 여러 사람이 함께 사용하는 경우    |

사무실에 있는 1대의 프린터를 여러명이 사용하는 경우를 예시로 들어보겠습니다.

이러한 상황에서 가장 정상적인 프린터 사용 법은, **1대만 존재하는 프린터를 여러 사람이 함께 공유하며 사용하는 방법이죠!**

하지만 위의 사진에서 오른쪽의 경우 처럼, **프린터를 사용하려는 사람들이 프린터를 각자 생성해서 사용하는것은 불가능 합니다.** 우리의 프린터는 단 1대만 존재하기 때문입니다.

정리해보면 다음 과 같습니다.

- 프린터는 단 1개만 존재 한다.
- 여러 사람이 1대의 프린터를 공유하여 사용한다.

이러한 상황은 현실에서도 존재하지만, 우리가 개발하는 프로그램 안에서도 존재할 수 있습니다.

프로그램 내 에서 단 1개만 존재해야 하는 객체가 있으며 이를 프로그램 내부의 여러 부분에서 호출하여 사용하는 경우 이죠.

예를들어, 프로그램 내부에서 발생하는 이벤트들을 스케쥴링 하고 처리하는 객체가 있다고 할 때 프로그램 내부의 모든 이벤트는 하나의 같은 스케쥴링 큐 에 들어가서 처리되어야 합니다. 이벤트 발생 마다 스케쥴링 큐를 따로 따로 생성하면 프로그래머의 의도와 어긋나는 일이죠!

싱글톤 패턴을 적용할 수 있을 만한 상황을 정리해보면 다음과 같습니다.

- 프로그램 내 에서 어떤 객체가 단 1개만 존재해야 한다.
- 프로그램 내부의 여러 부분에서 이 객체를 공유하며 사용한다.

위와 같은 상황에서, 싱글톤 패턴은 **객체가 프로그램 내부에서 단 1개만 생성됨 을 보장**하며 **멀티 스레드에서 이 객체를 공유하며 동시에 접근하는 경우에 발생하는 동시성 문제도 해결**해주는 디자인 패턴 입니다.

싱글톤 패턴이 필요한 이유, 이제 이해 되셨나요?

그렇다면 싱글톤 패턴을 소스코드로 구현하는 방법에 대해 살펴보겠습니다.

<br/>

<br/>

## 싱글톤 패턴의 구현 방법

---

싱글톤 패턴은 하나의 개념 이므로 프로그래밍 언어에 상관 없이 구현이 가능합니다.

저는 안드로이드 애플리케이션을 개발하며 싱글톤 패턴을 적용해봤으므로, Java를 기준으로 설명드리겠습니다.

<br/>

### 싱글톤 패턴의 가장 기본적인 구현

```java
public class Singleton {
    // 단 1개만 존재해야 하는 객체의 인스턴스, static 으로 선언
    private static Singleton instance;

    // private 생성자로 외부에서 객체 생성을 막음
    private Singleton() {}

    // 외부에서는 getInstance() 로 instance 를 반환
    public static Singleton getInstance() {
      // instance 가 null 일 때만 생성
      if (instance == null){
          instance = new Singleton();
      }
      return instance; 
    } 
}
```

싱글톤 패턴의 기본적인 구현 방법 입니다.

**private 생성자**로 외부에서 객체 생성을 막으며, **getInstance()** 함수 로만 instance 에 접근이 가능하게 만듭니다.

instance 는 **static 변수**로 선언되며 getInstance() 가 최초로 호출 될 때 에는 null 이므로 한 번만 생성하고, 이후에는 생성해놨던 instance 를 반환합니다.

따라서 다른 함수에서 getInstance() 를 여러 번 호출 하더라도 **단 하나의 동일한 instance** 만을 반환 해 줍니다.

마치 앞에서 살펴봤던 예시 처럼 여러명이 하나의 프린터를 사용하는 상황과 동일한 구현인 것 이죠!

**하지만, 위의 코드는 멀티 스레드 환경에서 Thread-Safe 를 보장해주지 않습니다.**

두 개의 스레드에서 동시에 getInstance() 를 호출 한경우 에 `if (instance == null)` 조건문에 동시에 도달하게 되어 instance 를 두번 생성할 수 도 있기 떄문이죠!

이를 해결하기 위해, Java 에서 스레드 동기화를 지원하는 `Synchronzied` 를 사용할 수 있습니다.

<br/>

### Synchronized 를 사용하여 Thread-Safe 를 보장하는 싱글톤 패턴

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    // synchronzied 스레드 동기화를 사용하며 멀티 스레드 환경에서의 동시성 문제 해결
    public static synchronzied Singleton getInstance() {
      if(instance  == null) {
         instance  = new Singleton();
      }
      return instance;
    }
}
  
```

위처럼, 여러개의 스레드가 동시에 호출 할 수 있는 getInstance() 함수에 `Synchronzied` 를 적용하여 멀티 스레드 환경에서 getInstance() 함수 호출 시에 발생할 수 있는 동시성 문제를 해결 할 수 있습니다.

과연 이 형태가 완벽한 싱글톤 패턴 일 까요?

 `Synchronzied` 의 가장 큰 단점은, `Thread-Safe` 를 보장하기 위해 속도를 포기했다는 점 입니다.

 `Synchronzied` 를 적용하면 보통 적용하지 않았을 때 보다 약 100배 정도 속도가 느려지는 단점이 있다고 알려져 있습니다. 물론 정확한 수치는 직접 측정해 봐야 알겠지만, 어쩄든 성능저하가 있는 것은 확실하죠.

조금 더 성능을 개선할 순 없을까요?

<br/>

### Double Check를 사용하여 Thread-Safe 를 보장하는 싱글톤 패턴

```java
public class Singleton {
    private volatile static Singleton instance;

    private Sigleton() {}
	
    // Double check
    public Singleton getInstance() {
     // instance 가 null 인 경우
     if(instance == null) {
         // synchronized 를 적용한 후, 한번 더 instance 가 null 인지를 체크
         synchronized(Singleton.class) {
            if(instance == null) {
               instance = new Singleton(); 
            }
         }
      }
      return uniqueInstance;
    }
}
```

`Double Check` 의 아이디어는, getInstace() 함수 전체에 `Synchronzied` 를 적용하는 것을 개선하기 위해 `Synchronzied` 를 사용하지 않고 instance 가 null 인지를 확인한 후, null 이라면 이때서야 `Synchronzied` 를 적용하여 스레드 동기화를 한 후 instance 가 null 인지 다시한번 체크하는 방법 입니다.

처음부터 `Synchronzied` 를 적용하기 보다 느리게, 게으르게 `Synchronzied` 를 적용하여 한번 더 체크하는 것 이죠!

이러한 아이디어는 `Lazy ~` 라는 키워드로 컴퓨터 공학의 다양한 분야에 적용되는 성능 개선 방식입니다.

굳이 처음부터 정직하게 스레드를 동기화 하여 확인할 필요는 없는 것 이죠! instance 가 null 이라면 그때 가서`Synchronzied` 를 적용한 후에 게으르게 한번 더 확인하면 되니까요.

이처럼 Lazy 한 Double Check 방식으로 `Synchronzied` 의 속도 저하를 어느정도 개선할 수 있습니다.

<br/>

### LazyHolder 를 사용하는 싱글톤 패턴

```java
public class Singleton {

    private Singleton() {}
	
    // private static inner class 인 LazyHolder
    private static class LazyHolder() {
        // LazyHolder 클래스 초기화 과정에서 JVM 이 Thread-Safe 하게 instance 를 생성
        private static final Singleton instance = new Singleton();
    }

    // LazyHolder 의 instance 에 접근하여 반환
    public static Singleton getInstance() {
        return LazyHolder.instance;
    }
    
}
```

Java 진영에서 가장 많이 사용되는 `LazyHolder` 를 사용하는 싱글톤 패턴 입니다.

private static class 인 `LazyHolder` 안에 instace 를 final 로 선언하는 방법인데요,

**JVM (Java Virtual Machine) 의 클래스의 초기화 과정에서 원자성을 보장하는 원리를 이용하는 방식입니다.**

getInstance() 가 호출되면 LazyHolder의 instance 변수에 접근하는데요, 이때 LazyHolder 가 static class 이기 때문에 클래스의 초기화 과정이 이루어 집니다.

LazyHolder 클래스가 초기화 되면서 instance 객체의 생성도 이루어 지는데요, JVM 은 이러한 클래스 초기화 과정에서 원자성을 보장합니다.

따라서, final 로 선언한 instance 는 getInstace() 호출 시 LazyHolder 클래스의 초기화가 이루어 지면서 원자성이 보장된 상태로 단 한번 생성되고, final 변수 이므로 이후로 다시 instance 가 할당되는 것 또한 막을 수 있습니다.

이러한 방법에 장점은 `Synchronzied` 를 사용하지 않아도 JVM 자체가 보장하는 원자성을 사용하여 Thread-Safe 하게 싱글톤 패턴을 구현할 수 있는 것 입니다.

[위키피디아](https://en.wikipedia.org/wiki/Initialization-on-demand_holder_idiom) 를 참고하면 JVM 의 클래스 초기화 과정에서 원자성을 보장한다는 내용을 조금 더 자세히 살펴볼 수 있습니다.

> Since the class initialization phase is guaranteed by the JLS to be sequential, i.e., non-concurrent, no further synchronization is required in the static `getInstance` method during loading and initialization.

Java Language Specification 에 명시된 대로 JVM 의 클래스 초기화 과정이 sequential, non-concurrent 하므로, 추가적인 `Synchronzied` 를 사용하여 스레드 동기화를 더 할 필요가 없다는 걸 알 수 있습니다!

**LazyHolder 는 JVM 자체의 특성을 최대한 이끌어내어 성능저하를 막는 나름대로 우아한 방식이라고 할 수 있습니다.**

<br/>

여기까지 싱글톤 패턴의 기본적인 구현 부터 멀티 스레드 환경에서 Thread-Safe 를 보장하는 방법, 개선 방법 까지에 대해서 알아보았습니다.

이제 제가 싱글톤 패턴을 실제로 서비스를 개발하면서 적용했던 사례에 대해서 말씀드리겠습니다.

<br/>

<br/>

## 실제 서비스에 적용해 보기

---

청각장애인이 운행하는 [고요한 택시](http://www.goyohantaxi.com/) 애플리케이션을 개발하면서 실제 서비스의 소스코드에 싱글톤 패턴을 적용해 볼 기회가 있었습니다.

| ![](https://injae-kim.github.io/assets/Dev/2020-08-06-singleton-pattern-usage/슬라이드2.jpg) | ![](https://injae-kim.github.io/assets/Dev/2020-08-06-singleton-pattern-usage/슬라이드3.jpg) |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|           청각장애인 기사님이 운행하는 고요한 택시           |            택시 내의 의사소통을 돕는 애플리케이션            |

고요한 택시는 청각장애인 분들이 택시기사로 운행할 수 있도록 택시 내의 승객과 의사소통을 돕는 애플리케이션 입니다.

고요한 택시 애플리케이션 내 에는 결제 요청이나 메시지 전송 등 **특정 버튼 클릭 시 택시 탑승객 에게 안내 음성을 출력**하는 기능이 있는데요, 여기서 **안내 음성을 출력하고 관리하는 클래스**를 싱글톤 패턴으로 구현하였습니다.

<br/>

| ![](https://injae-kim.github.io/assets/Dev/2020-08-06-singleton-pattern-usage/슬라이드4.PNG) |
| :----------------------------------------------------------: |
| 애플리케이션 내부의 특정 버튼 클릭 시 정해진 안내 음성이 출력 되어야 함 |

위의 사진은 고요한 택시 애플리케이션 의 여러 화면 중 하나 입니다.

좌측의 `목적지 근처`, `결제 방법 요청` 등의 버튼 을 클릭하게 되면 탑승객 에게 해당 버튼에 대한 안내 음성을 출력 하도록 구현되어 있습니다.

**이 때, 안내 음성은 애플리케이션 내부에서 항상 한번에 하나의 음성만 출력되어야 하며 동시에 여러개의 음성이 겹처서 출력되지 않게 구현해야 했습니다.**

여기에 싱글톤 패턴을 적용한다면 안내 음성을 관리하고 출력하는 객체가 프로그램 내부에서 단 하나만 존재한다는 것을 보장할 수 있겠죠!

<br/>

| ![](https://injae-kim.github.io/assets/Dev/2020-08-06-singleton-pattern-usage/슬라이드5.JPG) |
| :----------------------------------------------------------: |
| 버튼 클릭 시 안내 음성 출력 로직 예시,<br />음성이 겹쳐서 여러개가 출력되면 안되며, 한번에 하나의 음성만 출력되어야 함 |

안내 음성 출력 로직을 조금더 자세히 설명하면 위의 사진과 같습니다.

기존에 음성이 출력 중인 상황에서 다른 버튼을 클릭하면 기존에 출력 되던 안내 음성을 정지하고, 클릭한 버튼에 해당하는 안내 음성을 다시 출력해야 하는 로직 입니다.

안내 음성 출력 시 마다 음성 출력 객체를 생성한다면 안내 음성이 겹쳐서 여러개가 한번에 출력 될 수도 있는 상황이죠.

따라서 싱글톤 패턴을 적용하여 프로그램 내부에서 안내 음성을 출력하고 관리하는 객체는 단 하나만 존재하는 것 을 보장하고, 이를 통해 안내 음성을 한번에 한개만 출력하는 로직을 구현할 수 있었습니다.

이를 구현한 소스코드 예시는 다음과 같습니다.

```java
// 안내 음성 출력을 관리하는 클래스, Lazy Holder 싱글톤 패턴으로 구현
public class MediaPlayerUtil {

    private MediaPlayerUtil() {}
	
    // 싱글톤 패턴을 위한 Lazy Holder 클래스
    private static class LazyHolder() {
      private static final MediaPlayerUtil instance = new MediaPlayerUtil();
    }

    public static MediaPlayerUtil getInstance() {
      return LazyHolder.instance;
    }
    
    // 외부에서는 MediaPlayerUtil.start(media); 로 안내 음성을 출력
    public static void start(int media){
      MediaPlayerUtil.getInstance().start(media);
    }
}
```

LazyHolder 기반의 싱글톤 패턴을 사용하여 Thread-Safe하게 안내 음성을 출력할 수 있는 `MediaPlayerUtil` 을 구현하였습니다.

외부에서는 안내 음성을 출력하기 위해서 `MediaPlayerUtil.start(media)` 처럼 함수를 호출하면 됩니다.

싱글톤 패턴을 적용하였기 때문에 항상 안내 음성을 한번에 단 하나만 재생 할 수 있었습니다.

<br/>

지금까지 싱글톤 패턴을 제가 실제 서비스를 개발하면서 적용한 사례에 대해서 알아보았습니다.

안내 음성을 출력하는 경우에 요구되는 로직이 싱글톤 패턴과 잘 맞는 부분이 많아서 자연스럽게 싱글톤 패턴을 적용할 수 있었습니다.

실제로, 안내 음성 출력 클래스를 싱글톤 패턴으로 구현한 이후로 **안내 음성 출력에 관해선 1년 넘게 서비스를 운영하면서 단 한번의 버그도 발생하지 않았습니다!**

안내 음성 출력 클래스를 구현하면서 개인적으로는 디자인 패턴에 대해서 공부해보는 계기가 되었고, 버그 없이 잘 작동하는 기능을 보면서 개인적으로 만족스러웠던 경험이였습니다.

<br/>

<br/>

## 결론: 디자인 패턴은 적절한 상황에서 사용할 때 빛난다

---

디자인 패턴 중 하나인 싱글톤 패턴이 필요한 이유와 구현 방법, 실제 서비스에 적용 사례 까지를 이 글에서 알아보았습니다.

이 글을 작성한 이유는 여러 개발자 분들이 디자인 패턴이라는 유용한 분야 도 있구나 라고 관심이 생겼으면 해서 작성하게 되었습니다.

하지만, 디자인 패턴도 만능이 아닙니다.

수십개가 넘는 디자인 패턴을 하나하나 외우고 모든 소스코드를 디자인 패턴에 완벽하게 맞춰서 구현 할 필요는 없는 것 이죠!

다만 어떤 로직을 구현해야 할 때 어렴풋이 읽었던 디자인 패턴 중 하나가 기억난다면, 그 정도로도 충분하다고 생각합니다.

디자인 패턴은 특정 상황에서 적절한 로직을 구현하기 위한 하나의 개념일 뿐이지만 수십년간 개발을 해오면서 경험을 축적하신 선배 프로그래머 분들의 시행착오와 앞으로의 소스코드의 확장성에 대한 고려들이 녹아있기 떄문이죠.

> 디자인 패턴은 원래 부터 존재했고, 다만 발견 했 을 뿐이다.

개인적으로 디자인 패턴 관련해서 가장 기억에 남는 말 입니다.

디자인 패턴을 코드를 통째로 외우듯이 공부하시기 보다는 어떤 디자인 패턴이 필요한 이유에 대해 이해하는 것 이 중요하다고 생각합니다.

읽어주셔서 감사합니다.

<br/>

<br/>

## Reference

---

- [모던 C++ 디자인 패턴 책](http://www.yes24.com/Product/Goods/71969505)
- [위키피디아 - LazyHolder Singleton](https://en.wikipedia.org/wiki/Initialization-on-demand_holder_idiom)
- [자바 싱글톤 패턴 (Java Singleton Pattern)](https://gofnrk.tistory.com/63)
- [멀티스레드 환경에서의 싱글톤 패턴](https://jungwoon.github.io/java/2019/08/11/Singleton-Pattern-with-Multi-Thread/)

<br/>

<br/>