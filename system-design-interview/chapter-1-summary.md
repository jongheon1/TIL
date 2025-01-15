웹 서비스의 규모 확장성 설계: 단일 서버에서 수백만 사용자까지

#### 기본 구조와 발전 과정

**단일 서버 구조**
- 모든 컴포넌트가 한 서버에서 실행 (웹 앱, DB, 캐시 등)

기본적인 요청 흐름
1. 사용자는 도메인 이름을 이용해 웹사이트에 접속한다.
2. DNS 에 질의하여 도메인 이름을 IP 주소로 변환한다.
3. 해당 IP 주소로 HTTP 요청이 전달된다.
4. 요청을 받은 웹 서버는 HTML 페이지나 JSON 형태의 응답을 반환한다.


**계층 분리**
- 웹 서버와 데이터베이스 서버 분리
- 웹 서버는 사용자 요청 처리, 데이터베이스 서버는 데이터 저장 및 조회
- 각 계층을 독립적으로 확장 가능

**데이터베이스 선택**
- 관계형 DB: 대부분의 상황에 적합
- 비관계형 DB
    - case1: 초저지연 필요
    - case2: 비정형 데이터
    - case3: 대용량 데이터


**수직적 규모 확장 vs 수평적 규모 확장**
- 수직적 규모 확장: 서버에 고사양 자원을 추가
- 수평적 규모 확장: 더 많은 서버를 추가


**로드밸런서 도입**
- 트래픽을 여러 서버에 분산
- 서버 장애 시 자동 복구 가능

로드밸런서 동작 방식
1. 사용자는 로드밸런서의 IP 주소로 접속한다.
2. 로드밸런서는 트래픽을 웹 서버에 분산하고 웹 서버는 사용자 요청을 처리한다.
3. 만약 서버 하나가 다운되면 모든 트래픽은 나머지 서버들에게 분산된다.
4. 트래픽이 가파르게 증가하면 현재 서버들로 트래픽을 감당할 수 없는 시점이 오는데, 이 때 로드밸런서는 단순히 더 많은 서버를 추가하면 된다. 그러면 자동으로 트래픽이 분산된다.


**데이터베이스 다중화**
- 주-부 관계로 구성
- 읽기/쓰기 작업 분리
- 주 데이터베이스는 쓰기 연산 처리, 부 데이터베이스는 읽기 연산 처리

데이터베이스 다중화의 이점
- 읽기 연산이 여러 부 데이터베이스 서버로 분산되어 처리되므로 데이터베이스의 부하가 분산됨
- 여러 부 데이터베이스 서버가 병렬로 질의를 처리할 수 있어 전체적인 성능이 향상됨
- 데이터가 여러 지역에 복제되어 있어 한 데이터베이스에 장애가 발생하더라도 다른 데이터베이스에서 데이터를 복구할 수 있음
- 주 데이터베이스 장애 시 부 데이터베이스 중 하나를 주 데이터베이스로 승격시켜 시스템 가용성 보장



**캐시와 CDN**
- 캐시: 빈번한 데이터베이스 조회 방지
- CDN: 정적 콘텐츠 전송 최적화

캐시 동작
1. 요청을 받은 웹 서버는 캐시에 응답이 저장되어 있는지 확인한다.
2. 캐시에 응답이 저장되어 있으면 캐시에서 바로 응답을 반환한다.
3. 캐시에 응답이 없으면 데이터베이스에 질의하여 데이터를 찾아 캐시에 저장한 뒤 클라이언트에 반환한다.

CDN 동작
1. 사용자는 이미지 URL을 통해 이미지를 요청한다. URL 의 도메인은 CDN 서비스 사업자가 제공한 것이다.
2. CDN 서버의 캐시에 이미지가 있으면 캐시에서 이미지를 반환한다.
3. 캐시에 이미지가 없으면 원본 서버에 이미지를 요청한다.
4. 원본 서버는 이미지를 반환하는데 이 때 응답엔 이미지의 유효 기간을 나타내는 TTL 헤더가 있다.
5. 캐시는 이 헤더를 통해 이미지가 유효한 기간을 알 수 있고, 이 기간 동안 캐시에 이미지를 저장한다.


**무상태 아키텍처**
- 세션 정보를 공유 저장소로 이동
- 어떤 서버로도 요청 처리 가능

상태 의존 아케텍처에선 같은 클라이언트로부터의 요청은 항상 같은 서버로 전달되어야 한다.
무상태 아케텍처에선 같은 클라이언트로부터의 요청은 어떤 서버로도 전달될 수 있다.



#### 규모 확장 전략
- 웹 계층은 무상태로
- 모든 계층에 다중화를!
- 캐시는 가능한 많이
- 여러 데이터 센터!
- 정적 콘텐츠는 CDN 으로
- 샤딩을 통해 데이터 계층 규모 확장
- 각 계층은 독립적으로!
- 지속적으로 모니터링하고, 자동화하라!