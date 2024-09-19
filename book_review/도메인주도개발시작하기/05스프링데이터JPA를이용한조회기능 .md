# **04.12.23 Ch.05 스프링 데이터 JPA를 이용한 조회 기능** 
## **5.1 시작에 앞서** 
CQRS: 명령(Command)모델과 조회(Query)모델을 분리하는 패턴 

명령(Command)모델 : 상태 변경 기능 구현. 도메인모델에서 주로 사용  

조회(Query)모델: 데이터를 조회하는 기능 구현
## **5.2 검색을 위한 스펙** 
조회를 위한 검색 조건은 매우 다양하다. 다양한 검색 조건을 조합할떄 사용할 수 있는 것이 스펙. 

Spec: 애그리거트가 특정 조건을 충족하는지 검사할 때 사용하는 인터페이스 

**스펙**

public interface Speficiation<T> {

`	`public boolean isSatisfiedBy(T agg);

}

리포지터리나 DAO는 검색 대상을 걸러내는 용도로 스펙을 사용한다 

스프링 데이터 JPA를 이용한 스펙구현을 알아보자. 
## **5.3 스프링 데이터 JPA를 이용한 스펙 구현** 
스프링 데이터 JPA가 제공하는 Specification 인터페이스 

public interface Specification<T> extends Serializable {

`	`// not, where, and, or 메서드 생략 

`	`@Nullable

`	`Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder cb);

}

toPredicate 메서드는 JPA Criteria API에서 조건을 표현하는 Predicate을 생성한다. 

Specification 인터페이스를 구현하여 스펙을 구현한다. 

public class OrderIdSpec implements Specification<OrderSummery> {

`	`private String ordererId;

`	`public OrdererIdSpec(String ordererId) {

`		`this.ordererId = ordererId;

`	`}


`	`@Override

`	`public Predicate toPredicate(Root<OrderSummary> root, CriteriaQuery<?> query, CriteriaBuilder cb) {

`		`return cb.equal(root.get(OrderSummary\_.ordererId), ordereId)

`	`} 

}

OrderSummary\_  : JPA 정적 메타 모델을 정의함. 

메타모델클래스는 모델 클래스 이름 뒤에 '\_'을 붙인 이름을 갖는다.정적 메타 모델 클래스는 대상 모델의 각 프로퍼티와 동일한 이름을 갖는 정적 필드를 정의한다. 

스펙 생성 기능을 별도 클래스에 모아서 처리 할 수도 있다 .

public class OrderSummarySpecs {

`	`public static Specification<OrderSummary> ordereId(String ordereId) {

`		`return (Root<OrderSummary> root, CriteriaQuery<?> query, CriteriaBuider cb)-> cb.equal(root.<String>get("ordererId"), ordererId);

`	`}

` 	`public static Specification<OrderSummary> orderDateBetween(LocalDateTime from, LocalDateTime to) {

`		`return (Root<OrderSummary> root, CriteriaQuery<?> query, CriteriaBuider cb)-> cb.between(root.get(OrderSummary\_.orderDate), from, to);

`	`} 

}

람다식을이용해서 처리할 수도 있다. 

Specification<OrderSummary> betweenSpec = OrderSummarySpecs.orderDateBetween(from, to);

## **5.4 리포지터리/DAO에서 스펙 사용하기** 
스펙을 충족하는 엔티티를 검색하고 싶다면 findAll() 메서드를 사용하면 된다. 

//스펙 객체를 생성하고 

Specification<OrderSummary> spec = new OrderIdSpec("user1");

//findAll()메서드를 이용하여 검색 

List<OrderSummary> results = orderSummaryDao.findAll(spec); 

## **5.5 스펙 조합** 
스프링 데이터 JPA가 제공하는 스펙 인터페이스는 스펙을 조합할 수 있는 메서드를 제공한다

and : 두 스펙을 모두 충족하는 조건을 표현하는 스펙 생성 

Specification<OrderSummary> spec1 = OrderSummarySpecs.ordereId("user1");

Specification<OrderSummary> spec2 = OrderSummarySpecs.orderDateBetween(

`	`LocalDateTime.of(2022, 1, 1, 0, 0);

`	`LocalDateTime.of(2022, 1, 2, 0, 0)); 

Specification<OrderSummary> spec3 = spec1.and(spec2); 

