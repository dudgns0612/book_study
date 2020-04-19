---
description: 제 3장 모든 객체의 공통 메서드
---

# 3. 모든 객체의 공통 메서드

### 아이템 10 - equals는 일반 규약을 지켜 재정의하라.

equals 메서드는 재정의하기 쉬워 보이지만 곳곳에 함정이 도사리고 있어서 자칫하면 안좋은 결과를 초래한다. 문제를 회피하는 가장 쉬운 길은 아예 재정의하지 않는 것이다. 아래와 같은 경우는 재정의하지 않는 것이 좋다.

* **각 인스턴스가 본질적으로 고유하다.** 값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스가 여기 해당한다. 좋은 예로 Thread가 있다.
* **인스턴스의 '논리적 동치성\(logical equality\)'를 검사할 일이 없다.** 예컨대 java.util.regex.Pettern은 equals를 재정의해서 두 Pattern의 인스턴스가 같은 정규표현식을 나타내는지를 검사하는, 즉 논리적 동치성을 검사하는 방법도 있다. 즉 Object의 기본 equals만으로 해결된다.
* **상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.** 대부분의 Set 구현체는 AbstractSet이 구현한 equlas를 상속받아 쓰고, List는 AbstractList, Map은 AbstractMap으로부터 상속받아 그대로 쓴다.
* **클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.**

그렇다면 equals를 재정의해야 할 때는 언제인가? 바로 '객체 식별성\(두 객체가 물리적으로 같은가\)'이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때다. 주로 값 클래스들이 여기 해당한다.

두 값의 객체를 equals로 비교하는 프로그래머는 객체가 같은지가 아니라 값이 같은지를 알고 싶어 할 것이다. equals가 논리적 동치성을 화긴하도록 재정의해두면, 그 인스턴스는 값을 비교하길 원하는 프로그래머의 기대에 부응함과 Map의 키와 Set의 원소로 사용할 수 있게 된다.

값 클래스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스라면 equals를 재정의하지 않아도 된다. 인스턴스가 2개 이상 만들어지지 않으니 논리적 동치성과 객체 식별성이 사실상 똑같기 때문이다.\(ex\) Enum\) equals 메서드를 재정의할 때는 반드시 일반 규약을 따라야 한다. 아래는 Object 명세에 적힌 규약이다.

**equals 메서드는 동치관계를 구현하며, 다음을 만족한다.**

* 반사성\(reflexivity\): null이 아닌 모든 참조 값 x에 대해, x.equals\(x\)는 true이다.
* 대칭성\(symmetry\): null이 아닌 모든 참조 값 x, y에 대해, x.equals\(y\)가 true면 y.equals\(x\)도 true다.
* 추이성\(transitivity\): null이 아닌 모든 참조 값 x, y, z에 대해, x.eqauls\(y\)가 true이고 y.equals\(z\)도 true면 x.equals\(x\)도 true다.
* 일관성\(consistency\): null 이 아닌 모든 참조 값 x, y에 대해 x.equals\(y\)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
* null-아님: null이 아닌 모든 참조 값 x에 대해, x.equals\(null\)은 false다.

equals 메서드가 쓸모 있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 서로 교환할 수 있어야 한다. 아래는 동치관계를 만족시키기 위한 다섯가지 요건이다.

**반사성**은 단순하게 객체는 자기 자신과 같아야 한다는 뜻이다. 이 요건을 어긴 클래스의 인스턴스를 컬렉션에 넣은 다음 contains 메서드를 호출하면 방금 넣은 인스턴스가 없다고 답할 것이다.

대칭성은 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻이다. 반사성 요건과 달리 대칭성 요건은 자칫하면 어길 수 있어 보인다. 대소문자를 구별하지 않는 문자열을 구현한 다음 클래스를 예로 살펴보자.

 

### 아이템 11 - equals를 재정의하려거든 hashCode도 재정의하라.

### 아이템 12 - toString을 항상 재정의하라.

* Object의 기본 toString 메서드가 우리가 작성한 클래스에 적합한 문자열을 반환하는 경우는 거의 없다.
* 보통 PhoneNumber@abbbd처럼 단순히 **클래스\_이름@16진수\_해시코드**를 반환할 뿐이다. 
* toString의 일반 규약에 따르면 '간결하면서 사람이 읽기 쉬운 형태의 유익한 정보'를 반환해야 한다.
* 따라서 모든 하위 클래스에서 이 메서드를 재정의해야한다. toString을 잘 구현한 클래스는 사용하기에 훨씬 좋고, 그 클래스를 사용한 시스템은 디버깅하기 쉽다.
* **toString 메서드는 객체를 println, printf, 문자열 연결 연산자\(+\), assert 구문에 넘길 때, 혹은 디버거가 객체를 출력할 때 자동으로 불린다.**
* 아래와 같이 toString을 제대로 재정의했다면 다음 코드만으로 문제를 진단하기에 충분한 메세지를 남길수 있다.

  ```java
  System.out.println(phoneNumber + "에 연결할 수 없습니다.");
  ```

