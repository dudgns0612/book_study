---
description: 제 4장 클래스와 인터페이스
---

# 4. 클래스와 인터페이스

### 아이템 15 - 클래스와 멤버의 접근 권한을 최소화하라.

* 잘 설계된 컴포넌트와 그렇지 않은 컴포넌트의 가장 큰 차이는 클래스 내부 데이터와 내부 구현 정보를 외부 컴포넌트로부터 얼마나 잘 숨겼는가이다.
* 오직 API를 통해서만 다른 컴포넌트와 소통하며 서로의 내부 동작 방식에는 개의치 않고, 정보 은닉 혹은 캡슐화라고 하는 소프트웨어 설계의 근간이 되는 원리다.

#### 정보 은닉에 장점

* 시스템 개발 속도를 높인다. 여러 컴포넌트를 병렬로 개발할 수 있기 때문이다.
* 시스템 관리 비용을 낮춘다. 각 컴포넌트를 더 빨리 파악하여 디버깅할 수 있고, 다른 컴포넌트로 교체하는 부담도 적다.
* 정보 닉 자체가 성능을 높여주지는 않지만, 성능 최적화에 도움을 준다.
*  소프트웨어 재사용성을 높인다. 외부에 거의 의존하지 않고 독자적으로 동작할 수 있는 컴포넌트라면 그 컴포넌트와 함께 개발되지 않은 낯선 환경에서도 유용하게 쓰일 가능성이 크다.
* 큰 시스템을 제작하는 난이도를 낮춰준다.

#### 컴포넌트 설계 기본 원칙

* 모든 클래스와 멤버의 접근성을 가능한 줄여야한다.
* public 클래스의 인스턴스 필드는 되도록 public이 아니여야 한다.
  * 불변식을 보장할 수 없게 된다.
  * 상수라면 final로 선언하여 사용해도 무방다.
* 클래스에서 public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공해서는 안된다.

  * 클라이언트에서 배열 내용을 수정할 수 있게 된다.  아래 두 코드처럼 처리한다.

  ```java
  // private로 만들고 public 불변 리스트 추가
  private static final Thing[] PRIVATE_VALUES = {...};
  public static final List<Thing> VALUES = 
      Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));

  // private로 만들고 배열 복사본 반
  private static final Thing[] PRIVATE_VALUES = {...};
  public static final Thing[] values() {
      return PRIVATE_VALUES.clone();
  }
  ```

#### 결론

* 프로그램 요소의 접근성은 가능한 최소한으로 하라.
* public 클래스는 상수용 public static final 필드 외에는 어떠한 public 필드도 가져서는 안된다.

### 아이템 16 - public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라. 

#### 인스턴스 필드들을 모아놓는 클래스 작성

* 아래와 같이 인스스 필드들을 모아놓는 일이외에는 아무 목적도 없는 퇴보한 클래스를 작성할 때 필드는 public이어스는 안된다.

```java
class Point {
    public double x;
    public double y;
}
```

* 이런 클래스는 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다. API를 수정하지 않고는 내부 표현을 바꿀 수 없고, 불변식을 보장할 수 없으며, 외부에서 필드에 접근할 때 부수작업을 수행할 수도 없다.
* 철저한 객체 지향 프로그래머는 이런 클래스를 필드를 모두 private으로 바꾸고 public 접근자\(getter\)를 추가한다.

  ```java
  package effectivejava.chapter4.item16;

  // 코드 16-2 접근자와 변경자(mutator) 메서드를 활용해 데이터를 캡슐화한다. (102쪽)
  class Point {
      private double x;
      private double y;

      public Point(double x, double y) {
          this.x = x;
          this.y = y;
      }

      public double getX() { return x; }
      public double getY() { return y; }

      public void setX(double x) { this.x = x; }
      public void setY(double y) { this.y = y; }
  }
  ```

* public 클래스에서라면 이 방식이 확실히 맞는 방법이다. **패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함**으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.
* public 클래스가 필드를 공개하면 이를 사용하는 클라이언트가 생겨날 것이므로 내부 표현 방식을 마음대로 바꿀 수 없다. \(클라이언트가 필드를 어떤 방식으로 사용하고 있는지 모르기 때문\)

#### package-private 클래스 혹은 private 중첩 클래스

* pakcage-private 클래스 혹은 private 중첩 클래라면 데이터 필드를 노출한다 해도 하등의 문제가 없다.
* 클라이언트 코드가 이 클래스 내부 표현에 묶이기는 하나, 클라이언트도 어차피 이 클래스를 포함하는 패키지 안에서만 동작하는 코드일 뿐이다.

#### 결론

* public 클래스는 절대 가변 필드를 직접 노출해서는 안된다. final로 선언 된 불변 필드라면 노출해도 위험이 덜하지만 완전히 안심할 수는 없다.
* 하지만 pakcage-private 클래스 혹은 private 중첩 클래스라면 필드를 노출하는 편히 나을 때도 있다.

