### ✏️ 웹 성능 개선하기

Web Server
- 매우 작은 데이터를 대량으로 전송하거나,하드디스크 읽기와 쓰기가 발생하지 않는 경우에 적합하다.
- 클라이언트와의 커넥션을 어떻게 효율적으로 관리할지 집중한다.
  - 캐싱 설정
  - CDN 사용하기
    - 여러 노드를 가진 네트워크에 컨텐츠를 저장하여 제공하는 프록시의 일종
  - keep-alive 설정
  - gzip 압축
  - 불필요한 다운로드 제거
  - 불필요한 작업을 지연로딩
  - 스크립트 병합하여 요청수 최소화
  - HTTP 프로토콜 개선
    - TCP 기반의 HTTP는 요청마다 Connection이 생성되어 연결 비용이 매우 크다.
      - 3way handshake
      - slow start

Application Server
- 복잡한 연산 처리, 동영상 데이터 처리, 데이터베이스 처리 등에 적합하다.
- 비즈니스 로직 개선, 조회 성능 개선에 집중한다.
  - 비즈니스 로직 개선
  - 조회 성능 개선
  - 비동기 처리
  - 데이터 캐싱 등
    - ETag
    - Cache-Control
      - 일부 파일에 변경이 발생한 경우 배포 시간 또는 버전 등을 활용해 URL을 변경한다.

### ✏️ Servlet이란?

- 자바 진영에서 동적인 웹 페이지를 구현하기 위해 만든 표준 모델
- Servlet 표준에 대한 interface 구현체는 Servlet Container(tomcat, jetty)가 제공함

요청, 응답 과정

1. 사용자가 브라우저를 통해 서버에 HTTP 요청을 보낸다.
2. 요청을 받은 서블릿 컨테이너는 request, response 두 객체를 만든다.
3. 서블릿 컨테이너는 요청을 처리할 수 있는 서블릿을 찾아서 스레드를 할당하고 request, response 객체를 전달한다
4. HTTP method 에 따라 doGet(), doPost() 등 메소드를 호출한다.
5. 요청 결과를 응답한 후 request, response 객체를 제거하고 자원을 반납한다.

### ✏️ SQL 기본

```
SELECT Country
    ,  COUNT(*) AS `고객 수` 
  FROM Customers 
 WHERE Country <> 'Norway'
 GROUP BY Country
HAVING COUNT(Country) = 1
 ORDER BY Country;

1. FROM에서 데이터 집합을 만든다.
2. WHERE는 FROM에서 만든 데이터 집합을 조건에 맞게 걸러낸다.
3. GROUP BY는 WHERE에서 필터링한 (조건에 맞는 데이터를 걸러낸) 데이터를 그룹화한다.
4. HAVING은 GROUP BY에서 집계한 데이터 집합을 다시 조건에 맞게 필터링한다.
5. SELECT에서는 그룹화하고 필터링한 데이터 집합을 집계한다.
6. 모두 진행한 이후, ORDER BY를 통해 집계한 데이터 집합을 정렬한다.
```

***서브쿼리***
```
SELECT (SELECT COUNT(*) FROM [Customers] WHERE Country = 'Germany') AS `Germany인 사람`
    ,  (SELECT COUNT(*) FROM [Customers] WHERE Country = 'Mexico') AS `Mexico인 사람`
```
SELECT 절에 추가하기

```
SELECT Country
    ,  CustomerName
  FROM (SELECT * FROM Customers);

```
FROM 절에 추가하기

```
SELECT * 
  FROM Products 
 WHERE Price = (SELECT MAX(Price) FROM Products);

```
WHERE 절에 추가하기

***조인문***
```
SELECT * 
  FROM Products 
  JOIN OrderDetails 
    ON Products.ProductID = OrderDetails.ProductID 
 WHERE Products.ProductID IN (1, 30)
```
ProductID 1과 30을 검색하기 위해 Products 테이블을 먼저 찾는데 이처럼 먼저 접근하는 테이블을 드라이빙 테이블, 검색 결과를 통해 뒤늦게 데이터를 검색하는 테이블을 드리븐 테이블이라고 한다.

가능하면 적은 결과가 반환될 것으로 예상되는 테이블을 드라이빙 테이블로 선정해야 하고 드라이빙 테이블의 추출 건수가 곧 드리븐 테이블의 액세스 반복 횟수가 되기 때문이다.

### ✏️ DB 최적화 대상

