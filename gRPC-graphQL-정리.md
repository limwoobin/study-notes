# __gRPC__

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