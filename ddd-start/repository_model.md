---
description: 제4장 리포지터리와 모델구현(JPA 중심)
---

# 4.리포지터리와 모델구현\(JPA 중심\)

## JPA를 이용한 리포지터리 구현

### 모듈위치

* 리포지터리의 인터페이스는 애그리거트와 같이 도메인 영역에 속하고 리포지터리를 구현한 클래스는 인프라스트럭처 영역에 속한다.

### 리포지터리 기본 기능 구현

* 리포지터리의 기본 기능은 아래와 같이 두가지이다.

  * 아이디로 애그리거트 조회하기
  * 애그리거트 저장하기

  이 두 메서드를 위한 리포지터리 인터페이스는 다음과 같은 형식을 갖는다.

  ```java
  public interface OrderRepository {
      public Order findById(OrderNo no);
      public void save(Order order);
  }
  ```

* 리포지토리 인터페이스는 애그리거트 루트를 기준으로 작성한다.
* 애그리거트 조회하는 기능의 이름을 지을 때 틀별한 규칙은 없지만 findBy 형식을 사용한다.
* findById\(\)는 아이디에 해당하는 애그러거트가 존재하면 Order를 리턴하고, 존재하지 않으면 null를 리턴한다.
* save\(\)는 전달받은 애그리거트를 저장한다. 
* 이 인터페이스를 구현한 클래스는 JPA의 EntityManager를 이용해서 기능을 구현한다.`리스트 4.1참고`
* 애그리거트를 수정한 결과를 저장소에 반영하는 메서드를 추가할 필요는 없다. JPA를 사용하면 트랜잭션 범위에사 변경한 데이터를 자동으로 DB에 반영하기 때문이다.
  * 아래 코드에서 changeShippingInfo\(\) 메서드는 스프링 프레임워크의 트랜잭션 관리가능을 통해 트랜잭션 범위에서 실행된다. 메서드의 실행이 끝나면 트랜잭션을 커밋하는데 이때 JPA는 트랜잭션 범위에서 변경된 객체의 데이터를 DB에 반영하기위해서 UPDATE 쿼리를 실행한다.

    ```java
    public class ChangeOrderService {
        @Transactional
        public void changeShippingInfo(OrderNo no, Shipping newShippingInfo) {
            Order order = orderRepository.findById(no);
            if (order == null) throw new OrderNotFoundException();
            order.changeShippingInfo(newShippingInfo);
        }
    }
    ```
* 아이디가 아닌 다른 조건으로 애그리거트를 조회해야 하는 경우 findBy 뒤에 조건대상이 되는 프로퍼티 이름을 붙인다.
* 아이디 이외 다른 조건으로 애그리거트를 조회할 때는 JPA의 Criteria나 JPQL을 사용한다. `리스트 4.2참`
* 애그리거트를 삭제하는 기능이 필요할 경우 아래와 같이 삭제 할 애그리거트 객체를 파라미터로 받는다. 삭제 기능 또한 EntityManager로 remove\(\) 메서드를 이용해서 삭제기능을 구현한다. `리스트 4.3참고`

```java
public interface OrderRepository {
    ...
    public void delete(Order order);
}
```

## 매핑구현

### 벨류 컬렉션: 별도 테이블 매핑

* Order 엔티티는 한 개 이상의 OrderLine을 가질수 있고 OrderLine은 순서가 있다면 아래와 같이 List 타입을 이용해서 OrderLine 타입의 컬렉션을 프로퍼티로 갖게 된다

  ```java
  public class Order {
      private List<OrderLine> orderLines;
  }
  ```

* 벨류 타입의 컬렉션은 별도 테이블에 보관한다 `그림 4.4참고`
* 벨류 컬렉션을 별도 테이블로 매핑할 때는 `@ElementCollection` 과 `@CollectionTable`을 함께 사용한다.

  ```java
  import javax.persistence.*;

  @Entity
  @Table(name = "purchanse_order")
  public class Order {
      ...
      @ElementCollection
      @CollectionTable(name = "order_line",
                      joinColums = @JoinColum(name = "order_number"))
      // 인덱스 값
      @OrderColum(name = "line_idx")
      private List<OrderLine> orderLines;
      ...
  }

  @Embeddable
  public class OrderLine {
      @Embedded
      private ProductId productId;
    
      @Colum(name = "price")
      private Money price;
    
      @Colum(name = "quantity")
      private int quantity;
    
      @Colum(name = "amounts")
      private Money amounts;
    
      ...
  }
  ```

