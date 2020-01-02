---
description: 제2장 아키텍쳐 개요
---

# 2. 아키텍쳐 개요

## 들어가기 앞서

* 아키텍처를 설계할 때 출현하는 전형적인 영역은 '표현', '응용', '도메인', '인프라스트럭처' 이렇게 4가지 영역이다.

## 네 개의 영역

### 표현 영역\(UI 영역\)

* **표현 영역\(UI 영역\)은 사용자의 요청을 받아 응용 영역에 전달하고 응용 영역의 처리결과를 다시 사용자에게 보여주는 역할을 한다.**
* 웹 에플리케이션 개발 시 사용하는 스프링 MVC 프레임워크도 표현 영역을 위한 기술에 해당된다.
* 표현 영역의 사용자는 사람일 수도 있고, REST API를 호출하는 외부 시스템일 수도있다.
* 표현 영역은 HTTP 요청을 응용 영역이 필요한 형태로 변환하여 전달하고 응용 영역의 응답을 다시 필요한 HTTP 형으로 만들어 웹 브라우저에게 리턴한다.

### 응용 영역

* **표현 영역을 통해 사용자의 요청을 전달받은 응용 영역은 사용자에게 제공할 기능을 구현한다.**
* **응용 영역은 도메인 모델을 이용해서 사용자에게 제공할 기능을 구현한다.**

```java
public class CancleOrderService {
    @Transactional
    public void cancleOrder(String orderId) {
        Order order = findOrderById(orederId);
        if (order == null) throw new OrderNotFoundException(orderId);
        order.cancel();
    }
}
```

* 위처럼 응용 서비스는 로직을 직접 수정하기보다 도메인 모델에 로직 수행을 위임한다.
* Order라는 도메인에 cancle\(\)를 통하여 주문을 취소하고 있다. 



### 도메인 영역

* **도메인 영역은 도메인 모델을 구현한다.**
  * 위에서 보았던 Order.class와 같은 도메인 모델을 구성한다.
* 도메인 모델은 도메인의 핵심 로직을 구현한다.

### 인프라스트럭처 영역

* **인프라스트럭처 영역은 구현 기술에 대한 것을 다룬다.**
* **위에 설명 했던 것처럼 도메인 영역, 응용 영역, 표현 영역은 구현 기술을 사용한 코드를 직접 만들지 않는다.** 
  * 여기서 구현 기술이란 아래와 같은 예시를 뜻한다.
  * 따라서 응용 영역에서 DB에 보관된 데이터가 필요하면 인프라스트럭처 영역의 DB 모듈을 이용하고, 외부에 메일을 발송해야할 경우 인프라스트럭처 SMTP 모듈을 이용하여 메일을 발송한다.
* RDBMS 연동을 처리하거나 메세징 큐에 메세지를 전송하거나 수신하는 기능을 구현하고, 몽고DB나 HBase를 사용해서 데이터베이스 연동을 처리한다.
* STMP를 통한 메일 발송 기능이나 HTTP 클라이언트를 이용해서 REST API를 호출 하는것도 처리한다.
* 그림 2.3
  * 카프카 - 분산 메세징 시스템으로 대용량 실시간 로그처리에 특화된 아키텍처 설계로 이루어짐.
  * SMPT서버 - 전자메일전송을 위한 표준 프로토콜.

## 계층 구조 아키텍처

* **네 영역을 구성할 때는 표현 영역 -&gt; 응용 영역 -&gt; 도메인 영역 -&gt; 인프라스트럭처 영역으로 연결 된 계층 구조를 사용한다.** 
  * 사용에 따라 응용 영역과 도메인 영역을 합치기도 하지만 전체적으로는 위와 같은 구조를 사용한다.
* 계층 구조는 특성상 하위에서 상위로 의존하고 상위에서 하위로 의존하지 않는다.
* 구현의 편리함을 위하여 계층 구조를 유연하게 적용한다.

  * 예를들어 외부시스템 연동을 위하여 응용 영역이 DB연계를 위해 인프라스트럭처                                              계층에 의존 하기도 한다. 중요한 점은 다른 3가지 계층들이 상세한 구현 기술을 다루는 

          인프라스트럭처 계층에 종속 된다는 점이다. \(41p 그림 2.5\)

  * 42p 예시코드와 그림 2.6을 보면 응용 계층이 인프라스트럭처 계층을 의존 할 경우에 

          두가지 문제점을 보여주고 있다.

  * 1. 응용 계층의 로직만으로는 테스트가 불가능하다. 인프라스트럭처 모듈이 정상 작동 해야한다.
    2. 상위 계층인 인프라스트럭처 모듈을 수정하면 하위 응용 계층 로직도 수정해야하면서 기능 확장이 어려워 진다.




