-고객상황
GenieTV는 사용자가 좋아할 만한 영화를 추천하는 기능을 강화하고자 합니다. 이를 위해, 사용자가 Netflix에서 시청한 영화 정보를 연동하여 GenieTV에서 개인 맞춤형 영화 추천 서비스를 제공하려고 합니다. GenieTV는 고객의 시청 이력을 분석하여 더욱 정교한 추천 시스템을 개발하고자 하비다,

- 업무요건
1. 사용자 관리
사용자는 GenieTV에 등록되어야 하며, 사용자 정보는 고유ID, 이름, 연락처 정보(전화번호, 이메일)로 구성됩니다.
Netfix 사용자와 GenieTV 사용자가 동일인인지 확인하고, 사용자 연동을 위한 매핑 정보를 관리합니다.
2. Netflix 시청 기록 관리
사용자가 Netflix에서 시청한 영화 정보를 가져와야 합니다.
시청기록은 사용자ID, 영화ID, 시청 시작 시간, 시청 종료 시간, 평가(별점) 등의 정보를 포함합니다.
3. 영화 정보 관리
영화 정보는 고유ID, 제목, 장르, 출시 연도, 감독, 배우 목록 등의 정보를 포함합니다.
Netflix와 GenieTV의 영화 ID 매핑 정보를 관리합니다.
4. 영화 추천 관리
Netflix 시청 기록을 바탕으로 GenieTV에서 영화를 추천해야 합니다.
추천 알고리즘은 사용자가 시청한 영화의 장르, 배우, 감독 등을 기반으로 유사한 영화를 추천합니다.
추천 기록은 사용자ID, 추천된 영화ID, 추천 일자 등의 정보를 포함합니다.

- 업무시나리오
1. 사용자 등록 및 매핑
사용자가 GenieTV에 회원 가입을 합니다.
Netflix 사용자와 GenieTV 사용자가 동일한 경우, 사용자 정보를 매핑합니다.
2. Netflix 시청 기록 연동
주기적으로 사용자의 Netflix 시청 기록을 가져와 GenieTV 데이터베이스에 저장합니다.
3. 영화 정보 매핑
Netflix와 GenieTV에서 제공하는 영화 정보를 매핑합니다.
4. 영화 추천
Netflix 시청 기록을 바탕으로 GenieTV에서 유사한 영화를 추천합니다.
추천된 영화 목록을 사용자에게 제공합니다.

아래 ERD는 Netflix 시청 기록을 활용한 GenieTV 영화 추천 시스템(이하 시스템) 의 주요 엔티티 목록과 속성의 일부입니다.
주요 엔티티 목록과 함께, 제시된 제약사항을 확인하시고, 논리 모델 ERD를 작성해주세요.

주요 엔티티 속성, 속성 목록(일부)
- 사용자 : 사용자ID, 이름, 연락처
- Netflix t시청 기록 : 사용자, 영화정보, 시청시작시간, 시청종료시간, 평점
- Netflix 영화정보 : 제목, 장르, 출시연도, 감독, 배우 목록
- GenieTV 영화정보 : 제목, 장르, 출시연도, 감독, 배우 목록
- 영화정보 매핑 : Netflix 영화정보, GenieTV 영화정보
- 영화 추천 : 사용자, 영화정보, 추천일자

제약 사항
- 응시자가 생각하는 시스템의 형태에 따라 엔티티와 속성은 임의로 수정하여 작성하여도 됩니다.
- ERD 표기법은 자유롭게 선택하시면 됩니다.

### 논리 모델 설계 및 ERD 작성

GenieTV와 Netflix 데이터를 연동하여 영화를 추천하는 시스템을 설계하기 위해, 아래와 같은 엔티티 및 속성을 정의하고, 엔티티 간의 관계를 ERD로 표현하겠습니다.

### 1. **엔티티 및 속성**

#### 1.1 사용자 테이블 (`User`)

- **Primary Key (PK)**: `user_id`

- Attributes

  :

  - `user_id` (PK)
  - `name`
  - `phone_number`
  - `email`
  - `netflix_user_id` (Foreign Key to Netflix User ID for mapping)

#### 1.2 Netflix 시청 기록 테이블 (`Netflix_Watch_History`)

- **Primary Key (PK)**: `watch_history_id`

- **Foreign Key (FK)**: `user_id`, `netflix_movie_id`

- Attributes

  :

  - `watch_history_id` (PK)
  - `user_id` (FK)
  - `netflix_movie_id` (FK)
  - `watch_start_time`
  - `watch_end_time`
  - `rating`

#### 1.3 Netflix 영화정보 테이블 (`Netflix_Movie`)

