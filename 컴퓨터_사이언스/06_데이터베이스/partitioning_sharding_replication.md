# partitioning, sharding, replication
- partitioning
    - table을 목적에 따라 작은 table로 나누는 방식
- sharding
    - horizontal partitioning으로 나누어진 table들을 각각의 DB 서버에 저장하는 방식
- replication
    - DB를 복제해서 여러 대의 DB 서버에 저장하는 방식

## partitioning
- database table을 더 작은 table들로 나누는 것
- 종류
    - vertical partitioning
        - column을 기준으로 table을 나눔
    - horizontal partitioning
        - row를 기준으로 table을 나눔

## vertical partitioning
- column을 기준으로 table을 나눔
    - 성능, 보안 등의 다양한 이유로 인해 column을 기준으로 table을 나눔
- 정규화
    - 정규화는 vertial partitioning의 일종임!!
    - 정규화를 하게 되면...
        - 중복된 데이터가 저장되는 것을 방지
            - 동일 데이터를 여러 곳에 저장하지 않도록 테이블을 나눔
            - 중복을 줄이면 저장 공간 절약 & 데이터 일관성 유지 가능
        - 데이터 무결성(integrity) 유지
            - update anomaly 방지 - 중복된 데이터가 여러 곳에 있으면 하나만 수정해도 됨
            - delete anomaly 방지 - 불필요한 데이터 삭제 시 관련 데이터가 날아가는 문제 방지
            - insert anomaly 방지 - 필요한 데이터가 없어서 삽입할 수 없는 문제 방지
    - 이미 정규화된 테이블이라 하더라도 성능을 위해 vertical partitioning을 수행할 수 있음
- 성능 개선
    - 게시판에서 article 테이블에서 조회하는 경우
    - content column의 사이즈는 크지만, 게시글 리스트를 뿌릴 땐 content는 필요가 없음
    - selec할 때 I/O에 부담을 줘 performance에 영향을 줌
    - 이런 경우 vertical partitioning으로 해결
    - article 테이블과 article_content 테이블로 나눈다.
    - 이것도 vertical partitioning!!!
- 보안
    - 민감한 정보에 함부로 접근하지 못하도록 
    - vertical partitioning을 통해 테이블을 분리하여 따로 관리할 수도 있음

## horizontal partitioning
- row를 기준으로 table을 나누는 방식
- 스키마는 그대로 유지
- 문제 상황
    - 테이블의 크기가 커지면
    - 인덱스의 크기도 커진다
    - 읽기 쓰기가 있을 때 마다 인덱스에서 처리되는 시간도 조금씩 늘어난다.
- 해결: horizontal partitioning로 해결
- `hash-based`
    - 해시를 이용한 horizontal partitioning
    - 예시
        - hash의 결과가 0 또는 1일 때
        - useser_id를 hash에 넣은 결과에 따라 0번 테이블 또는 1번 테이블에 저장
    - 테이블을 나누고, 해시함수로 테이블을 결정하여 나눠 저장
    - 이 때 기준이 되는 컬럼을 `partition key` 라고 함
- `hash-based` 주의할 점
    - __가장 많이 사용될 패턴에 따라 partition key를 정하는 것이 중요__
        - partition key를 기준으로 테이블을 결정하여 조회하는데,
        - 이게 아닌 column으로 조회하면, 두 테이블을 모두 조회하여야 함.
        - 테이블을 나눈 이점을 온전히 누릴 수 없게 됨
        - partition key로 조회하면, 해당 테이블에서만 조회하면 되니까 성능 이점 생김
    - __데이터가 균등하게 분배될 수 있도록 hash function을 잘 정의해야 함__
    - __한번 partition이 나눠져 사용되면 이후에 partition을 추가하기 까다롭다__
- 그 외
    - range-based 방식도 있음!

## sharding 샤딩
- horizontal partitioning처럼 동작
- __각 partition이 독립된 서로 다른 DB 서버에 저장__
- horizontal partitioning으로 나눈 각 테이블들을 다른 서버에 저장한 것
- __DB 서버의 부하를 분산시킴!!!__
- 이 때의 partition key를 `shard key` 라고 함
- 이 때의 각 partiton을 `shard`라고 함

## replication 복제
- 주 DB 서버, 이를 복사해서 유지하는 보조 DB 서버를 둠
- 끊임 없이 copy해서 싱크를 맞춤
- master, primary, leader
    - 실제 read/write가 일어나는 서버
- slave, secondary, replica
    - copy해서 유지하는 서버
    - 여러 대 있을 수 있음
- 장점
    - __고가용성 제공__
        - HA - High availability - 가용성이 높음!
        - master에 문제 생겼을 때 slave로 바로 바꿔치기
            - 이를 `Failover` 라고 함
            - 시스템에 이상이 생겼을 때 예비 시스템으로 자동전환되는 기능
    - __서버 부하(load)를 낮춘다__
        - read 트래픽을 slave로 분산
        - master의 부하를 낮춤