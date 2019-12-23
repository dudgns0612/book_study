---
description: 제1장 도메인 모델 시작
---

# 도메인 모델 시작

## 들어가기 앞서

* 모델은 엔티티와 벨류로 구분할 수 있다.
* 엔티티와 벨류를 이해해야 도메인을 올바르게 설계하고 구현 할 수 있다.

## 엔티티

* \*엔티티의 가장 큰 특징은 엔티티 객체는 고유하기 때문에 식별자를 가진다. 따라서 엔티티 식별자가 같다면 두 엔티티는 서로 같다고 볼 수 있고 엔티티의 속성을 바꾸거나 삭제하기 전까지는 유지된다.
* 따라서 식별자를 구별할 수 있어야 하는데 equals 와 hashcode를 override 하여 사용한다.String.class는 내부적으로 equals를 재정의 하였기 때문에 구별이 가능하지만 다른 Object는 주소값을 비교하기 때문에 따로 override하여 구별 할 수 있도록 해준다. hashcode도 같이 override하여 사용하여야 하는데 이유는 재정의한 equals를 통해 같은 Object로 판별 하였는데 colection들은 hashcode 값이 다르기 때문에 다르게 인식하기 때문.
  * equals

    ```java
    @Override
    public boolean equals(Object obj) {
     if (this == obj) return true;
     if (obj == null) return false;
     if (obj.getClass() != 해당클래스.class) return false;
     해당클래스 compare  = (해당클래스)obj;
     if (this.식별자 == null) return false;
     return this.식별자.equals(compare.식별자);
    }
    ```

    * hashCode

      ```java
      @Override
      public int hashCode() {
      final int prime = 31;
      int result = 1;
      result = prime * result + ((식별자) == null) ? 0 : 식별자.hashCode());
      return result;
      }
      ```

## 엔티티의 식별자 생성

* 특정 규칙에 따라 생성
  * 대표적으로 날짜가 있지만 동시시간 생성 이슈에 신경써야한다.
* UUID 사용
* 값을 직접 입력
  * 이메일, 아아디 사용자에게 직접 입력받게 되므로 중복 입력에 대한 처리 필요하다.
* DB 자동증가 사용
  * DB 등록과 동시에 식별자를 알 수 있다.

## 벨류타입

* 개념적으로 서로 같은 의미의 필드를 묶어 하나로 표현
  * 아래와 같이 받는사람 정보와 주소 정보는 각각 벨류 타입으로 묶을수 있다.

    ```java
    public class shippingInfo {
    // 받는사람 정보
    private String receiverName;
    private String receiverPhoneNumber;

    // 주소 정보
    private String shippingAddress1;
    private String shippingAddress2;
    }
    ```

  * 각 벨류 타입으로 변환

    ```java
    public class Receiver {
      private String name;
      private String phoneNumber;

      // get method
    }
    ```

    ```java
    public class Address {
      private address1;
      private address2;

      // get method
    }
    ```
* 완벽한 하나의 개념으로 묶어 가독성을 높힌다. \(의미를 정확하게 파악\)
* 벨류 객체를 변경 할 때는 기존 데이터를 변경하기보다 새로운 객체를 만들어 생성한다.
  * 아래와 같이 단순한 정수연산이 아닌 돈 계산이라는 명확한 의미가 부여됨.

    ```java
     public class Money {
       private int value;

       ... //생성자, get method

       public Money add(Money money) {
         return new Money(this.value + money.value);
       }

       public Money multiply(int mutiplier) {
         return new Money(value * mutiplier);
       }
     }
    ```
* 변경 기능을 제공 하지 않는 것을 불변\(immutable\)이라 하는데 벨류타입을 불변으로 제공하는 이유는 보다 안전한 코드를 작성하기 위함.
* 불변객체가 아닐경우엔 아래와 같은 문제가 있기 때문에 벨류타입을 사용하는 엔티티에서 변경 값을 받아 새로운 객체로 만들어 준다.

  ```java
     Money price = new Money(1000);
     // 주문목록 생성
     // [price=1000, quantity=2, amount=2000]
     OrderLine line = new OrderLine(product, price, 2);
     // [price=2000, quantity=2, amount=2000]
     price.setValue(2000);
  ```

  * set메서드가 존재하는 불변 객체가 아닐경우 새로운 Money 객체생성을 통한 이슈방지

    ```java
    public class OrderLine {
      private Money price;

      public OrderLine(Product product, Money money, int quantity) {
        this.product = product;
        // 새로운 money객체 할당
        this.price = new Money(price.getValue());
        this.quantity = quantity;
        this.amounts = calaulateAmounts();
      }
    }
    ```

* 엔티티의 경우엔 식별자를 통하여 같은지 비교 하지만 벨류의 경우 모든 속성이 같은지 확인해야 한다.

## 엔티티 식별자와 벨류타입

* 엔티티의 식별자는 String인 경우가 많다. 식별자를 벨류타입으로 정하여 의미를 드러나게 하여 가독성을 높힐수 있다. \(이 부분에 대하여 이야기필요\)

## 도메인 모델에 set 메서드 넣지 않기

* set을 사용할 경우 도메인 객체가 완벽하지 않은 상태가 아닐 때 생성 될 수 있다.
  * 주석 3가지 모두 문제가 된다고 생각이 됨.

    ```java
     // Order가 생성성되는 시점에서 order는 완벽하지 않음.
     Order order = new Order();

     // set으로 모든 정보를 설정해야 함.
     order.setOrderLine(line);
     order.setShippingInfo(info);

     // 주문자의 정보가 설정되지 않은 상태에서 주문이 완료 됨.
     order.setState(OrderState.PREPARING);
    ```
* 생성자를 통해 필요한 데이터를 모두 받을 수 있도록하고 필요한 정보를 토대로 호출 시점에서 데이터들이 올바른지 검사할 수 있다.

  ```java
       public class Order {
         public Order(Orderer orderer, List<OrderLine> orderLines,
          ShippingInfo shippingInfo, OrderState state) {
           setOrderer(orderer);
           setOrderLines(orderLines);
           setShippingInfo(shippingInfo);
           ... // 그 외 설정
         }

         private setOrderer(Orderer orderer) {
           if (orderer == null) {
             throw new IllegalArgumentException("no orderer");
           }
           this.orderer = orderer;
         }

         private void setOrderLines(List<OrderLine> orderLines) {
           verifyAtLeastOneOnMoreOrderLines(orderLines);
           this.orderLines = orderLines;
           calulateTotalAmounts();
         }

         private void verifyAtLeastOneOnMoreOrderLines(List<OrderLine> orderLines) {
           if (orderLines == null || orderLines.isEmpty()) {
             throw IllegalArgumentException("no orderLines");
           }
         }

         private void calulateTotalAmounts() {
           this.totalAmounts = orderLines.stream().mapToInt(x -> x.getAmounts()).sum();
         }
       }
  ```

  * 불변에 벨류경우 set 메소드를 구현 할 필요가 없기 때문에 앞서 설명한 것처럼 벨리데이션이 필요 한 경우 private로 외부에서 변경 하지 못하도록 하고 특별한 이유가 없다면 벨류타입은 불변으로 구현한다.

