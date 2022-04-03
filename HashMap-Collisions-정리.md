# HashMap 정리

### **HashMap 동작 원리**

HashMap에 데이터를 저장하게 되면 bucket 이라는 공간에 데이터를 저장하게 됨  
이때 bucket 은 hashcode 라는 메소드를 이용하여 저장할 bucket 위치를 결정함 그리고 bucket 내에서 equals 메소드를 통해 저장할 값이 같은지 다른지 여부를 판단

- HashMap의 시간 복잡도 : O(1)

### **해시 충돌 (Hash Collisions)**

정상적으로 동작하려면 동일한 key는 같은 hash를 가져야만 함, 하지만 다른 key도 같은 hash를 가질 수 있음(bucket의 size, 혹은 여러 상황에 따라 다를 수 있음)
이를 해시 충돌이라고 부름, 해시 충돌에 대한 회피 방법은 두 가지가 있음

- Open Addressing: 데이터를 삽입하려는 bucket이 사용중이면 다른 비어있는 bucket을 찾아 삽입
  - Linear Probing: 순차적으로 비어있는 bucket을 찾을때까지 진행
  - Quadratic Probing: 이차함수를 이용해 버킷의 위치를 찾음
- Separate Chaining: hash가 같을때 Linked List or Tree(Red-Black Tree)를 사용해 해결하는 방법

> Java에서는 Seperate Chaining을 사용)  
> Java8 부터는 Seperate Chaining에서 처음엔 Linked List를 자료구조로 사용하고 데이터가 8개가 되면 자료구조를 Tree로 변경, 만약 데이터가 삭제되어 다시 6개가 되면 Linked List로 변경  
> hash 충돌이 발생하면 복잡도도 O(1) -> O(n) 으로 변경되는데 여기에 Tree 자료구조를 사용하면 O(n) -> O(log n) 으로 성능이 향상

<br>

### **Hash Bucket 동적 확장**

Hash Bucket의 개수가 적으면 메모리를 아낄수 있지만 해시 충돌로 인한 성능 손실 발생  
그래서 HashMap은 데이터 개수가 일정 이상이 되면 bucket의 개수를 두배로 늘림(bucket의 최대 개수는 2^30)  
bucket이 크기를 두배로 확장하는 임계점은 bucket의 75%가 찼을때의 시점, 자주 일어난다면 성능상 좋지 않음(기존 bucket들을 새로운 bucket으로 옮기는 과정이 일어남)  
HashMap의 초기 용량은 **16**, 로드팩터는 **0.75**

##### **로드팩터= 데이터의 개수/초기 용량**
