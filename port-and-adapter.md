# **포트 앤 어댑터 패턴(port-and-adapter-pattern)**

![port-and-adapter-1](https://user-images.githubusercontent.com/28802545/167252178-3f1d5f6b-6f46-465e-bb2d-3d187d6ad459.png)

모듈명 설명 비고

인바운드 어댑터 :외부에서 들어온 요청을 인바운드 포트를 호출해서 처리 예시 : Rest API Endpoint, gRPC  
아웃바운드 어댑터: 비즈니스 로직에서 들어온 요청을 외부 애플리케이션/서비스를 호출해서 처리 예시 : Database, ORM

#### **인바운드 포트** : 도메인 로직 사용을 위해 노출된 Interface

#### **아웃바운드 포트** : 도메인 로직에서 외부 영역을 사용하기 위한 Interface

#### **인바운드 어댑터** : 외부 application/service 와 내부 비지니스 영역(인바운드 포트) 간 데이터 교환을 조정

ex) RestApi, grpc, graphql resolver

#### **아웃바운드 어댑터** : 내부 비지니스 영역(아웃바운드 포트)과 외부 application/service 간 데이터 교환을 조정

ex) Database, ORM, Kafka Producer

이 구조의 핵심은 비지니스 로직이 presentation layer or data access layer 에 의존하지 않는 것
