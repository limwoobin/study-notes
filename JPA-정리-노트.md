### **LazyLoading 에서 debug 모드로 조회시 왜 data를 안가지고 있는가?**

- LazyLoading 시 추가쿼리를 발생해서 데이터를 가져오는것을 proxy 초기화라고함
- 이는 프록시에서 실제 entity를 반환하는것이 아닌 프록시의 target이 entity 와 연결되는것을 의미

### **그렇다면 OneToMany LazyLoading 의 경우는 왜 실제 엔티티를 반환하는가??**

예를들어 Team (1) <-> Member(n) 의 관계를 조회하게 된다면
Member의 경우는 Set or List 와 같은 Collection 객체의 형식을 갖게 된다
이때 Jpa에서는 컬렉션의 효율적인 관리를 위해 PersistentBag(List) or PersistentSet 등의 컬렉션을 이용해
객체를 갖는다. 이 PersistentCollection 은 Jpa의 컬렉션을 관리하면서 프록시의 역할도 같이 한다.  
하지만 Persistent Collection 객체 내의 entity 요소들은 proxy가 아닌 실제 entity 객체이다.  
이 부분은 라이브러리를 뜯어서 보려고 시도했지만 아직 명확한 답을 찾지 못했다...

> #### **JPA 조회 특징**
>
> - 이미 영속성 컨텍스트에 엔티티가 존재한다면 프록시 객체가 아닌 실제 엔티티로 초기화 된다.
> - 만약 영속성 컨텍스트에 엔티티가 아닌 프록시로 존재한다면 동일 엔티티를 조회해도 프록시로 생성된다
>
> ex) ManyToOne 으로 조회시 Lazy 로 설정된 entity 는 proxy형태로 있다
> 하지만 만약 Lazy로 가져올 entity가 그전에 이미 생성되어있다면 proxy로 가지지 않고 실제 entity로 반환된다.