or : 두 스펙 중 하나 이상 충족하는 조건을 표현하는 스펙을 생성 

not : 조건을 반대로 적용할 때 사용 

Specification<OrderSummary> spec = Specification.not(OrderSummarySpecs.ordererId("user1"));

null 여부를 판단하는 처리는 Where()메서드를 사용하면 좋다 

Specification<OrderSummary> spec = Specification.where(createNullableSpec()).and(createOtherSpec());

## **5.6 정렬 지정하기** 
스프링데이터 JPA에서 정렬을 지정하는 방법 

메서드 이름에 OrderBy를 사용해서 정렬 기준 지정 

List<OrderSummary> findByOrdererIdOrderbyNuberDesc(string orderId);

정렬 기준 프로퍼티가 두개 이상이면 메서드 이름이 길어지는 단점이 있음. 그리고 메서드 이름으로 정렬 순서가 정해지기 떄문에 상황에 따라 정렬 순서를 변경하기 힘들다. 

Sort 인자로 전달 

List<OrderSummary> findByOrdereId(String, ordereId, Sort sort); 

Sort sort = Sort.by("number").ascending();

List<OrderSummary> results = orderSummaryDao.findByOrderId("user1",sort);

//두개 이상 

Sort sort1 = Sort.by("number").ascending();

Sort sort2 = Sort.by("orderDate").descending();

Sort sort = sort1.and(sort2); 

// 짧게 표현해보기 

Sort sort = Sort.by("number").ascending().and(Sort.by("orderDate").descending()); 

## **5.7 페이징 처리하기** 
스프링데이터 JPA는 페이징 처리를 위해 Pageable 타입을 이용한다. 

find 메서드에 Pagable 타입 파라미터를 사용하면 페이징을 자동으로 처리해줌 

사용 예 

PageRequest pageReq = PageRequest.of(1,10); // 첫번째 인자는 페이지번호, 두번째인자는 한페이지의 갯수 

List<MemberData> user = memberDataDao.findByNameLike("사용자%",pageReq);

페이징과 정렬순서를 지정하는 법 

Sort sort = Sort.by("name").descending();

PageRequest pageReq = PageRequest.of(1,10, sort); // 첫번째 인자는 페이지번호, 두번째인자는 한페이지의 갯수 

List<MemberData> user = memberDataDao.findByNameLike("사용자%",pageReq);

페이지가 제공하는 메서드