* **실전에서 toString은 그 객체가 가진 주요 정보 모두를 반환하는 게 좋다.** 하지만 객체가 거대하거나, 객체의 상태가 문자열로 표현하기에 적합하지 않다면 무리가 있다. 이런 상황이라면 "맨하튼 거주자 전화번호부\(총 1000개\)" 나 "Thread\[main,5,main\]" 같은 요약 정보를 담아야 한다.
* toString을 구현할 때면 반환값의 포맷을 문서화할지 정해야한다. 이것은 아주 중요한 선택이다.
* 포맷을 명시하기로 했다면, 명시한 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 정적 팩터리나 생성자를 함께 제공해주면 좋다. 좋은예로 BigInteger, BigDecimal과 대부분의 기본 타입 클래스가 여기 해당한다.
* 하지만 단점도 있다. 포맷을 한번 명시하면 평생 그 포맷에 얽매이게되고 향후 릴리즈에서 포맷을 바꾼다면 사용하던 코드들과 데이터들은 엉망이 될 것이다.
* **포맷을 명시하든 아니든 우리의 의도는 명확히 밝혀야 한다.**

  ```java
  // 포맷을 명시하기로 한 경우
  /**
   * 이 전화번호의 문자열 표현을 반환한다.
   * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
   * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
   * 각각의 대문자는 10진수 숫자 하나를 나타낸다.
   *
   * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
   * 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
   * 전화번호의 마지막 네 문자는 "0123"이 된다.
   */
  @Override public String toString() {
      return String.format("%03d-%03d-%04d",
              areaCode, prefix, lineNum);
  }

  // 포맷을 명시하지 않기로 한 경우
  /**
   * 상세형식은 정해지지 않았으며 향후 변경될 수 있다. 
   */
  @Override public String toString() {...}
  ```

* **포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어올수 있는 API를 제공하자.**
* 위 예제로 보면 PhoneNumber 클래스는 지역코드, 프리픽스, 가입자번호 접근자를 제공해야 한다.
* 정적 유틸리티 클래스나 대부분의 열거 타입도 자바가 이미 완벽한 toString을 제공하니 따로 재정의하지 않아도 된다.
* 하지만 하위 클래스들이 공유해야 할 문자열 표현이 있는 추상클래스라면 toString을 재정의해줘야 한다.

#### 결론

* 모든 구체 클래스에서 Object의 toString을 재정의하자. 상위 클래스에서 이미 알맞게 정의한 경우는 예외이다.
* toString을 재정의한 클래스는 사용하기도 즐겁고, 클래스를 사용한 시스템을 디버깅하기 쉽게 해준다.
* toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다.

### 아이템 13 - clone 재정의는 주의해서 진행하라.

* Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스다.
* Cloneable은 clone 메서드를 선언한 곳이 Cloneable이 아닌 Object이며, 그마저도 protected이기 때문에 Cloneable을 구현하는 것 만으로 외부객체에서 clone 메서드를 호출할 수 없다.
* Cloneable 인터페이스는 Object의 protected 메서드인 clone의 동작 방식을 결정한다. clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환한다.
* 이 인터페이스를 implements 하지 않은 클래스를 복사할 경우 CloneNotSupportedException을 던진다.

#### clone 메서드 명세서

* 이 객체의 복사본을 생성해 반환한다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다. 일반적인 의도는 다음과 같다. 어떤 객체 x에 대해 다음 식은 참이다.

  * x.clone\(\) != x
  * x.clone\(\).getClass\(\) == x.getClass\(\)
  * x.clone\(\).equals\(x\)  / 항상 만족해야 하는건 아니지만 일반적으로 참이다.

* 관례상, 이 메서드가 반환하는 객체는 super.clone을 호출해 얻어야 한다.
* 관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

#### 가변 상태를 참조하지 않는 클래스

* 제대로 동작하는 clone 메서드를 가진 상위 클래스를 상속해 Cloneable을 구현하고 싶다고 했을 때, 먼저 super.clone을 호출한다.
* 모든타입이 기본 타입이거나 불변 객체를 참조한다면 이 객체는 완벽히 우리가 원하던 상태일 것이다.
* 불변 클래스는 굳이 clone 메서드를 제공하지 않는 게 좋다. 이 점을 고려하여 불변 클래스의 clone 메서드는 아래와 같이 구현할 수 있다.

  ```java
  // 아이템10의 PhoneNumber 클래
  @Override
  public PhoneNumber clone() {
      try {
          return (PhoneNumber)super.clone();
      } catch (CloneNotSupportExceoption e) {
          throw new AssertionError(); // 일어날 수 없는 일
      }
  }
  ```

* 위 메서드가 동작하게 하려면 PhoneNumber 클래스에 Cloneable을 구현해야한다. Object에 clone 메서드는 Object를 반환하지만 PhoneNumber의 clone 메서드는 PhoneNumber을 반환하게 했다.
* 이 방식으로 클라이언트에서 형변환하지 않아도 되게끔 하자.

#### 가변 상태를 참조하는 클래스 

