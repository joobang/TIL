## Redis란?

아래는 공식 문서 내용

### Introduction to Redis
Redis는 데이터베이스, 캐시, 메시지 브로커 및 스트리밍 엔진으로 사용되는 오픈 소스(BSD 라이선스), 메모리 내 데이터 구조 저장소입니다. Redis는 문자열, 해시, 목록, 집합, 범위 쿼리가 포함된 정렬된 집합, 비트맵, 하이퍼로그 로그, 지리공간 인덱스 및 스트림과 같은 데이터 구조를 제공합니다. Redis에는 복제, Lua 스크립팅, LRU 제거, 트랜잭션 및 다양한 수준의 온디스크 지속성이 내장되어 있으며 Redis Sentinel 및 Redis 클러스터를 통한 자동 파티셔닝을 통해 고가용성을 제공합니다.

문자열에 더하기, 해시의 값 증가, 요소를 목록으로 푸시하기, 집합 교집합, 합집합 및 차이 계산, 정렬된 집합에서 가장 높은 순위를 가진 멤버 가져오기 등의 원자 연산(atomic operations)을 실행할 수 있습니다.

최고의 성능을 달성하기 위해 Redis는 인메모리 데이터 세트와 함께 작동합니다. 사용 사례에 따라, Redis는 주기적으로 데이터 집합을 디스크에 덤프하거나 각 명령을 디스크 기반 로그에 추가하여 데이터를 보존할 수 있습니다. 풍부한 기능의 네트워크 인메모리 캐시만 필요한 경우 지속성을 비활성화할 수도 있습니다.

Redis는 비동기 복제를 지원하며, 빠른 비차단 동기화 및 망 분할 시 부분 재동기화를 통한 자동 재연결 기능을 제공합니다.

------

### Redis의 개념
Redis는 Remote Dictionary Server의 약자로 외부에서 사용 가능한 Key-Value 쌍의 해시 맵 형태의 서버라고 생각할 수 있다. 그래서 별도의 쿼리 없이 Key를 통해 빠르게 결과를 가져올 수 있다.
![](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F7542f7b0-bc80-416e-8039-9f2cdc4b17c3%2FUntitled.png?table=block&id=e60b1004-57ee-46ed-801b-2ba383086d7f&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

또한, 디스크에 데이터를 쓰는 구조가 아니라 메모리에서 데이터를 처리하기 때문에 작업 속도가 상당히 빠르다.

Redis는 고성능 키-값 저장소로서 String, list, hash, set, sorted set 등의 자료 구조를 지원하는 NoSQL이다.

### Redis의 특징
- 영속성을 지원하는 인 메모리 데이터 저장소
- 다양한 자료 구조를 지원함.
- 싱글 스레드 방식으로 인해 연산을 원자적으로 수행이 가능함.
- 읽기 성능 증대를 위한 서버 측 리플리케이션을 지원
- 쓰기 성능 증대를 위한 클라이언트 측 샤딩 지원
- 다양한 서비스에서 사용되며 검증된 기술

### Redis의 영속성
Redis는 영속성을 보장하기 위해 데이터를 디스크에 저장할 수 있다. 서버가 내려가더라도 디스크에 저장된 데이터를 읽어서 메모리에 로딩한다. 데이터를 디스크에 저장하는 방식은 크게 두 가지가 있다.

- RDB(Snapshotting) 방식
    - 순간적으로 메모리에 있는 내용 전체를 디스크에 옮겨 담는 방식

- AOF(Append On File) 방식
    - Redis의 모든 write/update 연산 자체를 모두 log 파일에 기록하는 형태


### Redis의 컬렉션
위 개념만 들으면 Redis는 단순히 데이터를 메모리에 저장하여 빠른 DB이고, Memcached처럼 단순한 Key-Value 쌍이 아니고 다양한 데이터 구조체를 지원한다. 이를 컬렉션이라고 부르는데 아래 그림을 살펴 보자.

![](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F6416f30d-bcfb-4f84-bdec-82fb69a73f43%2FUntitled.png?table=block&id=d56c51ce-1c17-47f7-b92f-624e875adf08&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

Key가 될 수 있는 데이터 구조체가 다양한 것을 알 수 있다. 이렇게 다양한 자료 구조를 지원하게 되면 개발의 편의성이 좋아지고 난이도가 낮아진다는 장점이 있다.

### 싱글 스레드를 사용하는 Redis
Redis는 싱글 스레드를 사용하므로 연산을 원자적으로 처리하여 Race Condition이 거의 발생하지 않는다. 예를 들어, 친구 리스트의 친구를 추가하는 연산을 시도해 보자. 아래와 같이 정상적인 상황에서는 유저 각각의 트랜잭션이 순서대로 잘 행해지고 있으므로 문제가 없다.
![](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F3991fe95-6548-4a1e-9ace-f4b4fa569fb4%2FUntitled.png?table=block&id=49f133cc-a8fd-470c-be63-849ba31a75b7&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

그러나 동시에 친구 리스트에 B, C를 추가한다고 하면 어떨까?

![](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F0d1c3851-0308-4eae-8b93-e772938a0d6d%2FUntitled.png?table=block&id=c5bde51d-f5bb-4a13-a2dc-65bdd3072c51&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

두 트랜잭션이 동일한 최종 상태인 A를 자신의 메모리로 읽어 들이고, 그 상태에서 각자 B 또는 C를 추가하게되면 최종 상태가 (A, B) 혹은 (A, C)가 된다. (A, B) 혹은 (A, C)라고 한 이유는 컨텍스트 스위칭에 따라 두 트랜잭션 중 누가 먼저 끝날 지 예측할 수 없기 때문이다. 물론 이러한 Race Condition을 해결하기 위해 격리 수준 등 여러 가지 기법이 있지만, Redis는 싱글 스레드를 사용하므로 하나의 트랜잭션은 하나의 명령만 실행할 수 있으므로 다수의 Race Condition을 해결할 수 있다. (모든이 아닌 다수라고 한 이유는 더블 클릭 이슈는 싱글 스레드만으로 해결 못함)

---

### Redis 관련 질문

#### Redis는 언제 사용하는가?
DB, Cache, Message Queue, Shared Memory 용도로 사용될 수 있다. 주로 Cache 서버를 구현할 때 많이 쓴다.
#### Redis는 왜 Thread Safe한가?
싱글 스레드 방식이어서 연산을 하나씩 처리한다.
#### Redis는 왜 속도가 빠른가?
Key-Value 방식이므로 쿼리를 날리지 않고 결과를 얻을 수 있다.
#### Redis와 Memcached의 차이는 무엇인가?
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbBUbzk%2FbtrrEUM0nE8%2FE6w2P9XsgeHvgZxn9uXg00%2Fimg.png)

## 출처
https://redis.io/docs/about/

https://steady-coding.tistory.com/586