* 위처럼 OrderLIne에는 List의 인덱스 값을 저장하기 위한 프로퍼티가 존재하지 않는다. 그 이유는 List 타입 자체가 인덱스를 갖고 있기 때문이다.
* @OrderColum 어노테이션을 이용해서 저장한 컬럼에 리스트의 인덱스 값을 저장한다.
* @CollectionTable은 벨류를 저장할 테이블을 지정할 때 사용한다. name속성으로 테이블 이름을 지정하고 joinColums 속성은 외부키로 사용하는 컬럼을 지정한다.

### 벨류 컬렉션: 한 개 컬럼 매핑

* 벨류 컬렉션은 별도 테이블이 아닌 한 개 컬럼에 저장해야 할 경우가 있다.
  * ex\) 도메인 모델에는 이메일 주소 목록을 Set으로 보관하고 DB에는 한 개 컬럼에 콤마로 구분해서 저장해야 할 경우
* 이럴 경우 AttributeConverter를 사용하면 벨류 컬렉션 한 개 컬럼에 쉽개 매핑할 수 있다. 단 AttributeConverter를 사용하려면  아래와 같이 벨류 컬렉션을 표현하는 새로운 벨로 타입을 추가해야 한다.

  ```java
  public class EmailSet {
      private Set<Email> emails = new HashSet<>();
    
      private EmailSet() {};
      private EmailSet(Set<Email> emails) {
          this.emails.addAll(emails);
      }
    
      public Set<Email> getEmails() {
          // 수정 불가능한 Set 리
          return Colletions.unmodifiableSet(emails);
      }
  }
  ```

* 벨류 컬렉션을 위한 타입을 추가했다면 AttributeConverter를 구현한다.

  ```java
  @Converter
  public class EmailSetConvertor implement AttributeConverter<EmailSet, String> {
      @Override
      public String converterToDatabaseColum(EmailSet attribute) {
          if (attribute == null) return null;
          return attribute.getEmails().stream()
                  .map(Email::toString)
                  .collect(Collectors.joining(","));
      }
    
      @Override
      public EmailSet convertToEntityAttribute(String dbData) {
          if (dbData == null) return null;
          String[] emails = dbData.split(",");
          Set<Email> emailSet = Arrays.stream(emails)
                              .map(value -> new Email(value))
                              .collect(toSet());
        
          return new EmailSet(emailSet);
      }
  }
  ```

* AttributeConverter를 생성했다면 EmailSet 타입 프로퍼티가 Converter로 사용하도록 설정한다.

  ```java
  @Colum(name = "emails")
  @Convert(converter = EmailSetConverter.class)
  private EmailSet emailSet;
  ```

### 벨류를 이용한 아이디 매핑

* 식별자는 최종적으로 문자열이나 숫자와 같은 기본 타입이기 때문에  String이나 Long 타입을 이용해서 식별자를 매핑한다.
* 식별자가 기본 타입을 사용하는 것이 나쁘진 않지만 식별자라는 의미를 부각시키기 위해서 식별자 자체를 별도 벨류 타입으로 만들 수도 있다. 식별자 타입을 기본 타입으로 설정하면 @Id 어노테이션을 사용하지만 벨류 타입으로 식별자 타입을 설정하게 되면 @EmbeddedId 어노테이션을 사용한다.

  ```java
  @Entity
  @Table(name = "purchase_order")
  public class Order {
      @EmbeddedId
      private OrderNo number;
      ...
  }

  @Embeddedable
  public class OrderNo implements Serializable {
      @Colum(name = "order_number")
      private String number;
  }
  ```

* JPA에서 식별자 타입은 Serializable 타입이여야 하므로 식별자로 사용될 벨류 타입은 Serializable 인터페이스를 상속받아야 한다.
* 벨류 타입으로 식별자를 구현할 때 얻을 수 있는 장점은 식별자에 기능을 추가할 수 있다는 점이다.

### 별도 테이블에 저장하는 벨류 매핑

