* Amazon Neptune
  * 특별히 구축된 고성능 그래프 데이터베이스
  * 인프라는 Aurora와 거의 동일
  * 그래프 솔루션 // 관계(relationship)
  * 기존 관계형DB를 사용할 수 있지만, 넵튠을 통해 데이터 관계를 쉽게 구축하고(모델링) 찾을 수 있다.
    * ex. 소셜 네트워킹, 추천 엔진(facebook)
  * 지원 프레임워크
    * Gremlin 순회 언어
    * SPARQL 쿼리 언어
    * opencypher : https://opencypher.org/
  * 참고
    * 주피터 노트북 : https://jupyter.org/ (실행 과정, 결과 및 주석 등을 노트북 형태로 기록할 수 있어 유용함)

* Amazon QLDB(Quantum Ledger Database)
  * Ledger : 원장 // 블록체인
    * 원장: 모든 계정의 거래 내역을 기록하는 장부
    * ledger를 데이터베이스라고 보면 됨. tables을 생성할 수 있음
  * 이점 : 변경 불가능 및 투명성(어떻게 변경되었는지 앎), 완전 서버리스(DDB와 비슷)
  * 문서형 데이터베이스
  * PartiQL 지원
  * 핵심 개념
    * 저널 (내부 데이터는 블록화 되고 체인화됨(블록체인), 데이터는 암호화) // 데이터가 저장되는 공간
    * history 함수를 통해, 데이터 변경 히스토리를(insert, update, delete 등) 모두 볼수 있음
      * SELECT * FROM history(telephone) // telephone 테이블 히스토리 조회
      * delete를 해도 데이터가 안지워지고 남아 있다
  * 스트림 기능 지원
    * 최근 데이터의 변경을 처리할 수 있음 // 스트림 지원 database : Aurora, DynamoDB

* Amazon ElasticCache
  * Memcached, Redis 오픈소스 호환성 지원 (RDS 느낌)
  * 이점 : 고성능(메모리상 관리. 디스크 사용하지 않음)
  * 일반적인 캐싱 전략
    * 레이지 로딩
    * 동시 기록 : ElasticCache와 데이터베이스에 동시에 기록함 (캐시에 항상 데이터 존재)
  * Memcached : 캐시용도로 사용
    * 클러스터 구성 (노드 20개 up) : 독립형 노드 (볃도의 노드 엔드포인트 존재. 클러스터임에도 앱상 각 엔드포인트를 관리해야함)
      * 독립형 노드라 데이터가 다른 노드로 복제되지 않음 (A노드 데이터 저장시 B노드로 복제되지 않아, A노드에서만 조회가능)
    * 데이터를 키에 따라 여러 노드로 파티셔닝하여 데이터를 분산해야 함 (프로그램 상 해야됨)
    * 구성 엔드포인트(Configuration Endpoint) 지원
      * 현시점에 어떤 노드가 존재하는지 정보 제공받을 수 있음
    * 프로그램 상 노드 추가/삭제를 관리해야 함
    * 목적 : 데이터베이스 캐시, 웹 세션 캐시
  * Redis : 인메모리 데이터베이스 용도로 사용 (캐시로도 사용)
    * 샤드 (샤드란 하나의 데이터베이스를 작은 데이터베이스(샤드)로 분할하는 분산 방법. 데이터를 분산 관리)
      * 각 샤드는 1~6개의 Redis 노드 모음 (읽기/쓰기용 기본(Primary)1개 노드, 최대 5개의 읽기전용(RR) 복제본 노드)
      * Primary가 죽으면 데이터가 손실이 됨(RR이 Primary로 failover되긴 함) // 데이터베이스로 사용하기 어려움. 대안) Amazon MemoryDB for Redis
    * Redis 클러스터 구성 모드
      * 단일노드 // 단순 캐시용도
      * 다중노드(클러스터모드 비활성화): 단일샤드 // 데이터 안정성 (데이터베이스 캐시로 사용시)
        * memcached보다 redis가 데이터베이스 캐시로 쓰는게 더나음. 캐시 죽으면 DB로 load가 몰려 DB가 죽을 수 있기 때문. redis는 failover 되서 DB가 죽지 않음
      * 다중노드(클러스터모드 활성화): 다중샤드 // 데이터 안정성, 용량 확장

* Amazon Redshift
  * 관계형 데이터베이스. 분석 솔루션
  * 클러스터 구성요소 (VPC내 위치)
    * 리더노드(leader)
    * 컴퓨팅 노드 : 노드당 cpu, 메모리, 디스크(최대 128TB)를 결정함
      * 슬라이스 : 논리적인 데이터를 나누는 개념
    * 클라이언트는 단일 엔드포인로 접속
  * PostgreSQL를 사용
  * 지능형 데이터 웨어하우징
  * 주요 특징
    * Spectrum
      * redshift 뿐만 아니라 s3 데이터 연결하여 데이터를 분석함 (redshift로 s3 데이터를 가져오지 않음)
    * 워크로드 관리(WLM)
      * 단기 쿼리 가속화(SQA) : 대부분 장기분석용 쿼리인데, 단기쿼리시 가속화 제공
      * 동시성 규모 조정(스케일 아웃/인)
  * 데이터 모델
    * 열 기반으로 저장(행 기반 아님) : 읽기 성능이 빠름 (행기반은 모든 행을 스캔하고 특정 열을 가져옴. 열기반은 해당 열만 가져오면 됨)
    * 데이터 분산
