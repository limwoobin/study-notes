# __gRPC__
`google remote procedure call` 확장 가능하고 빠른 API를 만드는데 사용되는 오픈소스 RPC 프레임워크.  
최적의 API 보호, HTTP/2, 프로토콜 버퍼 등과 같은 기술 스택을 사용

## __프로토콜 버퍼(Protobuf)__
- 클라이언트 라이브러리의 코드 생성을 자동으로 수행하는 직렬화 또는 역직렬화 표준
- 구조화된 데이터를 직렬화하기 위한 프로토콜, `.proto` 파일에 protocol buffers 메세지 타입을 정의
- 프로토버프는 바이너리 포맷이기에 위와 같은 장점이 없지만 JSON보다 사이즈가 작고 성능 면에서 일반적으로 더 뛰어남


## __스트리밍__
- 하나의 TCP연결을 통해 여러 응답 또는 요청을 동시에 보내거나 받을 수 있는 HTTP/2의 다중화 기능을 통해 이를 가능하게 함
- 즉 새로운 연결을 만들지 않고도 여러 개의 메세지가 병렬적으로 동시에 보낼 수 있는데 이를 `Multiplexing` 이라 함


- HTTP/1.0의 경우 매번 새로운 리퀘스트에 대해서 TCP 커넥션을 새로 만들어야 했으므로 시간과 리소스를 더 소비했다. 이에 비해 HTTP/1.1은 지속적인 연결(persistant connection)을 가능하게 해서 하나의 TCP 커넥션이 유지된 채로, 응답을 기다릴 필요 없이 여러 개의 리퀘스트를 보낼 수 있음.

    - HTTP/1.1은 1.0에 비해 성능을 향상시켰지만 여전히 latency 이슈가 있다. 하나의 커넥션에 여러 개의 요청을 보낼 때, 이 요청들을 queue에 관리하므로 queue의 헤드 요청이 오래 걸리는 경우 뒤에 있는 요청들은 블로킹되는 병목현상이 나타난다. 이를 head-of-line(HOL) blocking 이라고 부른다.

    - 렬적으로 여러 개의 TCP 커넥션을 만들면 이 문제를 완화할 수는 있지만, 클라이언트와 서버 사이에 동시에 유지할 수 있는 TCP 커넥션의 수가 제한되어 있을 뿐더러 TCP 커넥션을 새로 만들 때마다 추가적으로 리소스가 든다. (e.g. TCP handshake)

<br>

## __작동원리__
![gRPC-1](https://user-images.githubusercontent.com/28802545/236472013-e0cc7f5f-eeef-4a07-aabd-def55e24ab09.png)

1. 클라이언트는 stub 생성(서버랑 같은 메소드 제공)
2. stub는 gRPC 프레임워크 호출 (내부 네트워크를 이용해 호출)
3. 클라이언트와 서버는 상호작용을 위해 stubs 사용

<br>
<hr>

# __graphQL__

- 필요한 것만 요청하고 받아오기
- 단일 요청으로 많은 데이터 가져오기
- 가능한 케이스를 타입으로 표현하기

### Schema?
데이터의 모양, graphQl의 스키마는 클라이언트 중심이다

### __특징__
graphQL 은 모든 요청이 POST로 이루어짐  
graphQL 은 모둔 응답이 HttpStatus 200으로 오게됨  
- Query (read)
- Mutation (Create, Update, Delete)

### Scalar
graphql을 타입으로 나타낼 수 있음(커스텀 하게)

### __장점__
- HTTP 요청 횟수를 줄일 수 있음 -> 하나의 endpoint에 여러 api를 요청할 수 있음  
원하는 정보를 하나의 쿼리에 모두 담아 요청할 수 있음
- HTTP 응답 사이즈를 줄일 수 있음  
RestApi의 경우 응답의 형태가 정해져 있지만 gql은 필요한 정보만 요청할 수 있기 때문에
- `Over-Fetching, Under-fetching` 방지
    - Over-Fetching: api호출 시 필요보다 많은 데이터를 가져오는 것.
    - Under-fetching: 하나의 api요청으로 충분한 데이터를 받지 못해 두개 혹은 그 이상의 api를 요청하는 것

### __단점__
- 파일 전송에 대해 한계 -> grapghQL은 `application/json` 형식만 받을 수 있기 때문에  
파일 전송을 위한 `multipart/form-data` 형식을 사용할 수 없음  
이를 사용하기 위해서는 multipart -> json 으로 변환해주는 미들웨어나 라이브러리가 필요함
- Client에서 모든 필드를 작성해야 함
- 데이터 요청 필터링이 까다로움
resrt api는 정해진 형식이 있지만 graphQL은 클라이언트가 스스로 결정하기에 잘못된 요청을 할 수도 있음  
- 일반적으로 HTTP 캐싱전략은 URL마다 정책을 설정하는 형식인데 graphQL은  
엔드포인트가 하나(/graphql)이기에 HTTP에서 사용하는 캐싱전략을 그대로 사용하기 까다로움

Spring에서 GraphQL은 @Valid, OSIV 등의 기능이 정상동작 하지 않는다.  
이유는 graphQL Resolver는 Dispatcher Servlet을 통해 요청이 들어오지 않는다  
하지만 @Valid, OSIV 는 서블릿 필터나 스프링 인터셉터를 통해 생성되고 동작하기에  
rest api는 요청시 Dispatcher Servlet을 통해 컨트롤러로 오지만 graphQL 리졸버는 그렇지 않다  

다만 @Validated는 디스패처 서블릿이 아닌 스프링 AOP를 기반으로 메서드 요청을 가로채 동작하기에 잘 동작한다