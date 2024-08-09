https://github.com/HyunSun-Song/edu

https://github.com/HyunSun-Song/edu/blob/main/README.md

보상트랜잭션
https://blog.naver.com/PostView.nhn?blogId=kyy627&logNo=222306108349

사가의 구조

보상 가능 트랜잭션(compensatable transaction): 보상 트랜잭션으로 롤백 가능한 트랜잭션
피벗 트랜잭션(pivot transaction): 사가의 진행/중단 지점. 피봇 트랜잭션이 커밋되면 사가는 완료될 때까지 실행. 피봇 트랜잭션은 보상 가능 트랜잭션, 재시도 가능한 트랜잭션 그 어느 쪽도 아니지만, 최종 보상 가능 트랜잭션 또는 최초 재시도 가능 트랜잭션이 될 수 있음
재시도 가능 트랜잭션(retriable transaction): 피봇 트랜잭션 직후의 트랜잭션. 반드시 성공

[마이크로서비스 패턴] 4. 트랜잭션 관리: 사가
https://velog.io/@daehoon12/%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4-%ED%8C%A8%ED%84%B4-4.-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EA%B4%80%EB%A6%AC-%EC%82%AC%EA%B0%80

마이크로서비스 패턴: 핵심패턴만 빠르게 이해하기
https://happycloud-lee.tistory.com/154

[E-commerce] 주문 결제 트랜잭션 단위를 어떻게 가져가야 할까
https://seungjjun.tistory.com/322

)

[추천시스템] 추천시스템 A to Z : 추천 알고리즘의 종류
https://calmmimiforest.tistory.com/100

추천 시스템의 종류
⬛ Contents-based Recommender System (컨텐츠 기반 추천시스템)

사용자가 과거에 좋아했던 아이템을 파악하고 그 아이템과 비슷한 아이템을 추천한다.
예시) '부산행'에 5점 평점을 준 사용자 → '명량' 보다는 '반도'를 더 좋아할 것이다.
 ① 사용자가 과거에 접한 아이템이면서 만족한 아이템
 ② 사용자가 좋아했던 아이템 중 일부 또는 전체와 비슷한 아이템 선정
 ③ 선정된 아이템을 유저에게 추천

	장점	단점
Content Based	1. 첫번재 평가자에 대한 딜레마 없음
(Cold-start dilemma X)
2. 다른 사용자의 도움 없이 적절한 추천 받음
(User liberty)
3. 새로운 아이템에 대한 추천을 위해 해당 아이템을 rating할 필요가 없음
4. Scarcity 영향 없음
5. 사용자의 프로필 변경 및 업데이트를 모니터링하여 많은 사용자의 관심도나 관심도 변화 문제를 해결하는 데 실용적이다.
6. 더 적은 데이터로 동작	1. 제한된 콘텐츠 분석
2. Overspecialization (시스템은 사용자 프로필에 대해 높은 점수를 받은 항목만 추천할 수 있음. 사용자는 이미 평가된 항목과 유사한 항목으로 추천)
3. 너무 다르거나 비슷한 아이템을 설명할 수 없음.
4. 복잡한 관계를 해석할 수 없음
5. 계산 비용이 많이 듦
6. 동의어 또는 동음이의어 때문에 위기에 직면
7. 일부 유형의 아이템 콘텐츠는 분석하기 쉽지 않음
8. 사용자는 자신의 과거 경험과 유사한 항목만 받을 수 있음 (사용자를 놀라게 할 수 없음)
Collaborative Filtering	1. 콘텐츠 정보가 필요로 하지 않음
2. 사용자는 이전에 연락한 적이 없지만 관심이 있는 항목을 받을 수 있음(사용자를 놀라게 할 수 있음)
3. 다른 사용자의 점수는 시스템에 변경 사항이 포함되어 있기 때문에, deterministic 결과로 사용하진 않음 (시간이 지남에 따라 향상된 Adaptive Quality)


1. rating 데이터 필요
2. Sparse 데이터로 추천할 수 없음
3. 확장성 문제
4. Cold-start dilemma
(신규 사용자(커뮤니티) 및 신규 아이템 문제)
5. 엄청나게 많은 아이템 세트와 적은 수의 사용자로 인해 성능이 저하됨
6. 품질은 대량의 데이터 수집에 달려 있음
7. 취향이 특이한 유저에게 추천하기 어렵다
8. 선호도가 변하거나 포함된 사용자를 클러스터링하고 분류하는 것은 어려움

[쇼핑몰 만들기] 쇼핑몰 ERD 설계
https://velog.io/@1afterwon/%EC%87%BC%ED%95%91%EB%AA%B0-%EB%A7%8C%EB%93%A4%EA%B8%B0-%EC%87%BC%ED%95%91%EB%AA%B0-ERD-%EC%84%A4%EA%B3%84

https://velog.velcdn.com/images/1afterwon/post/e71ae49c-e6d6-4eea-8ec5-f52ebc7f34a4/image.png

게임사이트 ERD 설계
https://velog.io/@mandarine_punch/Project-NNgame-%EA%B2%8C%EC%9E%84-%ED%8C%90%EB%A7%A4-%EC%82%AC%EC%9D%B4%ED%8A%B8
https://ryulstudy.tistory.com/67

DR시스템 구축
https://blog.hectodata.co.kr/dr/
AWS 클라우드 DR
https://aws.amazon.com/ko/blogs/tech/disaster-recovery-dr-architecture-on-aws-part-i-strategies-for-recovery-in-the-cloud-1/
https://waspro.tistory.com/784


https://github.com/HyunSun-Song/edu

https://github.com/HyunSun-Song/edu/blob/main/README.md

보상트랜잭션
https://blog.naver.com/PostView.nhn?blogId=kyy627&logNo=222306108349

사가의 구조

보상 가능 트랜잭션(compensatable transaction): 보상 트랜잭션으로 롤백 가능한 트랜잭션
피벗 트랜잭션(pivot transaction): 사가의 진행/중단 지점. 피봇 트랜잭션이 커밋되면 사가는 완료될 때까지 실행. 피봇 트랜잭션은 보상 가능 트랜잭션, 재시도 가능한 트랜잭션 그 어느 쪽도 아니지만, 최종 보상 가능 트랜잭션 또는 최초 재시도 가능 트랜잭션이 될 수 있음
재시도 가능 트랜잭션(retriable transaction): 피봇 트랜잭션 직후의 트랜잭션. 반드시 성공

[마이크로서비스 패턴] 4. 트랜잭션 관리: 사가
https://velog.io/@daehoon12/%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4-%ED%8C%A8%ED%84%B4-4.-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EA%B4%80%EB%A6%AC-%EC%82%AC%EA%B0%80

마이크로서비스 패턴: 핵심패턴만 빠르게 이해하기
https://happycloud-lee.tistory.com/154

[E-commerce] 주문 결제 트랜잭션 단위를 어떻게 가져가야 할까
https://seungjjun.tistory.com/322
