* Amazon RDS
  * 완전 관리형
  * 엔진이 오라클, PostgreSQL
* Amazon Aurora // RDS 범주에 포함
  * 완전 관리형
  * 클라우드 네이티브 // 클라우드 환경에 맞게 설계되어 있음
    * 가용영역(AZ) 3개 설정
  * 엔진이 Aurora이고, PostgreSQL/MySQL 호환성을 갖음

* Amazon RDS
  * Multi-AZ // 장애복구
    * AWS 콘솔) 고가용성 및 내구성
      * Multi-AZ DB instance : 다중 AZ (P: 프라이머리, S: 스탠바이)
        * 장애 시 프라이머리 전환
        * 프라이머리 인스턴스만 사용 (스탠바이는 idle) // endpoint가 하나임
        * P -> S 동기식 복제 (consistency 보장)
      * Multi-AZ DB Cluster
        * 클러스터(집합) 개념으로 동작
        * 3개 다른 가용영역 사용 (Writer 인스턴스(r/w): 프라이머리, Reader1 인스턴스(r), Reader2 인스턴스(r): 스탠바이)
        * 장애 시 프라이머리 전환 + App이 개별 Reader 인스턴스에(endpoint가 다름) 직접 접근하여 읽기(select)를 할 수 있음 // 스탠바이 인스턴스도 사용 가능
        * https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/UserGuide/multi-az-db-clusters-concepts.html
        * 현재 지원 가능한 리전 존재
    * 스탠바이 -> 프라이머리로 넘어갈 때, 2~3분 소요되어 fail 감지됨 (사유: 스탠바이 -> 프라이머리 전환시간 및 RDS endpoint DNS 캐쉬 업데이트시간)
  * 읽기 전용 복제본(replica) // 확장성(읽기). Read only 복제본. 독립적 인스턴스
    * 프라이머리(P) : R/W
    * 읽기 전용 복제본(RR) : R // P -> RR 비동기식 복제 (비동기식: 시간차 존재)
    * 프라이머리와 복제본의 endpoint가 다름 (읽기에 대한 load 분산 가능)
    * RR 5개 만들 수 있음 (5개의 endpoint가 다름. App상 5개 주소를 관리해야함..)
      * RDS Proxy로 단일 endpoint 접속토록 도와줌 (LB 역할?)
    * 장애복구 X (P가 죽었다고 해서 RR이 P가 안됨)
    * 굳이 복구를 위해, RR을 promote해서(승격) 독립적인 프라이머리 인스턴스로 만들 수 있음
    * 다른 가용영역, 교차ZA 또는 교차리전 내 생성할 수 있음 (서울리전 -> 도쿄리전으로 RR 만들고, promote해서 운영 가능함)
    * 복제의 주체는 P라서 i/o가 소비됨 (운영시간대 외에 작업)
    * 콘솔상 Multi-AZ로 선택할 수 있음 // 어떤 목적? RR로 만들고 스탠바이로 전환하기 위해
  * Auto 스케일링 : storage 오토 스케일링 (scale up만 가능. down은 매뉴얼 작업)
  * DB connection full 스케일링 : RDS proxy로 가능
  * 최대 64TB까지 사용가능

* Amazon DMS(Database Migration Service)
  * snapshot : 해당 시점 순간의 데이터 마이그레이션
  * DNS : 이기종 엔진 DB로 이전 할때 or 지속적인 마이그레이션 할때

* Amazon Aurora
  * 엔터프라이즈 환경에서 고려해볼 만한 데이터베이스 (컴퓨팅 영역 + 스토리지 영역)
  * 분산 스토리지 볼륨 // 스토리지 영역은 자동으로 관리
    * 콘솔 상 스토리지를 선택하지 못하고 내부적으로 공유 스토리지 볼륨이 숨겨져 있음 (vs RDS는 스토리지 EBS 볼륨을 선택함)
    * 공유 스토리지 볼륨은 일종의 서버라고 보면됨(cpu, os 있음)
    * 3개 가용영역에 총 6개의 셀에 자동적으로 복제됨(1셀 : 10G) // 최대 128TB 까지 가용 스토리지
      * 하나의 insert 쿼리 시, 프라이머리 인스턴스에서 트랜잭션 수행후 공유 스토리지 볼륨으로 보내면 끝 (공유 스토리지 내부적으로 복제 동작)
      * vs RDS는 프라이머리 인스턴스가 모든 RR에 동기적 복제를 수행함. 따라 프라이머리를 기본 i/o성능에 추가적으로 오버 프로비져닝 해야함
        * RDS 사용시 multi-az 할거야? read only 복제본 할거야? 라는 질문 필요 (프라이머리 인스턴스 성능 산정을 위해)
  * Primary instance 1개 + Read only replica 2개(replica 최대 15개)
    * Primary 인스턴스(Writer) : 쓰기 전용(읽기도 가능), Replica(Reader) : 읽기 전용
    * 장애 복구 기능 (primary 장애시, replica가 primary로 승격) // 스탠바이 없음
    * Read only 복제본 기능
    * replica 우선순위를 주어, failover시 우선순위에 따라 선택됨
    * instance와 replica는 cpu, 용량을 다르게 선택할 수 있음
    * replica를 auto scaling 선택 가능 (자동으로 스케일 업/다운 가능)
  * 캐싱 (자주 반복되는 데이터) : 데이터베이스와 캐싱이 분리되어(RDS는 한몸), 재시작되어도 캐싱 유지됨
  * 스토리지 및 컴퓨팅 결합 해제 (RDS는 스토리지와 컴퓨팅이 결합. 인스턴스 당 한벌의 EBS 갖음)
  * 분산 스토리지 계층
  * Aurora MySQL 병렬 쿼리 클러스터
    * 누가 쿼리하냐? 스토리지 노드단
    * 추가 비용없음. 복잡한 쿼리(수백만개 행. Sum, join 등)를 쿼리 프로세서가 병렬 조건에 맞는지 판단하고, 스토리지 노드에 병렬로 처리 요청함
  * Aurora database clusters (싱글 마스터)
    * 클러스터 개념으로 동작함
    * Primary, Readonly 복제본 각각의 다른 endpoint를 갖음
    * Cluster endpoint : W용(primary로 연결), R용(Readonly 복제본들로 라우팅함. LB는 아니고 도메인 라운드로빈 방식으로 보면됨), Custom(인스턴스 지정가능)
  * Aurora database 멀티 마스터 clusters
    * 모든 인스턴스가 primary (쓰기, 읽기 처리). Readonly 복제본이 없음. 최대 4개
    * 액티브-패시브 : 장애시 다른 마스터로 바로 이동
      * 프라이머리 전환의 지연시간 없음(이미 프라이머리라서. 가동중지시간 최소화). RDS는 스탠바이->프라이머리 전환시간에 따른 지연있음
    * 액티브-액티브 : 워크로드를 분산 목적
    * 여러 인스턴스에서 동일 레코드에 값 변경 시(update) 커밋 전에 변경점을 미리 감지하는 기능을 지원함
    
  * Q) 오로라 최대 128TB 스토리지 제공하는데 복제본이(6개) 포함된 힐딩량인지요? (복제본 포함된 거라면 중복없는 데이터는 128/6 = 21.3TB로만 사용할 수 있는게 아닐지)