* 애그리거트에서 루트 엔티티를 뺀 나머지 구성요소는 대부분 벨류이다. 루트 엔티티 외에 또 다른 엔티티가 있다면 진짜 엔티티가 맞는지 의심해봐야 한다.
* 또 엔티티가 맞다면 다른 애그리거트는 아닌지 확인해야 한다.독자적인 라이프사이클을 갖는다면 다른 애그리거트일 확률이 높다.
* 애그리거트에 속한 객체가 벨류인지 엔티티 인지 구분하는 방법은 식별자를 갖는지 여부를 확인하는 것이다. 하지만, 식별자를 찾을 때 매핑되는 테이블의 식별자를 애그리거트 구성요소의 식별자와 동일한 것으로 착각하면 안 된다. `그림 4.5참`
* 별도 테이블에 저장하는 벨류 매핑할 경우 벨류 타입에는 @Embeddedable 매핑하고, 벨류를 매핑한 테이블을 지정하기 위해서 @SecondaryTable과 @AttributeOverride를 사용한다. `리스트 4.5참고`
* @SecondaryTable의 name 속성은 벨류를 지정할 테이블을 지정한다. pkJoinColumns 속성은 벨류 테이블에서 엔티티 테이블로 조인할 떄 사용할 컬럼을 지정한다. @AttributeOverride 어노테이션을 사용해서 해당 벨류 데이터가 저장된 테이블 이름을 지정한다.
* @SecondaryTable을 이용하면 아래 코드를 실행할 때 두 테이블을 조인해서 데이터를 조회한다.

  ```java
  // @SecondaryTable로 매핑된 article_content 테이블 조
  Article article = entityManager.find(Aticla.class, 1L);
  ```

### 벨류 컬렉션을 @Entity로 매핑하기

* 개념적으로 벨류인데  구현 기술의 한계나 팀 표준 때문에 @Entity를 사용해야 할 때가 있다 
* JPA는 @Embeddable 타입의 클래스 상속 매핑을 지원하지 않는다. 따라서 상속 구조를 갖는 벨류 타입을 사용하려면 @Embeddable 대신 @Entity를 이용한 상속 매핑으로 처리해야한다.
* @Entity로 매핑하면 식별자 매핑을 위한 필드도 추가해야한다.
* 개념적으로 벨류지만 구현 기술의 문제로 @Entity를 사용할 경우 상태 변경 메서드를 제공하지 않는다.
* 한 테이블에 하위 클래스를 매핑하게되면 @Inheritance 설정으로 strategy 값을 SINGLE\_TABLE로 사용하고 @DiscriminatorColumn을 이용해서 타입을 구분하는 용도로 사용할 컬럼을 지정한다. `리스트 4.6참고`
* 책에서 Image @Entitiy인 벨류로 독자적인 라이프사이클을 갖지 않고 Product에 완전히 의존하기 때문에 `cascade` 속성으로 Product와 함께 저장되고 함께 삭제되도록 설정한다. 리스트에서 Image 객체를 제거하면 DB에서 항께 삭제되도록 `orphanRemoval`도 true로 설정한다.

  ```java
  @OneToMany(cascade = {CascadeType.PERSIST, CascadeType.REMOVE},
                                  orphanRemoval = true)
  @JoinColumn(name = "product_id")
  @OrderColumn(name = "list_idx")
  private List<Image> images = new ArrayList<>();
  ```

* 하지만 @OneToMany 매핑에서 컬렉션의 clear\(\) 메서드를 호출하면 삭제과정이 효율적이지 않을 수 있다. SELECT 쿼리로 대상 엔티티를 로딩하고 각 개발로 DELETE를 실행한다.
* 하이버네이트는 @Embeddable 타입에 대한 컬렉션은 한번에 DELETE를 실행하여 위보다 성능 에서 좋다. 따라서 책  `리스트 4.6` 과 같은 문제를 if-else로 구분해야한다. 성능 부분을 잘 고려해서 ㅏㅇ법을 선택해야한다.

### ID 참조와 조인 테이블을 이용한 단방향 M-N 매핑

* 애그리거트 간 집합 연관은 성능상의 이유로 피해야한다.
* 요구사항을 구현하기 위해서 집합 연관을 사용해야 한다면 ID 참조를 이용한 단방향 집합 연관을 적용해 볼 수 있다.

  ```java
  @Entity
  @Table(name = "product")
  public class Product {
      @EmbeddedId
      private ProductId id;
    
      @ElemnetCollection
      @CollectionTable(name = "product_category",
                  joinColumns = @JoinColumn(name = "product_id"))
      private Set<CategoryId> categoryIds;
    
      ...
  }
  ```

* 위 코드는 Product에서 Category로 단방향 M:N 연관을 ID 참조 방식으로 구현한 것이다.
* 애그리거트를 직접 참조했다면 영속성 전파나 로딩 전략을 고민해야 하는데 ID 참조 방식으로 그런 이슈를 없앴다.

## 애그리거트 로딩 전략

## 애그리거트의 영속성 전파

## 식별자 생성 기 





