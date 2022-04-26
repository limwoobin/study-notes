### **LazyLoading 시 왜 data를 안가지고 있는가?**

- LazyLoading 시 추가쿼리를 발생해서 데이터를 가져오는것을 proxy 초기화라고함
- 이는 프록시에서 실제 entity를 반환하는것이 아닌 persistence context에 실제 엔티티 생성을 요청하고 생성된 실제 엔티티를 프록시 객체의 참조 변수에 할당하는것을 말함
- 즉 lazyloading 된 데이터들은 proxy 형태로 리턴됨

### Jpa Proxy 초기화 유의사항

ManyToOne

- Hibernate.initialize() or Hibernate.unproxy
- 초기화시에는 프록시객체의 필드에 접근하면 초기화되는걸로 알고있다.  
  다만 한가지 유의해야할 점이 **getId(pk)** 와 같은 pk 에 접근하는것은 초기화되지 않는다. 왜냐하면 프록시 객체에는 이미 식별자 값을 가지고 있기 때문이다.  
  그래서 추가 쿼리를 발생시키지 않고 갖고있는 식별자를 리턴한다.  
  그렇기 때문에 초기화를 하기 위해선 식별자값을 제외한 필드에 접근해야 초기화 할 수 있다.

OneToMany

- 해당 프록시객체는 Collection 형태로 갖고있을것이다.  
  컬렉션 객체의 요소에 접근하면 초기화가 된다  
  **ex) getList() -> x , getList().get(1) -> o**

### **그렇다면 OneToMany LazyLoading 의 경우는 왜 실제 엔티티를 반환하는가??**

예를들어 Team (1) <-> Member(n) 의 관계를 조회하게 된다면
Member의 경우는 Set or List 와 같은 Collection 객체의 형식을 갖게 된다
이때 Jpa에서는 컬렉션의 효율적인 관리를 위해 PersistentBag(List) or PersistentSet 등의 컬렉션을 이용해
객체를 갖는다. 이 PersistentCollection 은 컬렉션 래퍼가 지연로딩을 처리해준다.  
그렇기 때문에 컬렉션 내의 데이터들은 컬렉션 래퍼가 대신 지연로딩을 해주기때문에 굳이 지연로딩을 할 필요도,
프록시 객체로 들고있을 이유도 없다. 그래서 실제 entity 를 들고있는 것이다.

<hr>

### **getOne() , findOne() 의 차이**

findOne

- em.find 와 같이 실제 엔티티 객체를 리턴

getOne

- em.getReference 와 같이 프록시 객체를 리턴
- 존재하지 않는 id를 통해 조회시 javax.persistence.EntityNotFoundException 을 리턴한다.

> #### **JPA 조회 특징**
>
> - 이미 영속성 컨텍스트에 엔티티가 존재한다면 프록시 객체가 아닌 실제 엔티티로 초기화 된다.
> - 만약 영속성 컨텍스트에 엔티티가 아닌 프록시로 존재한다면 동일 엔티티를 조회해도 프록시로 생성된다
>
> ex) ManyToOne 으로 조회시 Lazy 로 설정된 entity 는 proxy형태로 있다
> 하지만 만약 Lazy로 가져올 entity가 그전에 이미 생성되어있다면 proxy로 가지지 않고 실제 entity로 반환된다.
