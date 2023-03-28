* Amazon Aurora
  * Amazon Aurora Serverless(서버리스)
    * 스토리지와 컴퓨팅을 신경쓸 필요없음
    * 콘솔상 db instance class를 선택할 필요없음
      * 인스턴스 선택하게 되면, 해당 인스턴스에 대한 비용 지불 (EC2 서버 비용(ec2에 db 인스턴스 설치되는 구조))
    * ACU(Aurora Capacity Unit) range min ~ max 지정 필요 (메모리 연동)
    * 사용한 만큼만 비용 지용 (on demand) // ACU당 과금
    * 프록시 플릿
      * 기본적으로 프록시가 설치되어, 어플리케이션은 프록시를 통해 하나의 endpoint로 연동 가능(내부적으로 여러 인스턴스를 사용)
    * 인스턴스 증가시 지연(fail)이 발생될 수 있음. 따라 갑작스런 load가 증가시 성능 보장을 위해, 혼재된 cluster 구성 가능
      * Writer 인스턴스(서버리스), Reader 인스턴스(서버리스), Reader 인스턴스(인스턴스) // Reader 인스턴스가 load 증가를 대응함
      * 단일 endpoint는 추가 구성 필요 (cluster 엔드포인트로 or RDS proxy로)
    * 성능은 ACU에 따라 차이 발생

  * Amazon Aurora 복제
    * 논리적 복제 : 인스턴스 DB내 log를 공유하여, 다른 인스턴스에서 log내 쿼리(insert등)를 그대로 수행. 지연 1초 이상
      * 콘솔 Action) Create cross-region read replica
    * 물리적 복제 : DB 네트워크 장치에서 데이터를 공유. 지연 1초 미만
      * 콘솔 Action) Add AWS Region

  * 데이터베이스 복제(clone)
    * RDS
      * RDS read replica를 만들고 promote해서 별도 DB 구성
      * 스냅샷으로 별도 DB 구성 
      * 단점 : 시간 많이소요. 비용발생 // 기존DB 스냅샷 데이터 cp -> S3 cp -> 신규DB로 cp
    * Aurora
      * 복제된 cluster는 데이터베이스를 공유하여 사용 (따라, 스냅샷 처럼 초기 구성필요 없어 빠름)
  * 백업 및 복원
    * 무조건 자동 백업기능 사용해야 함
    * 1day는 무료이고, 2일부터는 비용발생(최대 35일)
    * 복원
      * 스냅샷 : 지연 발생 (스냅샷 cp 등)
      * backtrack 기능 : 원하는 시점으로 바로 데이터 복원 가능
      * from S3
  * 콘솔 메뉴
    * Stop temporarily : 7일동안 stop (비용발생 안하도록) // Aurora 생성하면 삭제하기 전까지 비용 발생됨
    * Set up EC2 connection : 특정 ec2만 연결 (보안)
    * Blue/Green 배포 전략 : 어플리케이션 B/G 배포처럼, DB도 마찬가지 (Green으로 DB튜닝이 잘못되었을때 Blue로 돌아감)
    * Add replica auto scaling : Read Replica를 오토스케일링 (늘리고 줄이고 가능)

  * Aurora 병렬쿼리 실습
    * 콘솔 상 파라미터 그룹의 해당 테이블 DB cluster parameter group상 Filter parameters
      * aurora_parallel_query : ON
      * aurora_disable_hash_join : 0
    * SQL문 실행계획에서 병렬쿼리를 추천하는지 확인 가능
      * Extra열에 Using parallel query가 표시
    * SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool%';
      * InnoDB 버퍼 풀은 버퍼, 캐시, 인덱스 및 행 데이터를 비록하여 InnoDB의 많은 인 메모리 데이터 구조를 보유하는 메모리 공간
      * | Innodb_buffer_pool_read_requests      | 327660144  |
      * | Innodb_buffer_pool_read_requests      | 327662262  | // diff 2118 (vs 병렬쿼리 비활성화시, 30615470)
      * 병렬쿼리가 활성화되면 읽기 요청수가 현저히 줄어듬. 버퍼 풀 읽기 요청을 줄임
    * Aurora는 모든 쿼리문 결과를 캐시에 임시로 저장함. 이는 연속적인 쿼리문이 더 빠르게 실행되게 함
      * 쿼리문에 SQL_NO_CACHE 힌트 추가하여 캐시off (테스트 목적)
        * SELECT SQL_NO_CACHE AVG(DepDelayMinutes + ArrDelayMinutes) AS "Average Delay"
    * 인스턴스 유형에서 허용되는 최대 동시 병렬쿼리 세션 확인
      * SHOW GLOBAL STATUS LIKE 'aurora_pq_max_concurrent_requests';
        * db.r5.4xlarge : 8개 / db.r5.large : 1개
      * 동시에 실행할 수 있는 최대 병렬 쿼리 세션수가 AWS DB 인스턴스 클래스에 따라 달라지는 고정 개수라서, 워크로드에 따라 Aurora 클러스터의 인스턴스 유형을 선택할때 고려해야 함
    * 병렬쿼리로 얻을 수 있는 성능 확보로 인해, 인스턴스 유형을 굳이 높일 필요가 없다(비용절감)
      * 인스턴스 비용 8배 차이나는데 병렬쿼리로 성능 3배 차이로 줄어들면, 굳이 비싼 인스턴스를 사용할 필요없다(서비스 목적에 따라)
    * https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-mysql-parallel-query.html