Pageable pageReq = PageRequest.of(2,30;

Page<MemberData> page = memberDataDao.findByBlocked(false, pageReq);

List<MemberData> content = page.getContent(); // 조회결과목록

long totalElements = page.getTotalElements(); // 조건에 해당하는 전체 갯수. 페이지 타입 이용하면 조건에 해당하는 전체 갯수도 조회 가능 

int totalPages = page.getTotalPages(); // 전체 페이지 번호 

int number = page.getNumber(); // 현재 페이지 번호 

int numberOfElements = page.getNumberOfElements(); //조회 결과 갯수 

int size = page.getSize(); //페이지 크기 

참고 

Page<MemberData> findByBlocked(boolean blocked, Pageable pageable); // 카운트 쿼리 실행함

List<MemberData> findByNameLike(String name, Pageable pageable); // 카운트 쿼리 실행 안함



List<MemberData> findAll(Specification<MemeberData> spec, Pageable pageable); // 스펙을 사용하는 findAll 메서드에 Pageable 타입을 사용하면 리턴타입이 Page가 아니여도 카운트쿼리 실행 

페이징 처리 필요없다면 리턴타입이  List를 사용해서 불필요한 COUNT쿼리 실행하지 않도록 하자

N개의 데이터가 필요한 경우 

List<MemberData> findFirst3ByNameLikeOrderByName(String name);// name프로퍼티를 기준으로 오른차순으로 정렬해서 처음 3개 정렬

MemeberData findFirstByBlockedOrderById(boolean blocked)

## **5.8 스펙 조합을 위한 스펙 빌더 클래스** 
Spectification<MemberData> spec = SpecBuilder.builder(MemeberData.class)

.ifTrue(searchRequest.isOnlyNotBlocked(),()-> MemberDataSpecs.nonBlocked())

.ifHasText(searchRequest.getName(), name -> MemberDataSpecs.nameLike(searchRequest.getName()))

.toSpec();

List<MemeberData> Result = memberDataDao.findAll(spec, PageRequest.of(0,5));

## **5.9 동적 인스턴스 생성** 
public interface OrderSummaryDao extends Repository<OrderSummary, String> {

`	`@Query(""" 

`			`select new com.myshop.order.query.dto.OrderView(

`				`o.number, o.state, m.name, m.id, p.name

`			`)

`			`from Order o join o.orderLines ol, Member m, Product p

`			`where o.orderer.memberId.id = :ordererId

`			`and o.orderer.memberId.id = m.id

`			`and index(ol) = 0

`			`and ol.productId.id = p.id

`			`order by o.number.number desc

`			`""")

`	`List<OrderView> findOrderView(String ordererId);

}

조회 전용 모델을 만들어서 셀렉트 문으로 가져올 필드 값을 지정. 이렇게 조회 전용 모델을 만드는 이유는 표현 영역을 통해 사용자에게 데이터를 보여주기 위함.

DB와 비즈니스 레벨에 대한 이해와 설계가 중요하다. 한번 JPA를 써보고 싶으면 몽고 디비나 작은 레벨의 프로젝트에 한번 적용해보자. 
## **5.10 하이버네이트 Subselect 사용**
Subselect는 쿼리 결과를 Entity로 맵핑할 수 있음 

@Entity

@Immutable

@Subselect (

`	`"""

`	`select o.order\_number as number,

`	`o.version, o.ordere\_id, o.orderer\_name,

`	`o.total\_amounts, o.receiver\_name, o.state, o.order\_date,

`	`p.product\_id, p.name as product\_name 

`	`from purchase\_order o inner join order\_line ol 

`		`on o.order\_number = ol.order\_number

`		`cross join product p

`	`where 

`	`ol.line\_idx = 0

`	`and ol.product\_id = p.product\_id

`	`"""

)

@Synchronize({"purchase\_order", "order\_line", "product"})

public class OrderSummary {

`	`@Id

`	`private String number;

`	`private long version;

`	`@Column(name = "order\_id")

`	`private String ordereId;

`	`@Column(name = "orderer\_name")

`	`private String ordereName; 

...생략



`	`protected OrderSummary() {

`	`}

}

Immutable, Subselect, Synchronize는 하이버네이트 전용 애너테이션. 이 태그를 사용하면 테이블이 아닌 쿼리결과를 Entity로 매핑할 수 있다. 

Subselect: 쿼리 실행결과를 맵핑할 테이블 처럼 사용 

Immutable : 

뷰를 수정할 수 없듯이 subselect로 조회한 Entity 역시 수정할 수 없음. 

실수로 Entity의 맵핑필드를 수정했을때 변경내역을 반영하는 업데이트트 쿼리를 실행할 경우 에러가 발생 이러한 문제를 방지하기위해 Immutable 어노테이션 사용. 

Immutable 애노테이션을 사용하면 하이버네이트는 해당 엔티티의 맵핑 필드/ 프로퍼티가 변경되도 DB에 반영하지 않고 무시한다. 

Synchronize: 해당 엔티티와 관련된 테이블 목록을 명시. 

상태 변경 후 조회를 하면  하이버네이트는 커밋하는 시점에 변경사항을 디비에 반영하므로 최신값이 아닌 상태를 조회하게 된다. 

엔티티를 로딩하기 전에 테이블과 관련된 변경이 발생하면 플러시를 먼저 한다. 플러시는 **영속성 컨텍스트의 변경 내용을 데이터베이스에 반영하는 것을 의미**한다

<https://logical-code.tistory.com/130>

select osm.number as number1\_0\_,...생략

from (

`	`select o.order\_number as number, 

`	`o.version, 

...생략

`	`p.name as product\_name

`	`from purchase\_order o inner join order\_line ol

`		`on o.order\_number = ol.order\_number

`		`cross join product p 

`		`where 

`		`o.line\_idx = 0

`		`and ol.product\_id = p.product\_id

`	`) osm

`	`where osm.order\_id = ? order by osm.number desc


서브쿼리 사용하지 않고 있다면 네이티브 SQL 쿼리를 사용하거나 마이바티스와 같은 별도 맵퍼 사용해서 조회 기능 구현해야한다. 