* 위 앞서 보았던 구현이 클래스가 가변 객체를 참조 하는 순간 재앙으로 변한다. 아이템7에서 소개한 Stack 클래스를 예로 들어보자.

  ```java
  public class Stack {
      private Object[] elements;
      private int size = 0;
      private static final int DEFAULT_INITIAL_CAPACITY = 16;

      public Stack() {
          elements = new Object[DEFAULT_INITIAL_CAPACITY];
      }

      public void push(Object e) {
          ensureCapacity();
          elements[size++] = e;
      }

      public Object pop() {
          if (size == 0)
              throw new EmptyStackException();
          return elements[--size];
      }

      private void ensureCapacity() {
          if (elements.length == size)
              elements = Arrays.copyOf(elements, 2 * size + 1);
      }
  }
  ```

* clone이 단순히 super.clone의 결과를 그대로 반환한다면 반환된 Stack 인스턴스의 size 필드는 올바른 값을 갖겠지만, elements 필드는 원본 Stack 인스턴스와 똑같은 배열을 참조할 것이다.
* 따라서 복제본이나 원본을 수정하게 되면 다른하나도 수정되어 불변식을 해친다는 이야기다.
* **clone 메서드는 사실상 생성자와 같은 효과를 낸다. 즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야한다.**
* 따라서 clone 메서드를 제대로 동작하려면 스택 내부 정보를 복사해야한다. 가장 쉬운 방법은 elements 배열의 clone 을 재귀적으로 호출해 주는 것이다.

  ```java
  @Override public Stack clone() {
      try {
          Stack result = (Stack) super.clone();
          result.elements = elements.clone();
          return result;
      } catch (CloneNotSupportedException e) {
          throw new AssertionError();
      }
  }
  ```

* clone을 재귀적으로 호출하는 것만으로 충분하지 않을 때도 있다. 이번에는 해시테이블용 clone 메서드를 생각해 보자. 해시테이블 내부는 버킷들의 배열이고, 각 버킷은 키-값 쌍을 담는 연결 리스트의 첫 번째 엔트리를 참조한다.
* 성능을 위해 java.util.LinkedList 대신 직접 구현한 경량 연결 리스트를 사용하겠다.

  ```java
  public class HashTable implements Cloneable {
      private Entry[] buckets = ...;
    
      private static class Entry {
          final Object key;
          Object value;
          Entry next;
        
          Entry(Object key, Object value, Entry next) {
              this.key = key;
              this.value = value;
              this.next = next;
          }
      }
      ... // 나머지 코드는 생
  }
  ```

* Stack에서처럼 단순히 버킷 배열의 clone을 호출하게 되면 이 배열은 원본과 같은 연결리스트를 참조하여 원본과 복제본 모두 예기치 않게 동작할 가능성이 생긴다.
* 따라서 Hashtable에 Entry를 deepCopy를 할 수 있도록 메서드를 지원하고 연결리스트들을 재귀적으로 호출해서 연결리스트 전체를 복사해야한다.

#### clone 사용에 대한 주의점

* 생성자에서는 재정의될 수 있는 메서드를 호출하지 않아야 하는데 clone 메서드도 마친가지다.
  * 만약 clone이 하위 클래스에서 재정의한 메서드를 호출하면, 하위 클래스는 복제 과정에서 자신의 상태를 교정할 기회를 잃게 되어 온본과 복제본의 상태가 달라질 가능성이 크다.
* 상속해서 쓰기 위한 클래스 설계 방식 두 가지\(아이템19\) 중 어느 쪽에서든, 상속용 클래스는 Cloneable을 구현해서는 안된다.
  * 첫 번째, 제대로 작동하는 clone 메서드를 구현해 protected로 두고 CloneNotSupportedException을 던질 수도 이다고 선언하는 것이다.
  * 두번 째, clone을 동작하지 않게 구현해 놓고 하위 클래스에서 재정의하지 못하게 할 수도 있다.

    ```java
    @Override
    protected final Object clone() throws CloneNotSupportedException {
        throw new CloneNotSupportedException();
    }
    ```

#### 복사 생성자와 복사 팩터리 사용

* 위 처럼 모든 작업이  필요한가? 복잡한 경우는 드물다. 꼭 Cloneable을 구현해야 한다면 위처럼 해야하지만 그렇지 않다면 **복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공할 수 있다.**

  ```java
  // 복사 생성자
  public Yum(Yum yum) {
      ...
  }

  // 복사 팩터리
  public static Yum newInstance(Yum) {
      ...
  }
  ```

* 복사 생성자와 복사 팩터리는 Cloneable/ clone 방식보다 나은면이 많다. 위에 설명했던 여러 문제들에 대해 제약 받지 않는다.

#### 결론

* Cloneable/clone은 새로운 인터페이스, 클래스 생성 문제, 형 변,성능 최적화 관점에서 문제가 있다. 따라서 이런 문제에 대해서 문제가 없을 시에만 드물게 허용해야한다.
* 따라서 **복제기능은 복사생성자와 복사 팩터리를 이용하는게 가장 나은 방법이다.** 

### 아이템 14 - Comparable을 구현할지 고려하라.

### 아이템 15 - 클래스와 멤버의 접근 권한을 최소화하라.

### 아이템 16 - public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라. 





