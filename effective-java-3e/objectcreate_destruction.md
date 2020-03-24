---
description: 제 2장 객체 생성과 파괴
---

# 2. 객체 생성과 파괴

### 아이템 1 - 생성자 대신 정적 팩터리 메서드를 고려하라.

클라이언트\(제공 클래\)가 클래스의 인스턴스를 얻는 전통적인 수단은 public 생성자이다. 하지만 모든 프로그래머가 꼭 알아둬야 할 기법이 하나 더 있다. 클래스는 생성자와 별도로 정적 팩터리 메서드를 제공할 수 있다.

클래스의 인스턴스를 반환하는 단순한 정적 메서드를 말한다. 아래의 코드는 boolean 기본 타입의 박싱 클래스\(래퍼 클래\)를 발췌하는 간단한 예이다. 기본타입인 boolean 값을 받아 Boolean 객체 참조로 변환해준다.

```javascript
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

\***이야기 중인 팩터리 메서드는 디자인 패턴에서 설명하는 팩터리 메서드와는 다르다.**

위와 같이 클래스는 클라이언트에 public 생성자 대신 정적 팩터리 메서드를 제공할 수 있다. 이 방식에는 장점과 단점이 모두 존재한다. 먼저 정적 팩터리 메서드가 생성자보다 좋은 장점 다섯 가지를 알아보자.

**첫 번째, 이름을 가질 수 있다.** 생성자에 넘기는 매개변수와 생성자 자체만으로는 반활될 객체의 특성을 제대로 설명하지 못한다. 반면 정적 팩터리 메서드를 활용하면 반환될 객체의 특성을 쉽게 묘사할 수 있다. 예를 들어 생성자인 BigInteger\(int, int, Random\)와 정적 팩터리 메서드인 BigInteger.probablePrime 중 어느 쪽이 '값이 소수 인 BigInteger를 반환한다'라는 의미를 더 잘 설명할 수 있을지를 생각해 보라.

하나의 시그니처로는 생성자를 하나만 만들 수 있다. 입력 매개변수들의 순서를 다르게 한 생성자를 새로 추가하는 식으로 이 제한을 피해 볼 수도  있지만, 좋은않은 발상이다. 그런 API를 사용하는 개발자는 각 생성자가 어떤 역할을 하는지 정확히 기억하기 어려워 엉뚱한 것을 호출하는 실수를 유발 할 수 있다.

정적 팩터리 메서드 그에 맞고 각각의 차이를 잘 드러내는 이름을 지어서 사용해보자.

**두 번째, 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.** 이 덕분에 불변 클래스는 인스턴스를 미리 만들어 놓거나 새로 생성ㅅ한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다. 대표적인 예로는 Boolean.valueOf\(boolean\) 메서드는 객체를 아예 생성하지 않는다. 따라서 \(생성 비용이 큰\) 같은 객체가 자주 요청 되는 상황이라면 성능을 상당히 끌어올려 준다.

```java
//Boolean의 valueOf

public static final Boolean TRUE = new Boolean(true);

public static final Boolean FALSE = new Boolean(false);

