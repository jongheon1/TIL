## 처리율 제한 장치의 설계

처리율 제한 장치는 클라이언트 또는 서비스가 보내는 트래픽의 처리율을 제어하기 위한 장치다.

처리율 제한 장치의 이점
- Dos(Denial of Service) 공격에 의한 자원 고갈을 방지할 수 있다.
- 비용을 절감한다. 처리를 제한하면 서버를 줄일 수 있고, 우선순위가 높은 API 에 더 많은 자원을 할당할 수 있다.
- 서버 과부화를 막는다. 봇 또는 비정상적인 트래픽을 차단해 서버 부하를 줄일 수 있다.


### 문제 이해 및 설계 범위 확정

면접관과 소통하면서 어떤 처리율 제한 장치를 설계할 것인지 결정한다.

요구사항
- 설정된 처리율을 초과하는 트래픽을 차단한다.
- 낮은 응답시간
- 가능한 적은 메모리
- 분산형 처리율 제한 장치
- 요청이 제한되었을 때는 사용자에게 그 사실을 알린다.
- 제한 장치에 장애가 생겨도 시스템은 계속 동작해야 한다.


### 개략석 설계안 제시

**처리율 제한 장치의 위치**

클라이언트 측에 둔다면
- 일반적으로 클라이언트 측에 두는 것은 좋지 않다. 쉽게 위번조가 가능하기 때문이다.

서버 측에는 서버 내부 혹은 미들 웨어에 둘 수 있다.
- 서버 내부에 두면 서버 내부에서 처리율 제한 장치를 구현해야 한다.
- 미들웨어엔 클라우드 업체가 제공하는 처리율 제한 장치를 사용할 수 있다.

둘 중 결정하는 몇 가지 지침
- 현재 기술 스택이 서버 측 구현을 지원하기 충분한지 확인 해야 한다.
- 서버 측에서 구현한다면 알고리즘을 자유롭게 선택할 수 있다.
- 직접 만들려면 시간이 든다. 구현하기 힘들다면 상용 API 게이트웨이를 사용하는 것이 좋다.


처리율 제한 알고리즘엔 여러 가지가 있는데, 각기 다른 장단점이 있다.


**토큰 버킷 알고리즘**

토큰 버킷 알고리즘의 동작 원리는 다음과 같다.
1. 토큰 버킷은 지정된 용량을 갖는 컨테이너다.
2. 이 버킷에는 사전 설정된 양의 토큰이 주기적으로 채워진다.
3. 토큰이 꽉 찬 버킷에는 더 이상 토큰이 채워지지 않는다.
4. 요청이 도착하면
4-1. 버킷에 토큰이 있다면 요청을 처리하고 토큰을 소모한다.
4-2. 버킷에 토큰이 없다면 해당 요청은 버려진다.

토큰 버킷 알고리즘은 인자 2개를 받는다.
- 버킷에 담을 수 있는 토큰의 최대 개수
- 초당 몇 개의 토큰이 버킷에 공급되는지

버킷 개수는 공급 제한 규칙에 따라 달라진다.
- API 엔드 포인트 마다 별도의 버킷을 둘 수 있다.
- IP 주소별로 처리율 제한을 적용해야 한다면 IP 주소별로 버킷을 둘 수 있다.
- 시스템의 처리율을 제한하고 싶다면, 모든 요청이 하나의 버킷을 공유하도록 해야 할 것이다.

장점
- 구현이 쉽다.
- 메모리 사용량이 적다.
- 짧은 시간에 집중되는 트래픽을 처리할 수 있다.

단점
- 버킷 크기와 토큰 공급률을 적절하게 튜닝하기 어렵다.


**누출 버킷 알고리즘**

누출 버킷 알고리즘의 동작 원리는 다음과 같다.
1. 누출 버킷은 보통 큐로 구현한다.
2. 요청이 도착하면
2-1. 큐에 빈자리가 있으면 큐에 요청을 추가한다.
2-2. 큐가 가득 찼다면 요청은 버린다.
3. 지정된 시간마다 큐에서 요청을 꺼내어 처리한다.

누출 버킷 알고리즘은 인자 2개를 사용한다.
- 큐의 크기
- 지정된 시간당 몇 개의 항목을 처리할지

장점
- 큐의 크기가 제한되어 있어 메모리 사용량이 적다.
- 처리율이 고정되어 있기 때문에 출력이 안정적이다.

단점
- 단시간에 많은 트래픽이 몰리는 경우엔 처리하기 어려울 수 있다.
- 인자를 적절하게 튜닝하기 어렵다.


**고정 윈도 카운터**

고정 윈도 카운터의 동작 원리는 다음과 같다.
1. 타임라인을 고정된 간격의 윈도로 나누고 각 윈도에 카운터를 둔다.
2. 요청이 접수될 때 마다 이 카운터의 값을 증가시킨다.
3. 카운터의 값이 설정된 임계치에 도달하면 새로운 요청은 새 윈도가 열릴 때까지 버려진다.

장점
- 메모리 효율이 좋다.
- 이해하기 쉽다.
- 윈도 단위로 초기화 하는 방식은 특정한 트래픽 패턴을 처리하기 적합하다.

단점
- 윈도 경계 부근에서 일시적으로 많은 트래픽이 집중될 경우, 기대했던 시스템 처리 한도보다 더 많은 요청이 처리될 수 있다.
    - 타임라인을 분 단위로 나눴을 때, 가령 2:00:30 부터 2:01:30 사이의 1분 동안은 허용 한도의 2배까지 처리될 수도 있다.


**이동 윈도 로깅 알고리즘**

