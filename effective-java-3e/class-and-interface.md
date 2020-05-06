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

### 아이템 17 - 변경 가능성을 최소화하라.

#### 불변 클래스란 인스턴스 내부 값을 수정할 수 없는 클래스며 불변 클래스느 가변 클래스보다 설계하고 구현하고 사용하기 쉽고 오류가 생길 여지가 적다. 

아래는 클래스를 불변으로 만들기 위한 다섯 가지 규칙을 따른다.

* 객체의 상태를 변경하는 메서드\(변경자\)를 제공하지 않는다.
* 클래스를 확장할 수 없도록 한다. 하위 클래스에서 부주의하게 나쁜의도로 객체의 상태를 변하게 만다는 사태를 막아준다.
* 모든 필드를 final로 선언한다.
* 모든 필드를 private로 선언한다. 필드가 참조하는 가변 객체를 클라이언트에서 직접 접근해 수정하는 일을 막아준다.
* 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.

```java
 public final class Complex {
    private final double re;
    private final double im;

    public static final Complex ZERO = new Complex(0, 0);
    public static final Complex ONE  = new Complex(1, 0);
    public static final Complex I    = new Complex(0, 1);

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    // 코드 17-2 정적 팩터리(private 생성자와 함께 사용해야 한다.) (110-111쪽)
    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        // == 대신 compare를 사용하는 이유는 63쪽을 확인하라.
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

위 클래스는 복소수를 표현한다. Object의 메서드 몇 개를 재정의하고, 실수부와 하수부  값을 반환하는 접근자 메서드와 사칙연산 메서드를 정의했다. 이 사칙연산 메서드들이 인스턴스 자신은 수정하지 않고 새로운 Complex 인스턴스를 반환한다. 이처럼 피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴을 함수형 프로그래밍이라 한다.

* **불변 객체는 단순하다.** 불변 객체는 생성된 시점의 상태를 파괴될 때까지 그대로 간직한다.
* **불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요가 없다.** 여러 스레드가 동시에 사용해도 절대 훼손되지 않는다.
* **불변 객체는 안심하고 공유할 수 있다.** 불변 클래스라면 한번 만든 인스턴스를 최대한 재활용하기를 권한다. 자주 쓰이는 값들을 상수\(public static final\)로 제공하는 것이다.
* **불변 객체는 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다.**
* **객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.** 값이 바뀌지 않는 구성요소들로 이뤄진 객체라면 그 구조가 아무리 복잡하더라도 불변식을 유지하기 훨씬 수월하기 때문이다.
* **불변 객체는 그 자체로 실패 원자성\(예외가 발생해도 객체는 여전히 유효한 상\)을 제공한다.**  상태가 절대 변하지 않으니 잠깐이라도 불일치 상태에 빠질 가능성이 없다.
* **불변 클래스에도 단점은 있다. 값이 다르면 반드시 독립된 객체로 만들어야 한다는 것이다.** 값이 가지수가 많다면 이들을 모두 만드는 데 큰 비용을 치러야 한다. 

  _**해결방안**_

  * 다단계 연산을 미리 예측하여 기본 기능으로 제공하는 방법이다. 여러 다단계 연산을 기본으로 제공한다면 더 이상 각 단계마다 객체를 생성하지 않아도 된다. 가변 동반 클래스를 package-private로 만들어 다단계 연산 속도를 높여준다. 예\)String 클래스의 가변 동반 클래스 StringBuilder

#### 불변 클래스를 만드는 또 다른 설계 방법

* 가장 쉬운 방법은 클래스를  final로 선언하는 것이지만, 더 유연한 방법은 생성자를 private나 package-private로 만들고 public 정적 팩터리를 제공하는 방법이다.

#### 결

* getter가 있다고 무조건 setter를 만들지 말자. **클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.**
* **불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.** 객체가 가질수 있는 상태의 수를 줄이면 그 객체를 예측하기도 쉬워지고 오류가 생길 가능성이 줄어든다. 꼭 변경해야할 필드를 뺀 나머지 모두를 final로 선언하자.
* **다른 합당한 이유가 없다면 모든 필드는 private final이어야 한다.**
* **생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.** 확실한 이유가 없다면 생성자와 정적 팩터리 외에는 어떤 초기화 메서드도 public 으로 제공해서는 안된다.

### 아이템 18 - 상속보다는 컴포지션을 사용하라.

### 아이템 19 - 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라.

#### 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야한다.

* 클래스의 API로 공개된 메서드에서 클래스 자신의 또 다른 메서드를 호출할 수도 있다. 그런데 마침 호출되는 메서드가 재정의 가능 메서드라면 그 사실을 호출하는 메서드의 API 설명에 직시해야 한다.
* 재정의 가능 메서드를 호출할 수 있는 모든 상황을 문서로 남겨야한다.
* API 문서의 메서드 설명 끝에서 종종 'Implementation Requirements'로 시작하는 절을 볼 수 있는데, 그 메서드의 내부 동작 방식을 설명하는 곳이다. 이 절은 메서드 주석에 @implSpec 태그를 붙여주면 자바독 도구가 생성해 준다.
* 내부 매커니즘을 문서로 남기는 것만이 상속을 위한 설계의 전부는 아니다. **효율적인 하위 클래스를 큰 어려움 없이 만들 수 있게 하려면 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅\(hook\)을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있다.**

#### 어떤 protected 메서드를 노출해야 할까?

안타깝게도 정답은 없다. 하위 클래스를 만들어 시험해보는 것이 최선이다. protected메서드 하나하나가 내부 구현에 해당하므로 그 수는 가능한 적어야 한다.

* **상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 '유일'하다.** 꼭 필요한 protected 멤버를 놓쳤다면 하위 클래스를 작성할 때 그 빈자리가 확연히 드러난다. 하위 클래스 여러개를 만들 때까지 전혀 쓰이지 않는 protected 메서드는 private 였어야 할 가능성이 높다.
* 따라서 **상속용으로 설계한 클래스는 배 전에 반드시 하위 클래스를 만들어 검증해야 한다.**
* **상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.** 

  * 상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행되므로 하위 클래스에서 재정의한 메서드가 하위 클래스의 생성자보다 먼저 호출된다. 이때 그 재정의한 메서드가 하위 클래스의 생성자에서 초기화하는 값에 의존한다면 의도대로 동작하지 않을 것이다. 아래 코드를 보자.

  ```java
  public class Super {
      // 잘못된 예 - 생성자가 재정의 가능 메서드를 호출한다.
      public Super() {
          overrideMe();
      }

      public void overrideMe() {
      }
  }

  public class Sub extends Super{
      // 초기화되지 않은 final 필드. 생성자에서 초기화한다.
      private final Instant instant;

      Sub() {
          instant = Instant.now();
      }

      // 재정의 가능 메서드. 상위 클래스의 생성자가 호출한다.
      @Override public void overrideMe() {
          System.out.println(instant);
      }

      public static void main(String[] args) {
          Sub sub = new Sub();
          sub.overrideMe();
      }
  }
  ```

  위 프로그램이 instant가 두 번 출력할거라 기대하지만, 첫 번째는 null을 출력한다. 상위 클래스의 생성자는 하위 클래스의 생성자가 인스턴스 필드를 초기화 하기도 전에 overrideMe를 호출하기 때문이다.

#### Cloneable과 Serialize 인터페이스는 상속용 설계의 어려움을 더한다.

* clone과 readObject 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안된다.
* clone은 복제본 뿐만아니라 원본 객체에도 피해를 줄 수 있다.
* Serializable을 구현한 상속용 클래스가 readResolve나 writeReplace 메서드를 갖는다면 이 메서드들은 private가 아닌 protected로 선언해야한다. private로 선언하면 하위 클래스에서 무시되기 때문이다.

#### 상속용으로 설계하지 않은 클래스는  상속을 금한다.

상속을 금하는 방법은 두가진데 불변 클래스를 만드는 방법과 동일하다.

* 클래스를 final로 선언한다.
* 모든 생성자를 private나 package-private로 선언하고 public 정적 팩터리를 만들어준다.
* 별개의 이유로 상속을 허용해야 한다면 클래스 내부에 재정의 기능 메서드를 완전이 제거한다.

#### 결론

* 상속용 클래스는 클래스 내부에서 스스로 어떻게 사용하는지 모두 문서로 남겨야한다.
* 다른 이가 효율 좋은 하위 클래스를 만들 수 있도록 일부 메서드를 protected로 제공해야 할 수도 있다.
* 상속용으로 설계하지 않은 클래스는 상속을 금한다.
* 별개의 이유라면 재정의 메서드를 제거하여 상속한다.

### 아이템 20 - 추상 클래스보다는 인터페이스를 우선하라. 