public static Boolean valueOf(String s) {
    return parseBoolean(s) ? TRUE : FALSE;
}
```

반복되는 요청에 같은 객체를 반환하는 식으로 정적 패거리 방식의 클래스는 언제 어느 인스턴스를 살아 있게 할지를 철저히 통제 할 수 있다. 이런 클래스를 인스턴스 통제 클래스라 한다. 그렇다면 인스턴스를 통제하는 이유는 무엇일까? 인스턴스를 통제하면 클래스를 싱글턴으로 만들 수도, 인스턴스화 불가로 만들 수도 있다. 또한 불변 값 클래스에서  동치인 인스턴스가 단 하나 임뿐임을 보장할 수 있다. \(a == b일 때만 a.equals\(b\)가 성립\). 인스턴스 통제는 플라이웨이트 패턴의 근간이 되며, 열거 타입은 인스턴스가 하나만 만들어짐을 보장한다.

**세 번째, 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.** 이 능력은 반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 '엄청난 유연성을' 선물한다. API를 만들 때 이 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지할 수 있다. 이는 인터페이스를 정적 패터리 메서드의 반환 타입으로 사용하는 인터페이스 기반 프레임워크를 만드는 핵심 기술이기도하다.

Java 8 전에는 인터페이스에 정적 메서드를 선언할 수 없었다. 그렇게 때문에 이름이 "Type"인 인터페이스를 반환하는 정적 메서드가 필요하면, "Types"\(인스턴스화 불가인\) 동반 클래스를 만들어 그안에 정의하는 것이 관례였다.

Java 8 이전의 표준 라이브러리에서는 인터페이스와 관련된 정적 메서드들을 동반 클래스\(companion class\)에서 제공했다. 대표적인 예로 Collection 인터페이스와 Collections 동반 클래스가 있다.

```java
// 인터페이스와 동반 클래스의 예.
// Collections 인스턴스화 불가 클래스 
Collection<String> empty = Collections.emptyList();
```

컬렉션 프레임워크는 45개의 클래스를 공개하지 않아 API의 의견을 훨씬 작게 만들 수 있었다. 프로그래머는 명시한 인터페이스대로 동작하는 객체를 얻을 것임을 알기에 굳이 별도 문서를 찾아가며 실제 구현 클래스가 무엇인지 알아보지 않아도 되었고 따라서 API를 사용하기 위해 익혀야 하는 개념의 수와 난이도도 낮췄다.

Java 8 부터는 인터페이스에 바로 정적 메서드를 추가할 수 있기 때문에 동반 클래스를 따로 정의하지 않아도 된다. Java 8에 추가된 Stream 인터페이스는 유용한 정적 메서드들을 제공한다. Java 9 부터는 private 정적 메소드까지 허락하지만 정적 필드와 정적 멤버 클래스는 여전히 public이어야 한다.

```java
Stream<String> chosunKings     = Stream.of("김영훈", "이현규", "임준엽", "몽키");
Stream<String> southKoreaKings = Stream.empty();
```

**네 번째, 입력 매개변수에 따라 매번 다른 클래스의 객체를 반활할 수 있다** 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다. 심지어 다음 릴리스에서 또 다른 클래스의 객체를 반환해도 된다.

예로 EnumSet 클래스는 public 생성자 없이 오직 정적 팩터리만 제공하는데, OpenJDK에서는 원소의 수에 따라 두가지 하위 클래스 중 하나의 인스턴스를 반환한다. 원소가 64개 이하면 원소들을 long 변수 하나로 관리하는 ReaularEnumSet의 인스턴스를,  65개 이상이면 long 배열로 관리하는 JumboEnumSet의 인스턴스를 반환한다.

클라이언트는 이 두 클래스의 존재를 모른다. 만약 원소가 적을 때 RegularEnumSet을 사용할  이점이 없어진다면 다음 릴리즈 때는 이를 삭제해도 아무 문제가 없다. 성능을 더 개선한 세 번째, 네 번째 클래스를 다음 릴리즈에 추가 할 수 있다. 클라이언트는 팩터리가 건네주는 객체가 어느 클래스의 인스턴스인지 알 수도 없고 알 필요도 없다. EnumSet의 하위 클래스이기만 하면 되는 것이다.

**다섯 번째, 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.** 이런 유연함은 서비스 제공자 프레임워크를 만드는 근간이 된다. 사용자 제공자 프레임워크는 **다양한 서비스 제공자들이 하나의 서비스를 구성하는 시스템**으로 대표적인 서비스 제공자 프레임워크로는 JDBC가 있다.  서비스 제공자 프레임워크에서의 제공자는 서비스의 구현체다. 그리고 이 구현체들을 클라이언트에 제공하는 역할을 프레임워크가 통제하여, 클라이언트를 구현체로부터 분리해준다.

서비스 제공자 프레임워크는 세 가지 핵심 컴포넌트로 구성된다.

1. 서비스 제공자가 구현하는 **서비스 인터페이스**
2. 구현체를 시스템에 등록하여 클라이언트가 쓸 수 있도록 하는 **제공자 등록 API**
3. 클라이언트에게 실제 서비스 구현체를 제공하는 **서비스 접근 API**
4. **\(더불어\) 서비스 제공자 인터페이스** 

JDBC를 예로들어 Connection이 서비스 인터페이스 역할을 DrivrManager.registerDriver가 제공자 등록 API 역할을, DirverManager.getConnection이 서비스 접근 API 역할을, Driver가 서비스 제공자 인터페이스 역할을 수행한다.

아래는 간단한 소스와 설명이다.

```java
// DB Connection 정보
String driverName = "com.mysql.jdbc.Driver";
String url = "jdbc:mysql://localhost:3306/effective";
String user = "root";
String password = "1234";
 
