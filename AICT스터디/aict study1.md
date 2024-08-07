![image-20240807182020282](./image-20240807182020282.png)

![image-20240807182107749](./image-20240807182107749.png)

## ■ 업무용 응용프로그램 개발 방안

###  \- A고객사 시스템의 채널 확대 및 사용자 증가에 따라 발생하는 인증 및 세션 관리 문제 해결을 위한 인증 방식 개선 방안



1. OAuth 2.0과 OpenID Connect 도입

OAuth 2.0과 OpenID Connect(OIDC)를 도입하여 인증 및 권한 관리를 분리하는 방식은 사용자의 인증과 애플리케이션의 권한 부여를 더 안전하고 간편하게 관리할 수 있습니다.

OAuth 2.0: 권한 부여 프레임워크로, 클라이언트(애플리케이션)가 자원 소유자(사용자)를 대신하여 자원 서버(예: API 서버)에 접근할 수 있도록 합니다.

OpenID Connect: OAuth 2.0 위에 구축된 인증 프로토콜로, 사용자 인증을 처리하고 사용자 정보를 안전하게 공유할 수 있습니다.

```
OAuth(OAuth2.0)란 무엇일까? 소셜로그인이 작동하는 법
```

출처: https://matamong.tistory.com/entry/OAuthOAuth20란-무엇일까-소셜로그인이-작동하는-법 [마타몽의 개발새발제발:티스토리]

https://matamong.tistory.com/entry/OAuthOAuth20%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C-%EC%86%8C%EC%85%9C%EB%A1%9C%EA%B7%B8%EC%9D%B8%EC%9D%B4-%EC%9E%91%EB%8F%99%ED%95%98%EB%8A%94-%EB%B2%95



2. 복수 인증 단계(Multi-Factor Authentication, MFA)

MFA를 추가하여 보안을 강화할 수 있습니다. 이는 특히 중요한 사용자나 민감한 데이터를 다룰 때 매우 유용합니다. 예를 들어, SMS, 이메일, 또는 전용 MFA 앱을 통한 코드를 추가 입력하게 할 수 있습니다.

3. 세션 관리 개선

세션 관리는 특히 사용자 증가 시 중요한 이슈입니다. 다음과 같은 방안을 고려할 수 있습니다:

세션 타임아웃(Timeouts): 적절한 세션 타임아웃을 설정하여 오래된 세션을 자동으로 종료하는 방식.

세션 스토리지: 분산 세션 저장소(예: Redis, Memcached)를 사용하여 세션 데이터를 중앙에서 관리하므로, 여러 서버 간에 세션 상태를 일관되게 유지할 수 있습니다.

4. 토큰 기반 인증

토큰 기반 인증 방법을 활용하여 세션 관리 문제를 해결합니다.

JWT(JSON Web Tokens): 사용자의 상태를 서버에 저장하지 않는 stateless한 방식으로, 토큰을 클라이언트 측에 저장하고 주고 받음으로써 인증 상태를 관리할 수 있습니다.

토큰 만료 및 갱신: 액세스 토큰과 리프레시 토큰을 사용하여 토큰의 유효 기간을 관리하고, 일정 시간 후 또는 액세스 토큰이 만료될 때 새로운 토큰으로 갱신.

5. 중앙 인증 서비스(Identity Provider, IdP)

SAML, OAuth 등 표준 프로토콜을 사용하는 중앙 인증 서비스를 도입하여, 여러 채널에서 일관된 인증 및 권한 부여를 관리할 수 있습니다.

사례: Keycloak, Okta, Auth0 등의 IdP를 활용하여 사회적 로그인(Social Login)과 같은 기능을 제공하고 사용자의 편의성을 높일 수 있습니다.

6. 비밀번호 정책 강화

비밀번호 정책을 강화하여 보안을 더욱 단단히 합니다.



비밀번호 복잡성 요구: 대문자, 소문자, 특수문자, 숫자를 포함한 복잡한 비밀번호 사용.

비밀번호 이력 저장: 일정 기간 동안 이전 비밀번호를 사용하지 못하게 하는 기능.



7. 모니터링 및 로깅

