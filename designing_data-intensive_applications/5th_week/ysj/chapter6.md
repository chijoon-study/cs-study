### 6장 파티셔닝

#### 정리

- 파티셔닝(샤딩)
    - 데이터를 파티션으로 쪼개는 것.
    - 나뉘어진 데이터를 파티션, 샤드, 리전 등으로 부른다.

- 파티셔닝 목적
    - 확장성: 한 데이터베이스로 처리하기에는 너무 많은 데이터를 다루는 경우, 데이터베이스를 분리하여 더 높은 확장성을 얻을 수 있다.
        - 여러 요청이 분산되어 처리되므로 더 높은 효율성
        - (상대적으로 적은 데이터, 병렬 처리 기법)

- 파티셔닝과 복제
    - 파티셔닝과 복제는 일반적으로 함께 조합하여 사용한다.
    - 일반적으로 한 노드가 여러 파티션을 가진다.
        - 각 노드는 하나의 리더 파티션과 여러 팔로워 파티션을 가진다.
        - 즉, 모든 노드는 일부 파티션의 리더이면서, 나머지 파티션의 팔로워이다.

- 파티셔닝 주의점
    - 분산을 잘 처리하여 핫스팟이 발생하지 않아야 한다.
        - 핫스팟: 특정 파티션(노드)에만 요청이 쏠리는(과부하) 현상 (DB만의 개념은 아니다.)
        - skew(쏠림, 편향)이라고도 함.
    - 파티션을 가지는 노드의 변경(축소, 확장 등)를 (사용 불가) 문제 없이 처리할 수 있어야 한다.

- 파티셔닝 기법
    - key-value:
        - 특정 키의 범위를 기준으로 파티셔닝.
        - 키의 범위는 서비스에 따라 파티션의 경계를 데이터에 맞춰 조정함.
        - 정렬된 상태를 유지하므로 범위 조건 검색에 유리.
        - 단, 서비스 특징에 따라 핫스팟이 발생하기 더 쉬운 편.
            - 특정 키의 범위를 가지는 검색이 자주 발생하는 등.
    - hashing:
        - key를 hashing한 값을 기준으로 파티셔닝.
        - 정렬된 상태를 가지지 못하므로 범위조건 검색에 불리.
        - hashing을 통해서 데이터가 고르게 분산되므로 핫스팟이 더 적게 발생하는 편.
    - 혼합:
        - 여러 방식을 섞어서 쓰는 것. (첫 번째 조건은 hashing, 나머지는 범위)
            - `hash(user_id), 정렬조건1, 정렬조건2` << 이런 식
        - e.g. 카산드라 복합 기본키

- 핫스팟 완화
    - 아직 현대 어플리케이션은 skew를 해결해줄 수 없음.
    - 따라서 어플리케이션 단에서 추가적으로 처리해줘야 함.
    - e.g. 요청이 많은 유명인의 경우. 어느 기법을 사용하던 쏠림 현상 발생.
        - 특정 부하를 주는 사용자의 ID를 여러 ID로 분리하고 나눠서 관리. 대신 조회 시 여러 ID를 읽어야 함.

- 파티셔닝과 보조 인덱스
    - 데이터 검색 결과를 높이기 위해서 많은 데이터베이스에서 보조 인덱스 기능을 제공.
    - 주로 보조 인덱스는 각 파티션에 저장되고, 인덱스 조건과 원본 데이터를 찾을 수 있는 ID(key)를 가진다.
    - 종류:
        - 지역:
            - 쓰기 작업 편리: 한 파티션에만 쓰면 됨.
            - 읽기 작업 불리: 여러 파티션을 전부 읽어야 함.
                - 스캐터/개더 방식이라고 함.
        - 전역:
            - 쓰기 작업 불리: 여러 파티션을 전부 써 줘야함.
                - 그래서 주로 비동기로 처리됨.
            - 읽기 작업 편리: 한 번만 읽으면 됨.

- 파티션 재균형화
    - 목표: 노드의 변경 발생 시에도 skew 없이 균등하게 데이터 요청을 처리할 수 있어야 한다.
    - 주의점: 데이터를 필요 이상으로 이동시키지 않아야 한다. (mod 연산을 쓸 수 없는 이유)
    - 전략:
        - 파티션 개수 고정: - (장단점까지는 잘 모르겠음.)
            - 처음 분할 시 파티션 개수가 고정되고, 노드 변경 시 일부분을 나눠 사용하는 방식
            - 운영 간단함.
            - 파티션 개수를 변경할 수 없으므로 잘 설정해야 함.
        - 동적 파티셔닝:
            - 파티셔닝을 DBMS가 동적으로 생성함.
        - 노드 비례 파티셔닝:
            - 노드 별로 가지는 파티션 개수를 고정.

- 자동/수정 재균형화
    - 자동은 편리하지만 예측하기 어려우므로 잘 사용하지 않는다.
    - 수동을 선호하며, 자동을 사용하더라도 선택은 관리자가 하도록 하는 편이다.

- 요청 라우팅 기법
    - 필요성: 노드가 자주 바뀌므로 클라이언트는 어디에 요청해야 할지 잘 모른다.
        - 구성 정보를 저장하고 변경을 관리하는 어떤 역할이 필요하다.
            - 분산 환경에서 자주 다루어지는 service discovery 문제.
    - 방법: 
        - (잘 안쓰는거) 각 노드가 구성정보 관리, 라우터 계층에서 관리, 클라이언트가 관리
        - 주로 별도의 코디네이션 서비스를 사용한다.
            - 이 경우 클라이언트가 코디네이션 서비스를 알아야 하는데, 코디네이션 서비스는 잘 안 바뀌므로 DNS를 사용한다.

#### 후기

##### 6장이 분량이 적은 이유?

결국에 대규모 시스템(분산 환경)이 어려운게 일관성 때문이다.  
그런데 파티셔닝은 일관성을 지키는게 핵심이 아니다. 각 서비스가 별개의 서비스처럼 독립적인 상태(일관성)를 가지고 있으니까...   
물론 보조적으로 필요한 건 맞는데, 복제에 이미 다룬 내용이고, 파티셔닝의 핵심이 아니니까 짦게 다루고 넘어간다.   
그래서 분량이 적은게 아닐까?