try {
    // 제공자 등록 API
    Class.forName(driverName);
    
    // 서비스 접근 API DriverManager.getConnection
    // 서비스 인터페이스 Connection
    Connection conn = DriverManager.getConnection(url, user, password); 
} catch (Exceoption e) {
    e.printStackTrace();
}
```

class.forName\(String name\)은 자바 가상머신이 동작을  시작하기 전까지는 어떤 JDBC 드라이버가 사용 될 지 모르기 때문에, 동적으로 드라이버를 로딩하기 위해 리플렉션\(java.lang.reflect\)을 이용한다. 따라서 그에 맞는 Driver.class 위치를 통해 생성하는 것인데. Class.forName이 아무런 반환도 없이 어디에 어떻게 등록 되는지는 static에 있다. 정적 블록을 통한 인스턴스 생성을 통하여 인스턴스를 관리하기 때문이다.

참조 - [https://heavyfive.tistory.com/entry/Class-%ED%81%B4%EB%9E%98%EC%8A%A4%EC%9D%98-%EC%9A%A9%EB%8F%84](https://heavyfive.tistory.com/entry/Class-%ED%81%B4%EB%9E%98%EC%8A%A4%EC%9D%98-%EC%9A%A9%EB%8F%84) 

그렇다면 단점을 무엇일까? 정적 팩터리 메서드의 단점을 알아보자.

**첫 번째, 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.** 앞서 이야기한 컬렉션 프레임워크의 유틸리티 구현 클래스들은 상속할 수 없다. 이 제약은 상속보다 컴포지션을 이용하도록 유도하고 불변 타입으로 만들려면 이 제약을 지켜야 한다는 점에서 오히러 장점으로 받아들일 수도 있다.

**두 번째, 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.** 생성자처럼 API설명에 명확히 드러나지 않으니 사용자는 정적 팩터리 메서드 방식 클래스를 인스턴스화할 방법을 알아내야 한다.

정적 팩토리 메서드에 흔히 사용되는 명명 방식들이다.

1\) from: 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형 변환 메서드

    예\) Date date = Date.from\(instant\);

2\) of: 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집게  메서

    예\) Set&lt;Rank&gt; faceCards = EnumSet.of\(JACK,  QUEEN, KING\)

3\) valueOf: form과 of의 더 자세한 버전

    예\) BigInteger prime = BigInteger.valueOf\(Integer.MAX\_VALUE\)

4\) instance & getInstance: 매개변수를 받는다면 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다.

    예\) StackWalker luke = StackWalker.getInstance\(options\)

5\) create & newInstance: instance & getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다.

    예\) Object array = Array.newInstance\(classObject, arrayLength\)

6\) getType: getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. "Type"은 팩터리 메서드가 반환할 객체의 타입이다.

    예\) FileStore fs = Files.getFileStore\(path\);

7\) newType: newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팫터리 메서드를 정의할 때 쓴다. "Type"은 팩터리 메서드가 반환할 객체의 타입이다. 

   예\) BuffererdReader br = Files.newBufferedReader\(path\);

8\) type: getType과 newType의 간결한 버전

   예\) LIst&lt;Complaint&gt; comList = Collenctions.list\(legacyCom\); 

**`간단정리 정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있다. 장단점을 이해하고 사용하는것이 좋다. 그렇다고 하더라도 정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공하지는 말자.`**

### 아이템 2 - 생성자에 매개변수가 많다면 빌더를 고려하라.

**빌더를 사용하는 이유**

* 정적 팩토리 메서드에는 public 생성자와 같이, 매개변수가 많은 경우 적절히 대응하기 어렵다는 단점이 있다.
* 기존에는 필수 매개변수 / 선택적 매개변수를 받는 생성자를 늘려가며 작성하는 **점층적 생성자 패턴**이 주로 사용됐다.
* 하지만 점층적 생성자 패턴도 결국은 원치않는 매개변수가 포함될 가능성이 높고, 이런 코드를 사용하는 클라이언트 측 코드가 작성하기 어렵고 읽기 어려워진다.
* 두 번째는 객체를 생성하고 Setter로 값을 설정하는 **자바빈즈 패턴**이다.
* 이 방법은 **점층적 생성자 패턴**에 비해 읽기는 쉽지만 객체 하나 생성을 위해 클라이언트가 여러 메소드를 호출해야하고, 값 설정이 완료되기 전까지는 일관성이 무너진 상태에 놓이게 된다.
* 또한 불변성이 깨지고 스레드 안정성을 위해 추가 작업이 필요하다.

**점층적 생성자 패턴의 안정성과 자바빈즈 패턴의 가독성을 겸비한 빌더 패턴**

* 빌더 패턴은 클라이언트가 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자\(혹은 정적 팩토리 메소드\)를 호출해 빌더 객체를 얻는다.
* 이후 선택적 매개변수를 설정해주고 마지막으로 build 메소드를 호출하여 필요한 객체를 얻어낸다.
* 빌더는 생성할 클래스 안에 정적 멤버 클래스로 만들어두는게 일반적이다.
* 빌더 패턴 예시

  ```java
  public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 (default 값 초기화)
        private int calories = 0;
        private int fat = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            this.calories = val;
            return this;
        }

        public Builder fat(int val) {
            this.fat = val;
            return this;
        }        

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        this.servingSize = builder.servingSize;
        this.servings = builder.servings;
        this.calories = builder.calories;
        this.fat = builder.fat;
    }
  }
  ```

* 빌더 패턴에서 입력값의 유효성 검사 후 IllegalArgumentException을 사용해 오류를 발생시켜 주면 된다.

 **빌더패턴과 계층 설계 클래스**

* 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다
* 추상 클래스는 추상 빌더를, 구체 클래스\(concrete class\)는 구체 빌더를 갖게 한다.
* 추상 클래스 예시
* 아래 예시에서 **self 추상 메소드**는 셀프 타입 관용구\(simulated self-type\)라 함

  ```java
  public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        // addTopping 메소드에 형변환 없이 메소드 체이닝을 지원하기 위해 self 이용
        public T addTopping(Topping topping) {
            toppings.add(topping);
            return self();
        }

        abstract Pizza build();

        public abstract T self();
    }

    Pizza(Builder<?> builder) {
        this.toppings = builder.toppings.clone();
    }
  }
  ```

* 하위 클래스 예시
* build 메소드 재정의 시 **자신의 타입을 반환\(공변 반환 타이핑\)**을 통해 클라이언트 코드에서 형변환을 하지 않아도 되게함

  ```java
  public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        // 하위 클래스의 build 메소드에서 상위 클래스의 정의대신 하위 타입을 반환하는 것을 공변 반환 타이핑이라 함
        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }

        // self 메소드 구현
        @Override
        public Builder self() {
            return this;
        }
    }

    NyPizza(Builder builder) {
        super(builder);
        this.size = builder.size;
    }
  }
  ```

#### 결론 

* 생성자나 정적 팩토리 메소드가 처리해야할 매개변수가 많다면 빌더 패턴을 선택하는게 낫다.
* 매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 그렇다.
* 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간편하다.
* 빌더는 자바빈즈보다 훨씬 안전하다.

### 아이템3 - private 생성자나 열거 타입으로 싱글턴임을 보증하라.

싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다. 싱글턴의 전형적인 예로는 함수와 같은 무상태 객체나 설계상 유일해야 하는 시스템 컴포넌트를 들 수 있다. **클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기 어려워 질 수 있다.** 타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴이 아니라면 인스턴스를 가짜 구현으로 대체할 수 없기때문이다.

싱글턴을 만드는 방식은 보통 둘 중 하나다. 두가지 방식 모두 private 생성를 통하여 인스턴스화를 막는다.

**첫 번째, public static final 필드 방식의 싱글**

```java
public class Single {
    public static final Single INSTANCE = new Single();
    