이동 윈도 로깅 알고리즘은 앞선 알고리즘의 문제를 해결한다.

그 동작 원리는 다음과 같다.
1. 이 알고리즘은 요청의 타임스탬프를 추적한다. 
2. 새 요청이 오면 만료된 타임스탬프를 제거한다. 이는 타임스탬프가 현재 윈도의 시작 시점보다 오래된 타임스탬프를 의미한다.
3. 새 요청의 타임스탬프를 로그에 추가한다.
4. 로그의 크기가 허용치 이내라면 요청을 시스템에 전달한다. 그렇지 않은 경우엔 전달하지 않고 둔다.

장점
- 처리율 제한 메커니즘이 아주 정교하다. 어느 순간의 윈도를 보더라도, 허용되는 요청의 수가 시스템의 처리율 한도를 초과하지 않는다.

단점
- 모든 타임스탬프를 보관하기 때문에 메모리 사용량이 많다.


**이동 윈도 카운터 알고리즘**

이것은 고정 윈도 카운터 알고리즘과 이동 윈도 로깅 알고리즘을 결합한 것이다.

현재 윈도에 몇 개의 요청이 온 것인지를 다음과 같이 계산한다.
- 현재 1분 간의 요청 수 + 직전 1분 간의 요청 수 * 이동 윈도와 직전 1분이 겹치는 비율
- 이전 1분 동안 5개의 요청, 현재 1분 동안 3개의 요청, 직전 1분의 70%만 이동 윈도에 포함된다면
- 현재 윈도에 온 요청 수 = 3 + 5 * 0.7 = 6.5

장점
- 이전 시간대의 평균 처리율에 따라 현재 윈도의 상태를 계산하므로 짧은 시간에 몰리는 트래픽에도 잘 대응한다.
- 메모리 사용량이 적다.

단점
- 직전 시간대에 도착한 요청이 균등하게 분포되어 있다고 가정한 상태에서 추정치를 계산하기 때문에 다소 느슨하다.


**개략적인 아키텍처**

요청을 추적하는 카운터를 대상 별로 두고 이 값이 임계치를 초과하면 요청을 모두 버리는 방식이다.

동작 원리는 다음과 같다.
1. 클라이언트가 처리율 제한 미들웨어에게 요청을 보낸다.
2. 미들웨어는 카운터를 가져와서 한도에 도달했는지 확인한다.
3. 한도에 도달하지 않았다면 요청을 API 서버로 전달하고 카운터를 증가시킨다.
4. 한도에 도달했다면 요청을 버린다.


### 상세 설계

1. 처리율 제한 규칙은 디스크에 보관한다. 이 규칙은 수시로 캐시에 저장된다.
2. 클라이언트가 요청을 서버에 보내면 요청은 먼저 처리율 제한 미들웨어에 도달한다.
3. 미들웨어는 제한 규칙과 카운터를 캐시에서 가져온다. 이 값들에 근거해 미들웨어는 다음과 같은 결정을 내린다.
3-1. 해당 요청이 제한에 걸리지 않은 경우에는 API 서버로 보낸다.
3-2. 해당 요청이 제한에 걸린다면 429 too many requests 에러를 클라이언트에 보낸다. 해당 요청은 버릴 수도 있고 큐에 보관할 수도 있다.


**분산 환경에서의 처리율 제한**

단일 서버에선 처리율 제한 장치를 구현하는 것이 쉽다. 하지만 분산 환경에선 처리율 제한 장치를 구현하는 것이 어렵다. 다음 두 문제를 풀어야 하기 때문이다.
- 경쟁 조건
- 동기화


**경쟁 조건**

경쟁 조건을 해결하는 가장 널리 알려진 해결책은 락이다. 하지만 락은 성능을 상당히 떨어뜨린다.
락 대신 루아 스크립트 혹은 정렬 집합을 사용해 해결할 수 있다.


**동기화**

수백만 사용자를 지원하려면 한 대의 처리율 제한 장치 서버로는 충분하지 않다. 따라서 여러 대의 서버를 두면 동기화가 필요해진다.

이에 대한 한 가지 해결책은 고정 세션을 활용해 같은 클라이언트의 요청이 같은 서버로 가도록 하는 것이다.

하지만 이는 확장 가능하지도 않고 유연하지도 않다.

레디스 같은 중앙 집중형 데이터 저장소를 쓰는 것이 더 좋다.


**성능 최적화**

서버에서 멀리 떨어진 사용자를 지원하려면 지연시간이 증가할 수 밖에 없다.

이를 위해 여러 데이터센터를 지원 해 사용자의 트래픽을 가장 가까운 에지 서버로 보내 지연시간을 줄이는 것이 좋다.


**모니터링**

처리율 제한 장치가 효과적으로 동작하는지 확인하기 위해 모니터링이 필요하다.

기본적으로 모니터링을 통해 다음 두 가지를 확인해야 한다.

- 채택된 처리율 제한 알고리즘이 효과적이다.
- 정의한 처리율 제한 규칙이 효과적이다.

상황에 따라 알고리즘과 규칙을 조정해야 할 수도 있다.


### 마무리

처리율 제한 알고리즘과 그것을 구현하는 아키텍처, 분산 환경에서의 처리율 제한 장치, 성능 최적화, 모니터링을 통해 처리율 제한 장치를 설계할 수 있다.

다음과 같은 부분은 더 생각해보면 좋을 것 같다.
- hard, soft 처리율 제한
- 다양한 계층에서의 처리율 제한
- 처리율 제한을 회피하는 방법, 클라이언트를 어떻게 설계하는 것이 최선인가?
