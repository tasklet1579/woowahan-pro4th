✔️ 테스트 대상

```
지하철 노선도 서비스

기능
- 역 관리
- 노선 관리
- 구간 관리
- 구간 검색

구성
- 프록시 (Nginx) ↔ 웹 (Tomcat) ↔ 데이터베이스 (MySQL)

* 성능 부하 테스트는 Bastion 서버에서 진행함
```

✔️ 목표값 계산

```
1일 사용자 수 : 10,000
피크 시간대의 집중률 : 최대 트래픽이 평균 트래픽 보다 3배 높다고 가정
1명당 1일 평균 요청 건수 : 50

Throughput : 시간당 평균, 최대 처리량 
- 1일 사용자 수 X 1명당 1일 평균 요청 건수 = 1일 총접속 수 = 500,000
- 1일 총접속 수 / 86,400s = 1일 평균 RPS = 5.787
- 1일 평균 RPS X 피크 시간대의 집중률 = 1일 최대 RPS = 17.361

Latency : 지연 시간
- 100ms 이하

VUser : 가상 사용자
- Request Rate : measured by the number of requests per second
- R : the number of requests per VU iteration
- T : a value larger than the time needed to complete a VU iteration

T = (R * http_req_duration) (+L) ; 내부망에서 테스트할 경우 예상 latency를 추가한다
VUser = (목표 rps * T) / R

예를 들어, 8개의 요청이 있고 왕복 시간이 0.5s, 지연 시간을 1s로 한다면 
평균 트래픽 VUser = 5.787 * 5 / 8 = 3.616 → 4
최대 트래픽 VUser = 17.361 * 5 / 8 = 10.850 → 11
```

✔️ 목표값 설정

```
Smoke
- VUser : 1~2
- Throughput : ~17.361
- Latency : ~100ms
- 소유 시간 : 10분

Load
- 평균 VUser : 4
- 최대 VUser : 11
- Throughput : ~17.361
- Latency : ~100ms
- 소유 시간 : 30분

Stress
- VUser : 점진적으로 증가시킨다
- 측정 시간 : 5분
```

✔️ Smoke, Load, Stress 테스트 스크립트

테스트 순서
- 랜딩 페이지 접속
- 아이디와 패스워드를 사용하여 로그인
- 지하철역 조회
- 시작역부터 출발역까지의 최단거리 경로 검색
- 즐겨찾기 등록

Smoke
- k6 run --out influxdb=http://localhost:8086/k6 smoke.js
```
import http from 'k6/http';
import {check, sleep} from 'k6';

export let options = {
    vus: 2,
    duration: '10m',
    
    thresholds: {
        http_req_duration: ['p(99)<1500'], // 99% of requests must complete below 1.5s
    },
};

const BASE_URL = 'https://www.tasklet1579.p-e.kr/';
const USERNAME = 'tasklet1579@next.co.kr';
const PASSWORD = 'test1234';

export default () => {
    let homeUrl = `${BASE_URL}`;
    let lendingPageResponse = http.get(homeUrl);
    check_lending_page(lendingPageResponse);
    
    let loginResponse = login();
    check_login_access_token(loginResponse);
    
    sleep(1);
};

function login() {
    var body = JSON.stringify({
        email: USERNAME,
        password: PASSWORD,
    });
    var headers = {
        headers: {
            'Content-Type': 'application/json',
        },
    };

   return http.post(`${BASE_URL}/login/token`, body, headers);
}

function check_lending_page(lendingPageResponse) {
    check(lendingPageResponse, {
        '랜딩 페이지 점검': (response) => response.status === 200
    });
}

function check_login_access_token(loginResponse) {
    check(loginResponse, {
        '로그인 후 토큰 획득': (response) => response.json('accessToken') !== '',
    });
}
```

Load
- k6 run --out influxdb=http://localhost:8086/k6 load.js
```
import http from 'k6/http';
import {check, sleep} from 'k6';
import { URL } from 'https://jslib.k6.io/url/1.0.0/index.js';

export let options = {
    stages: [
        { duration: '1m', target: 1 },
        { duration: '2m', target: 5 }, 
        { duration: '3m', target: 7 }, 
        { duration: '5m', target: 9 }, 
        { duration: '5m', target: 11 },
        { duration: '5m', target: 11 },
        { duration: '5m', target: 6 }, 
        { duration: '4m', target: 0 }, 
    ],
    thresholds: {		
        http_req_duration: ['p(99)<1500'], // 99% of requests must complete below 1.5s
    },
};

const BASE_URL = 'https://www.tasklet1579.p-e.kr/';
const USERNAME = 'tasklet1579@next.co.kr';
const PASSWORD = 'test1234';

export default () => {
    let homeUrl = `${BASE_URL}`;
    let lendingPageResponse = http.get(homeUrl);
    check_lending_page(lendingPageResponse);
    
    let loginResponse = login();
    check_login_access_token(loginResponse);
    
    let stationsUrl = `${BASE_URL}/stations`;
    let stationsResponse = http.get(stationsUrl);
    check_stations_size(stationsResponse);
    
    let findPathUrl = new URL(`${BASE_URL}/paths`);
    findPathUrl.searchParams.append('source', 1);
    findPathUrl.searchParams.append('target', 15);
    
    let findPathResponse = http.get(findPathUrl.toString());
    check_find_path(findPathResponse);
    
    let favoriteResponse = add_favorite(loginResponse.json('accessToken'));
    check_favorite(favoriteResponse);
    
    sleep(1);
};

function login() {
    var body = JSON.stringify({
        email: USERNAME,
        password: PASSWORD,
    });
    var headers = {
        headers: {
            'Content-Type': 'application/json',
        },
    };
    
    return http.post(`${BASE_URL}/login/token`, body, headers);
}

function add_favorite(accessToken) {
    var body = JSON.stringify({
        source: 1,
        target: 15,
    });
    let headers = {
        headers: {
            Authorization: `Bearer ${accessToken}`,
            'Content-Type': 'application/json',
        },
    };
    
    return http.post(`${BASE_URL}/favorites`, body, headers);
}

function check_lending_page(lendingPageResponse) {
    check(lendingPageResponse, {
        '랜딩 페이지 점검': (response) => response.status === 200
    });
}

function check_login_access_token(loginResponse) {
    check(loginResponse, {
        '로그인 후 토큰 획득': (response) => response.json('accessToken') !== '',
    });
}

function check_stations_size(stationsResponse) {
    check(stationsResponse, {
        '모든 지하철역 개수 조회': (response) => response.body.length > 1,
    });
    
    // console.log(stationsResponse.json());
}

function check_find_path(findPathResponse) {
    check(findPathResponse, {
        '지하철 최단거리 경로 조회': (response) => response.json('stations').length > 1,
    });
}

function check_favorite(favoriteResponse) {
    check(favoriteResponse, {
        '지하철 경로 즐겨찾기': (response) => response.status === 201
    });
}
```

Stress
- k6 run --out influxdb=http://localhost:8086/k6 stress.js
```
export let options = {
    stages: [
        { duration: '1m', target: 64 },
        { duration: '1m', target: 128 },
        { duration: '1m', target: 256 },
        { duration: '1m', target: 512 },
        { duration: '1m', target: 1024 },
    ],
    thresholds: {
        http_req_duration: ['p(99)<1500'], // 99% of requests must complete below 1.5s
    },
};
```