    // private를 사용한 인스턴스화 방지
    private Single {} ();
}
```

private 생성자는 public static final 필드인 Single.INSTANCE를 초기화할 때 딱 한 번만 호출된다. public이나 protected 생성자가 없으므로 Single 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에 하나뿐임이 보장된다.

public static final 필드 방식의 싱글턴의 장점은 첫 번째 해당 클래스가 싱글턴임이 API에 명백히 드러난다는 것이다. public static 필드가 final이니 절대로 다른 객체를 참조할 수 없다. 두 번째 장점은 간결함이다. 

**두 번째, 정적 팩터리 방식의 싱글턴**

```java
public class Single {
    private static final Single INSTANCE = new Single();
    public static Single getInstance() {
        return INSTANCE;
    }
    
    // private를 사용한 인스턴스화 방지
    private Single {} ();
}
```

Single.getInstance는 항상 같은 객체의 참조를 반환하므로 제2의 Single 인스턴스란 만들어지지 않는다.

정적 팩터리 방식의 싱글턴의 장점은 첫 번째 장점은 API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다는 점이다. 유일한 인스턴스를 반환하던 메서드가 스레드별로 다른 인스턴스를 넘겨주게 할 수 있다. 두번 째 장점은 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다는 점이다.세번 째 장점은 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있다는 점이다. 위에 장점들이 필요하지 않다면 public 필드 방식이 좋다.

예외는 단 한가지가 있다. 권한이 있는 클라이언트는 리플렉션 API인 AccessibleObject.setAccessible을 사용하여 private 생성자를 호출할 수 있다. 이러한 공격을 방어하려면 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지게 하면 된다.

싱글턴 클래스를 직렬화하려면 단순히 Serializable을 구현한다고 선언하는 것만으로는 부족하다. 모든 인스턴스 필드를 일시적\(transient\)이라고 선언하고 readResolve 메서드를 제공해야한다.

readResolve는 새로운 역 직렬화 된 인스턴스를 반환하기 때문에 아래와 같이 기존에 인스턴스를 반환해 줘야한다.

```java
private Object readResovle() {
    // 기존 인스턴스를 반환하고 가짜인 새로운 인스턴스는 가비지 컬렉션에 맡긴다.
    return INSTANCE;
}
```

**세 번째, 열거 타입 방식의 싱글턴 - 바람직한 방법**

```java
public enum Single {
    INSTANCE:
}
```

public 필드 방식과 비슷하지만, 더 간결하고, 추가 노력 없이 직렬화 할수 있고, 아주 복잡한 직렬화 상황이나 리플렉션 공격에도 제2의 인스턴스가 생기는 일을 완벽히 막아준다. **대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.**

### 아이템4 - 인스턴스화를 막으려거든 private 생성자를 사용하라.

단순히 정적 메서드와 정적 필드만 담은 클래스를 만들고 싶을 때가 있을 것이다. 객체 지향적으로 사고하지 않은 이들이 종종 남용하는 방식이기에 곱게 보지 않지만, 분명 나름의 쓰임새가 있다. 예컨대 `java.lang.Math`와 `java.util.Arrays` 처럼 기본 타입 값이나 배열 관련 메서드들을 모아놓을 수 있다. 또한,  `java.util.Collections`처럼 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드를 모아놓을 수도 있다. 마지막으로, final 클래스와 관련한 메서드들을 모아놓을 때도 사용한다. 

정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 사용하기 위해 설계한 것이 아니다. 하지만 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어준다. 즉, 매개변수를 받지 않는 public 생성자가 만들어지며, 사용자는 이 생성자가 자동 생성된 것인지 구분할 수 없다.

**추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.** 하위 클래스를 만들어 인스턴스화하면 그만이다. 따라서 인스턴스화를 막는 방법은 **private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다**

\*\*\*\*