실시간 모니터링: 로그인 시도, 실패, 비정상적인 세션 행동 등을 모니터링하여 빠르게 대응.

로깅 및 알림: 중요한 인증 및 세션 이벤트를 로깅하여 이슈가 발생할 시 알림을 받을 수 있게 설정.

이러한 방안들은 각각의 장점을 고려하여 최적의 조합으로 사용할 수 있습니다. 특히, 사용자 편의성과 보안의 균형을 맞추는 것이 중요합니다.



사용자 인증

1. 사용자 인증(Authentication)과 권한 부여(Authorization)에 대한 이해

 https://velog.io/@m_jae/Here-we-go

다중 서버환경에서의 세션 불일치 문제와 해결방법

세션관리

[https://velog.io/@ddangle/%EB%8B%A4%EC%A4%91-%EC%84%9C%EB%B2%84%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C%EC%9D%98-%EC%84%B8%EC%85%98-%EB%B6%88%EC%9D%BC%EC%B9%98-%EB%AC%B8%EC%A0%9C%EC%99%80-%ED%95%B4%EA%B2%B0%EB%B0%A9%EB%B2%95](https://velog.io/@ddangle/다중-서버환경에서의-세션-불일치-문제와-해결방법)

### \- A고객사 비즈니스 유연성과 성능 관점에서 상품을 관리하기 위한 데이터 모델을 새롭게 설계하고 설계 사유 제시

데이터 모델 설계

1. 제품 테이블 (Products)

sql 



CREATE TABLE Products (

  ProductID INT PRIMARY KEY AUTO_INCREMENT,

  Name VARCHAR(255) NOT NULL,

  Description TEXT,

  CategoryID INT,

  Price DECIMAL(10, 2) NOT NULL,

  Stock INT NOT NULL DEFAULT 0,

  CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  UpdatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  FOREIGN KEY (CategoryID) REFERENCES Categories(CategoryID)

);

2. 카테고리 테이블 (Categories)

sql 



CREATE TABLE Categories (

  CategoryID INT PRIMARY KEY AUTO_INCREMENT,

  Name VARCHAR(255) NOT NULL,

  Description TEXT

);

3. 가격 히스토리 테이블 (PriceHistory)

sql 



CREATE TABLE PriceHistory (

  PriceHistoryID INT PRIMARY KEY AUTO_INCREMENT,

  ProductID INT,

  OldPrice DECIMAL(10, 2) NOT NULL,

  NewPrice DECIMAL(10, 2) NOT NULL,

  ChangeDate TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  FOREIGN KEY (ProductID) REFERENCES Products(ProductID)

);

4. 재고 움직임 테이블 (InventoryMovements)

sql 



CREATE TABLE InventoryMovements (

  MovementID INT PRIMARY KEY AUTO_INCREMENT,

  ProductID INT,

  ChangeAmount INT NOT NULL,

  MovementDate TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  Reason VARCHAR(255),

  FOREIGN KEY (ProductID) REFERENCES Products(ProductID)

);

5. 기타 관련 테이블 (예: Promotion, Reviews 등)

이와 더불어 프로모션, 리뷰, 고객 등 관련 데이터 테이블도 필요할 수 있습니다.



설계 사유

1. 유연성

Products와 Categories 테이블 분리: 제품과 카테고리를 분리하여 관리하면 카테고리마다 제품을 다양하게 추가, 수정, 관리하는 데 유연성을 확보할 수 있습니다.

가격 히스토리 관리: PriceHistory 테이블을 두어 가격 변동 내역을 관리함으로써 가격 변화에 따른 데이터를 추적하고 분석할 수 있습니다. 이는 가격 정책 변경이나 프로모션을 분석하는 데 도움이 됩니다.

2. 성능

정규화(Normalization): 데이터 정규화를 통해 반복 데이터를 최소화하여 저장 공간을 절약하고 데이터 일관성을 유지할 수 있습니다.

인덱스: ProductID와 같은 기본 키에 인덱스를 적용하면 조회 성능이 향상됩니다. 필요한 컬럼에 추가 인덱스를 설정하여 성능을 최적화할 수 있습니다.

타임스탬프 활용: 생성일과 수정일을 타임스탬프를 사용해 기록함으로써 빠른 시간 내 데이터 변경 내역을 추적하고 정렬할 수 있습니다.

3. 데이터 무결성

외래 키 제약조건: 외래 키 제약 조건을 통해 데이터 무결성을 유지합니다. 예를 들어, 재고 움직임은 특정 제품에만 적용될 수 있도록 외래 키를 설정합니다.

검증된 참조(Reference Integrity): Parent 테이블(예: Categories)의 레코드가 삭제되면, 참조하고 있는 자식 테이블(예: Products)의 데이터도 연쇄 삭제 또는 업데이트할 수 있게 설정하여 데이터 무결성을 보호합니다.

4. 확장 가능성

샤딩(Sharding): 사용자 수와 데이터 크기가 크게 증가할 경우 여러 데이터베이스 인스턴스에 데이터를 분산 저장하는 샤딩을 고려할 수 있습니다. 이는 성능과 가용성을 높이는 데 효과적입니다.

캐싱(Caching): Redis 또는 Memcached와 같은 캐싱 솔루션을 사용하여 빈번하게 접근하는 데이터를 캐시하면 응답 속도를 크게 개선할 수 있습니다.

5. 모니터링 및 분석

로그 테이블: 주요 활동(예: 상품 가격 변경 등)에 대한 로그 테이블을 별도로 둘 수 있습니다. 이는 실시간 모니터링과 데이터 분석에 도움이 됩니다.

지표 생성: 재고 변동, 판매 추이 등 다양한 비즈니스 인텔리전스 도구를 활용해 지표를 생성하고 관리할 수 있습니다.

이처럼 여러 관점에서 설계된 데이터 모델은 비즈니스의 유연성과 성능을 극대화할 수 있도록 도와줍니다. 고객사의 특수한 요구사항에 따라 일부 테이블과 컬럼을 추가하거나 조정할 수도 있습니다

### \- 예약 처리 프로세스와 데이터 모델에서 발생하고 있는 동시성 이슈의 해결 방안 제시



예약 처리 시스템에서 동시성(Concurrency) 이슈는 중요한 문제입니다. 이는 다수의 사용자가 동일한 자원을 동시에 예약하려고 시도할 때 발생할 수 있습니다. 이러한 문제를 효과적으로 해결하기 위한 방법을 몇 가지 제안합니다.



1. 데이터 모델 개선

동시성 문제를 해결하기 위해 데이터 모델을 적절히 설계하는 것이 필수적입니다. 이때 Locking Mechanism을 포함하여 동시성 제어를 강화할 수 있습니다.



테이블 구조 예시:

sql 



CREATE TABLE Reservations (

  ReservationID INT PRIMARY KEY AUTO_INCREMENT,

  ResourceID INT NOT NULL,

  UserID INT NOT NULL,

  StartTime DATETIME NOT NULL,

  EndTime DATETIME NOT NULL,

  Status ENUM('PENDING', 'CONFIRMED', 'CANCELLED') DEFAULT 'PENDING',

  CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  UpdatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  CONSTRAINT UC_ResourceBooking UNIQUE (ResourceID, StartTime, EndTime)

);

이 구조에서 ResourceID, StartTime, EndTime의 고유 제약조건(Unique Constraint)이 겹치는 예약을 방지합니다.



2. 동시성 제어 기법

2.1. 비관적 잠금(Pessimistic Locking)

비관적 잠금은 한 트랜잭션이 자원을 잠글 때 다른 트랜잭션이 해당 자원에 접근하지 못하도록 막는 방식입니다. 이는 동시성 문제가 자주 발생할 가능성이 높을 때 유효합니다.



sql 



-- 잠금 설정 예시

BEGIN;

SELECT * FROM Reservations WHERE ResourceID = ? AND StartTime = ? AND EndTime = ? FOR UPDATE;

-- 예약 처리 로직

INSERT INTO Reservations (ResourceID, UserID, StartTime, EndTime, Status) VALUES (?, ?, ?, ?, 'CONFIRMED');

COMMIT;

2.2. 낙관적 잠금(Optimistic Locking)

낙관적 잠금은 자원 접근 충돌이 드물 것으로 예상되지만, 발생했을 경우 이를 감지하고 조치를 취하는 방식입니다. 일반적으로 버전 번호나 타임스탬프를 활용합니다.



sql 



-- 낙관적 잠금용 테이블 예시

CREATE TABLE Reservations (

  ReservationID INT PRIMARY KEY AUTO_INCREMENT,

  ResourceID INT NOT NULL,

  UserID INT NOT NULL,

  StartTime DATETIME NOT NULL,

  EndTime DATETIME NOT NULL,

  Version INT NOT NULL DEFAULT 1, -- 버전 번호

  Status ENUM('PENDING', 'CONFIRMED', 'CANCELLED') DEFAULT 'PENDING',

  CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  UpdatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP

);



-- 업데이트 시 버전 번호 검증

UPDATE Reservations SET Version = Version + 1, Status = 'CONFIRMED'

WHERE ResourceID = ? AND StartTime = ? AND EndTime = ? AND Version = ?;

2.3. 타임스탬프 기반 제어

타임스탬프 기반 제어는 낙관적 잠금과 유사하게 타임스탬프를 사용하여 데이터를 검증합니다.



sql 



-- 예약 생성 시 타임스탬프 기록

INSERT INTO Reservations (ResourceID, UserID, StartTime, EndTime, Status, CreatedAt) 

VALUES (?, ?, ?, ?, 'PENDING', CURRENT_TIMESTAMP);



-- 업데이트 시 타임스탬프 검증

UPDATE Reservations SET Status = 'CONFIRMED', UpdatedAt = CURRENT_TIMESTAMP

WHERE ResourceID = ? AND StartTime = ? AND EndTime = ? AND CreatedAt = ?;

3. 비동기 처리 및 큐

예약 요청을 비동기로 처리하여 동시성 문제를 해소할 수 있습니다. 이를 위해 메시지 큐 시스템(예: RabbitMQ, Apache Kafka)을 활용할 수 있습니다.



메시지 큐 기반 처리

예약 요청을 큐에 추가

예약 처리 워커가 큐에서 예약 요청을 가져와 처리

처리 완료 후 결과를 사용자에게 반환

이러한 접근법은 시스템의 부하를 분산시키고 동시성 이슈를 줄이는 데 효과적입니다.



4. 분산 잠금 관리

분산 환경에서 동시성 문제를 해결하기 위해 분산 잠금 관리 도구를 사용할 수 있습니다. Redis를 사용한 분산 잠금 관리 예시입니다.



python 



import redis

import uuid

from contextlib import contextmanager



\# Redis 클라이언트 설정

redis_client = redis.StrictRedis(host='localhost', port=6379, db=0)

lock_key = 'reservation_lock'



@contextmanager

def distributed_lock(lock_key, lock_timeout=10):

  lock_id = str(uuid.uuid4())

  \# 잠금 시도

  while not redis_client.set(lock_key, lock_id, ex=lock_timeout, nx=True):

​    pass

  try:

​    yield

  finally:

​    \# 잠금 해제

​    if redis_client.get(lock_key) == lock_id:

​      redis_client.delete(lock_key)



\# 예약 처리 예시

with distributed_lock(lock_key):

  \# 예약 처리 로직

  process_reservation()

이와 같은 다양한 방법을 통해 예약 처리 시스템에서 발생할 수 있는 동시성 문제를 효과적으로 해결할 수 있습니다. 각각의 방법이 가진 장단점을 고려하여 A고객사 시스템에 가장 적합한 방법을 선택하는 것이 중요합니다.



[https://velog.io/@akfls221/%EB%8F%99%EC%8B%9C%EC%84%B1%EC%97%90-%EB%8C%80%ED%95%9C-%ED%95%B4%EA%B2%B0%EB%B0%A9%EB%B2%95%EC%9D%84-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90](https://velog.io/@akfls221/동시성에-대한-해결방법을-알아보자)

동시성에 대한 해결방법을 알아보자

https://brewagebear.github.io/concurrency-distributed-transaction-with-tcc/



### 국가별 결제 수단

https://www.nextunicorn.kr/insight/9a8f3cbac42714e4