- **Primary Key (PK)**: `netflix_movie_id`

- Attributes

  :

  - `netflix_movie_id` (PK)
  - `title`
  - `genre`
  - `release_year`
  - `director`
  - `actors` (List of actors)

#### 1.4 GenieTV 영화정보 테이블 (`GenieTV_Movie`)

- **Primary Key (PK)**: `genietv_movie_id`

- Attributes

  :

  - `genietv_movie_id` (PK)
  - `title`
  - `genre`
  - `release_year`
  - `director`
  - `actors` (List of actors)

#### 1.5 영화정보 매핑 테이블 (`Movie_Mapping`)

- **Primary Key (PK)**: `mapping_id`

- **Foreign Key (FK)**: `netflix_movie_id`, `genietv_movie_id`

- Attributes

  :

  - `mapping_id` (PK)
  - `netflix_movie_id` (FK)
  - `genietv_movie_id` (FK)

#### 1.6 영화 추천 테이블 (`Movie_Recommendation`)

- **Primary Key (PK)**: `recommendation_id`

- **Foreign Key (FK)**: `user_id`, `genietv_movie_id`

- Attributes

  :

  - `recommendation_id` (PK)
  - `user_id` (FK)
  - `genietv_movie_id` (FK)
  - `recommendation_date`

### 2. **엔티티 간의 관계**

- User - Netflix_Watch_History

  : 1

  

  관계 (

  ```
  user_id
  ```

  ), 한 명의 사용자는 여러 Netflix 시청 기록을 가질 수 있습니다.

- **Netflix_Watch_History - Netflix_Movie**: N:1 관계 (`netflix_movie_id`), 여러 시청 기록이 하나의 Netflix 영화와 연관될 수 있습니다.

- User - Movie_Recommendation

  : 1

  

  관계 (

  ```
  user_id
  ```

  ), 한 명의 사용자는 여러 영화 추천을 받을 수 있습니다.

- **Movie_Recommendation - GenieTV_Movie**: N:1 관계 (`genietv_movie_id`), 여러 추천이 하나의 GenieTV 영화와 연관될 수 있습니다.

- Netflix_Movie - Movie_Mapping

  : 1

  

  관계 (

  ```
  netflix_movie_id
  ```

  ), 하나의 Netflix 영화가 여러 GenieTV 영화와 매핑될 수 있습니다.

- GenieTV_Movie - Movie_Mapping

  : 1

  

  관계 (

  ```
  genietv_movie_id
  ```

  ), 하나의 GenieTV 영화가 여러 Netflix 영화와 매핑될 수 있습니다.

### 3. **ERD**

```
plaintext코드 복사User
--------------------
- user_id (PK)
- name
- phone_number
- email
- netflix_user_id (FK)

Netflix_Watch_History
--------------------
- watch_history_id (PK)
- user_id (FK)
- netflix_movie_id (FK)
- watch_start_time
- watch_end_time
- rating

Netflix_Movie
--------------------
- netflix_movie_id (PK)
- title
- genre
- release_year
- director
- actors

GenieTV_Movie
--------------------
- genietv_movie_id (PK)
- title
- genre
- release_year
- director
- actors

Movie_Mapping
--------------------
- mapping_id (PK)
- netflix_movie_id (FK)
- genietv_movie_id (FK)

Movie_Recommendation
--------------------
- recommendation_id (PK)
- user_id (FK)
- genietv_movie_id (FK)
- recommendation_date
```

### 4. **설명 및 제약사항 고려**

- **User** 테이블에서 `netflix_user_id`를 추가하여 Netflix 사용자와의 매핑을 관리합니다.
- **Netflix_Watch_History** 테이블은 사용자의 Netflix 시청 기록을 저장하며, 이는 추천 알고리즘에서 활용됩니다.
- **Netflix_Movie**와 **GenieTV_Movie**는 각각 Netflix와 GenieTV의 영화 정보를 관리합니다. 각 영화는 고유한 ID로 식별됩니다.
- **Movie_Mapping** 테이블은 Netflix 영화와 GenieTV 영화 간의 매핑을 관리합니다. 이 매핑을 통해 동일한 영화를 각 플랫폼에서 식별할 수 있습니다.
- **Movie_Recommendation** 테이블은 GenieTV에서 사용자에게 추천된 영화 정보를 관리합니다. 추천된 영화는 GenieTV 영화 정보와 연결되며, 추천 날짜가 기록됩니다.

이 논리 모델은 GenieTV에서 Netflix 시청 기록을 기반으로 영화 추천 기능을 구현하는 데 필요한 데이터를 구조화하며, 사용자가 더욱 개인화된 영화 추천을 받을 수 있도록 지원합니다.