🎯 Client
- 복수 건의 레코드를 한번의 호출로 집합 처리하거나, 두 개 이상의 쿼리를 한 쿼리로 통합 처리함으로써 호출 수를 줄일 수 있다.
- JDBC Statement는 쿼리 문장 분석, 컴파일, 실행의 단계를 캐싱하는데 PreparedStatement는 처음 한 번만 세 단계를 거친 후 캐시에 담아서 재사용한다.
- DB Connection Pool을 사용하여 객체를 생성하는 부분에서 발생하는 대기시간을 줄이고 네트워크 부담을 줄일 수 있다.
- Fetchsize 조정하거나 Paging을 활용한다.

🎯 Database Engine
- 파일 시스템에 저장된 데이터가 조회되면 해당 데이터를 메모리에 저장해 이후 동일 데이터 조회 시 파일 시스템의 물리적인 입출력이 발생하지 않도록 한다.
- 서버 파라미터를 튜닝한다.

🎯 Filesystem
- SSD를 사용한다. 
- SQL을 최적화하여 필요 이상의 데이터 블록을 읽는 것을 방지한다.

[쿼리 최적화 : 빠른 쿼리를 위한 7가지 체크리스트](https://medium.com/watcha/%EC%BF%BC%EB%A6%AC-%EC%B5%9C%EC%A0%81%ED%99%94-%EC%B2%AB%EA%B1%B8%EC%9D%8C-%EB%B3%B4%EB%8B%A4-%EB%B9%A0%EB%A5%B8-%EC%BF%BC%EB%A6%AC%EB%A5%BC-%EC%9C%84%ED%95%9C-7%EA%B0%80%EC%A7%80-%EC%B2%B4%ED%81%AC-%EB%A6%AC%EC%8A%A4%ED%8A%B8-bafec9d2c073)

### ✏️ SQL 최적화 대상

참고 : [MySQL 내부 구조](https://brunch.co.kr/@jehovah/21)

🧰 쿼리 동작 방식

Query Caching
- SQL문이 Key, 쿼리의 실행결과가 Value인 Map 형태
- 데이터가 변경되면 모두 삭제해야 하는데 이는 동시 처리 성능 저하를 유발하고 많은 버그의 원인이 되어 MySQL 8.0으로 올라오면서 제거되었다.

Parsing
- 사용자로부터 요청된 SQL을 잘게 쪼개서 서버가 이해할 수 있는 수준으로 분리한다.

Preprocessor
- 해당 쿼리가 문법적으로 틀리지 않은지 확인하여 부정확하다면 여기서 처리를 중단한다.

Optimization
- 쿼리 분석 : Where 절의 검색 조건인지 Join 조건인지 판단한다.
- 인덱스 선택 : 각 테이블에 사용된 조건과 인덱스 통계 정보를 이용해 사용할 인덱스를 결정한다.
- 조인 처리 : 여러 테이블의 조인이 있는 경우 어떤 순서로 테이블을 읽을지 결정한다.

Handler (Storage Engine)
- MySQL 실행엔진의 요청에 따라 데이터를 디스크로 저장하고 디스크로부터 읽어오는 역할을 담당한다.
-  MySQL 엔진에서는 Storage 엔진으로부터 받은 레코드를 조인하거나 정렬하는 작업을 수행한다.

🧰 Index Range Scan과 Table Full Scan

Index Range Scan
- Random access와 Single Block I/O 방식으로 레코드 하나를 읽기 위해 매번 I/O가 발생한다.
- 읽을 데이터가 일정량을 넘으면 인덱스보다 Table Full Scan이 유리하다.
- Index를 사용할 경우 스캔 범위를 줄여 Random I/O 횟수를 감소시키는 것이 중요하다.

Table Full Scan
- Sequential access와 Multiblock I/O 방식으로 효율적으로 디스크를 읽는다.

🧰 Index 탐색 과정

MySQL 5.7 이상은 InnoDB를 사용하며 InnoDB는 B-Tree 인덱스를 기본적으로 사용한다.

수직적 탐색
- 루트 노드에서부터 탐색을 시작한다. 
- 루트 노드와 브랜치 노드는 인덱스 키와 자식 노드 정보로 구성된 페이지(단위)다.
- 리프 노드 스캔 시작점과 끝점을 찾기 위한 탐색 과정이다.

수평적 탐색
- 인덱스 리프 노드끼리는 양방향 Linked List이므로 서로 앞뒤 블록에 대한 주소값을 갖는다.
- 필요한 컬럼을 인덱스가 모두 갖고 있어 인덱스만 스캔하고 끝나는 경우도 있다.
- 그렇지 않을 경우 ROWID가 필요한데 데이터블럭 주소 + 로우 번호(블록내 순번)로 구성된다.
- InnoDB의 경우, Primary Key가 순차적으로 저장되어 있어 Data record의 물리적인 위치를 알 수 있다.
- ROWID가 가리키는 데이터 페이지를 버퍼풀에서 먼저 찾아보고 못찾을 때만 디스크에서 블록을 읽는다.(읽은 후에는 버퍼풀에 적재)

🎯 Index 튜닝

인덱스 스캔 효율화

🧰 랜덤 액세스 최소화
- 인덱스 스캔 후 테이블 레코드를 액세스할 때, 랜덤 I/O 횟수를 줄이는 것을 의미한다.
- 인덱스 스캔 보다 랜덤 액세스를 최소화하는 것이 더 효과적이다.

### ✏️ DB 서버 튜닝

🎯 메모리 튜닝

***Thread***

MySQL은 커넥션마다 하나의 Thread를 생성하여 요청을 처리하는데 Thread를 메모리에 할당하고 해제하는데 비용이 크므로 이를 줄일 필요가 있다.
```
# 현재 쓰레드(연결) 개수 확인
mysql> SELECT * FROM performance_schema.threads
mysql> SHOW STATUS LIKE '%THREAD%';
```

thread_cache_size는 지나치게 높여둘 필요는 없으며 일반적으로 threads_connected의 피크 치보다 약간 낮은 수치 정도를 설정하는 것이 좋다. 이를 통해 쓰레드가 생성되고 소멸되면서 겪게 되는 메모리, 각종 자원, 시간 등의 낭비를 줄일 수 있다.

***Caching***

버퍼는 mysqld에서 내부적으로 하나만 확보되는 Global Buffer와 Thread(Connection)별로 확보되는 Thread Buffer가 있다. Thread Buffer에 많은 메모리를 할당하면 성능이 올라가지만, 설정값 * Connection 수만큼 확보하므로 Connection이 갑자기 늘어나면 메모리가 부족해져 swap이 발생할 수도 있다.
- innodb_buffer_pool_size : InnoDB의 데이터나 인덱스를 캐시하기 위한 메모리상의 영역, 글로벌 버퍼이므로 크게 할당할 것을 권한다. 보통 시스템 전체 메모리의 80% 수준으로 설정한다. (최대 512MB)
- key_buffer_size : 인덱스를 메모리에 저장하는 버퍼 크기를 의미하며, 보통 총 메모리 크기의 25% 정도로 설정한다.
- Key Buffer 사용률은 1 - ((Key_reads/Key_read_requests) * 100)이다. 90% 이상일 경우 key_buffer_size가 효율적으로 설정되어 있다고 판단할 수 있다.

```
mysql> SHOW STATUS LIKE '%key%';
```

🎯 커넥션 튜닝
```
mysql> SHOW VARIABLES LIKE '%max_connection%';
mysql> SHOW STATUS LIKE '%CONNECT%';
mysql> SHOW STATUS LIKE '%CLIENT%';
```

connect_timeout
- MySQL이 클라이언트로부터 접속 요청을 받는 경우 몇 초까지 기다릴지를 설정하는 변수로, 기본 값은 5초이며 일반적으로 수정할 필요는 없다.

Interactive_timeout
- ‘mysql>’과 같은 콘솔이나 터미널 상에서의 클라이언트 접속을 의미하며, 기본 값으로 8시간이 잡혀 있으나 1시간 정도로 낮추는 것이 좋다.

wait_timeout
- 접속한 후 쿼리가 들어올 때까지 기다리는 시간으로, 접속이 많은 DBMS에서는 이 값을 낮춰 sleep 상태의 Connection들을 정리하여 전체 성능을 향상시킬 수 있습니다. 하지만 값을 너무 낮추게 되면 지나치게 잦은 커넥션이 발생할 수 있으므로, 보통 15~20 사이의 값을 설정합니다. Aborted client는 2% 아래인 것이 바람직한 상태다.

max_connections
- 서버가 허용하는 최대한의 커넥션 수. 서버의 사양에 따라 달라질 수 있으며 일반적으로 120~250개 정도로 설정한다. 하지만 접속이 많고 고용량 서버의 경우 1000개 정도의 높은 값을 설정하는 것도 가능하니, Too many connection 에러가 발생하지 않도록 적절한 값을 설정하는 것이 중요하다.

back_log
- max_connection 설정값 이상의 접속이 발생할 때 얼마만큼의 커넥션을 큐에 보관할지에 대한 설정 값으로, 기본 값은 50이며 접속이 많은 서버의 경우 이 값을 늘릴 필요가 있다.
