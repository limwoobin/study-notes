# **포트 앤 어댑터 패턴(port-and-adapter-pattern)**

모듈명 설명 비고

인바운드 어댑터 :외부에서 들어온 요청을 인바운드 포트를 호출해서 처리 예시 : Rest API Endpoint gRPC  
아웃바운드 어댑터: 비즈니스 로직에서 들어온 요청을 외부 애플리케이션/서비스를 호출해서 처리 예시 : Database, ORM
인바운드 포트: 도메인 코드에 접근하기 위한 인터페이스 클래스 데이터 흐름 : Endpoint → Domain  
아웃바운드 포트: 도메인 코드에 접근하기 위한 인터페이스 클래스 데이터 흐름 : Domain → outbound Adapter  
서비스: 실제 도메인 내용을 처리하는 코드