* DocumentDB
  * MongoDB 호환
  * 문서 데이터베이스
  * NoSQL은 DocumentDB, DynamoDB, QLDB 중에서 서비스에 맞게 선택 필요
  * 구조는 Aurora와 동일
  * 읽기 기본 설정 // 클라이언트가 연결시 요청함
    * Primay or PrimayPreferred : 일관적 (쓰기 후 읽기)
    * Secondary or SecondaryPreferred : 최종적 (현시점 읽기. 쓰기 후 or 전 일수도 있음)
    * nearest : 여러 노드에서 클라이언트에서 가까운 쪽에서 읽기
  * 문서(Document)
    * 데이터는 json
    * 유연한 스키마 및 인덱싱
      * Database안에 Collection(= 테이블) 생성 // 컬렉션에 문서 추가. 문서의 집합
    * 표현식 쿼리언어
    * 일회성 쿼리 및 집계
  * 참고
    * https://s3.amazonaws.com/rds-downloads/rds-combined-ca-bundle.pem
    * https://www.mongodb.com/docs/ : 몽고DB와 api동일함
  * 고가용성 : 3개 가용영역과 3개 노드를 권장함
  * 클러스터 최대 7일동안 중지 가능(중지 안하면 비용발생)
  * 변경 스트림
    * 변경된 데이터에 대해서 이벤트를 저장하고(1~24시간동안) 처리할 수 있름

* DynamoDB
  * 스토리지 뿐만 아니라 컴퓨팅 영역도 관리할 필요없는 완전 관리형 서비스이다
  * cluster 같은 것이 없고, table로 바로 빠르게 생성 가능
  * 3개 가용영역에 데이터가 분산 관리됨
  * 키
    * 단순 기본키(파티션 키) : 기본 index. 중복없는 고유한 값
      * 일정 범위의 영역들이 하나의 파티션에 모인다
    * 복합 기본키(파티션 + 정렬키)
  * 액세스 패턴에 따라 키 설계 필요
  * 읽기 동작
    * Scan : 필터 추가로 모든 아이템 조회 // 느림
    * Query : 기본키로(파티션) 조회 // 빠름
  * 보조 인덱스
    * 로컬 보조 인덱스(LSI)
      * 테이블 생성시만 만들 수 있음 (최대 5개)
      * SK만 추가함 (파티션 키 추가 X)
    * 글로벌 보조 인덱스(GSI)
      * 중간에 추가, 삭제 가능 (최대 20개)
      * 별도의 테이블을 만드는 것과 같음 (LSI 대비해서 비용 추가됨)
      * 파티션 키를 추가할 수 있음 // SK도 추가
        * 검색을 위한 키라 기본키와는 달라서, 중복 가능함
  * 배치 및 트랜잭션
    * 배치 처리
      * BatchWriteItem, BatchGetItem
      * 10개 묶어 처리하여 2개 실패해도 성공으로 API 응답옴. 응답내 실패 요청을 보고 retry 처리해야 함
      * 이점 : 왕복횟수감소, 동시 처리됨(병렬)
    * 트랜잭션
      * TransactionWriteItems, TransactionGetItems
      * 10개 묶어 처리하여 1개라도 실패하면 실패로 처리
      * 격리 : https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/transaction-apis.html#transaction-isolation
  * 데이터 처리시 비용발생함. delete도 비용발생..
    * TTL로 데이터 삭제하면 비용X
  * 액세스 패턴 식별
    * 데이터가 아닌 액세스 패턴으로 부터 시작함
  * DynamoDB 스트림
    * 수정사항 캡처 및 저장
      * PutItem, UpdateItem, DeleteItem
    * 목적 : 데이터 감사, 데이터 복제
    * 24시간 동안 데이터 보유 (이후에 사라짐. 버퍼처럼)
    * 시스템 변경사항을 TTL 기능을 연동하여, 스트림을 통해 이벤트 받아 처리 가능함
  * 글로벌 테이블
    * 여러 리전에 걸친 중복성 및 복원력
  * DAX
    * DynamoDB 캐시
    * DynamoDB는 리전에 위치하기 때문에, EC2 처럼 VPC내 위치하지 않음. ec2에서 DDB 접근시 인터넷 or vpc endpoint로만 연결가능
    * DAX 클러스터는 VPC내 위치하여 ec2에서 바로 접근 가능 (DAX 클러스터는 DDB를 프록시처럼 연동함. 연동후 DDB 데이터를 캐시하여 처리하므로 빠름)
  * 읽기/쓰기 용량
    * Provisioned : 미리 RCU, WCU 설정 (auto scaling 사용하는 것이 이상적) // 초당 RCU, WCU 설정
      * RCU, WCU는 각각 4KB/항목, 1KB/항목
    * on-demand : RRU(읽기요청유닛), WRU(쓰기요청유닛) 요청당 요금 지불
  * PartiQL
    * 안녕하세요. PartiQL Select 문에 관련된 공식 문서입니다. 파티션 키 관련된 내용은 "중요" 부분을 확인하시면 됩니다.
    * https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/ql-reference.select.html
