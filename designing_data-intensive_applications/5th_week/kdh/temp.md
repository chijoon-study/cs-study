## 사담
shared-nothing  
sharded nothing이 아니라 "share"d nothing임 헷갈리지 말 것(내가 헷갈려했음)

## partitioning시 고려할 점 
데이터를 Partitioning 할 때  
데이터를 좀 고르게 분산해야 효율적일 것임  
극단적인 예로 partitioning 해도 한 노드에만 모든 데이터가 몰린다면 partitioning 한 의미가 없음.  


## random방식
가장 간단한 방법으로는 랜덤하게 데이터를 분산해주는 것.  
가장 간단하고 공평하게 데이터를 나눠주지만  
큰 문제점 하나가 있음  
데이터를 요청할때 어느 데이터가 어느 노드에 있는 지를 알 수 없음.  


## binning
다른 방법은 키 기준으로 분산하는 거임.    
예로 14개 노드가 있다면 각 데이터의 첫 글자를 기준으로 ㄱ,ㄴ,ㄷ,ㄹ.. 별로 나눠주는 것.  
하지만 이는 완전히 공평하게 나눠질 수는 없음  
각 글자별로 편차가 존재하기 때문  
간단하게 이 글에서 현재 지점까지 'ㅌ'으로 시작하는 단어는 하나도 없음.  


## binning시 고려할 점
그 분산하는 키에 대해서도 생각해봐야 할 것이  
위의 예제 처럼 단순히 담당자가 직접 분산할 키 기준을 정할 수도 있고  
단순히 timestamp로 분산할 수도 있음  
timestamp로 할 경우 만약 day기준으로 분산한다 할때 하루동안은 write부하가 분산되지 않는 문제가 발생함.  


## hash
또 다른 방법으론 해쉬함수를 이용하는 거임.  
해쉬함수 특성상 데이터는 당연히 고르게 분산될 것이고, 암호화도 필요치 않음(예시로 카산드라와 몽고DB에선 MD5 알고리즘을 사용함)  
각 언어별로 해쉬함수도 보통 빌트인함수로 구현돼있기에 구현도 어렵지 않음.  
하지만 언어의 해쉬함수를 사용하는 건 한가지 문제가 있는 것이 자바의 Object.hashcode()와 Ruby 객체의 hash는 같은 키여도 프로세스마다 값이 다를 수 있음  
 

## hash의 단점
하지만 해쉬함수에서는 키 기준으로 분산하는 방식의 큰 장점 하나를 놓치게 됨.  
바로 효율적인 range query임.  
인접한 데이터들도 파티션별로 나눠지면 전부 흩어지기 때문에 
데이터를 정렬한 의미가 사라짐.  
예로 몽고DB에서 만약 hash-based sharding mode를 키면 range query를 날렸을때 모든 파티션에 같은 쿼리를 날림  


## 해쉬 + range
cassandra에선 그래서 두 방법의 타협점을 찾은 것이,  
복합키를 이루는 컬럼등 중 한 컬럼으로는 해쉬를해 파티셔닝하고 다른 컬럼으로 index역할을 하는 것임.  
예를 들어 post를 업데이트한 로그의 키를 (user_id, timestamp)로 한다면  
한 유저의 특정 시간대의 업데이트 로그를 가져온다면 효율적으로 가져올 수 있음.  
물론 해쉬값으로 쓰인 다른 user_id의 로그는 다른 파티션에 있고 따로따로 쿼리해줘야함

## skew when using hash
해쉬값을 쓴다하더라도 드물겠지만 부하의 비대칭이 발생할 수 있다.  
예를 들어 sns에서 인플루언서가 어떤 액션을 취한다면 수많은 유저들의 액션을 불러일으킬거고 그런 액션들의 키가 인플루언서의 userID라 하면, 같은 키에 해슁을 하는 것이므로 같은 값이 나오고 해당 키를 가지고있는 파티션은 높은 부하를 감당해야한다.  
때문에 책에서는 만약 특정키의 부하가 높은게 발견된다면 해당 키에 랜덤한 수를 더하는 것을 간단한 방법으로 제안한다.  
확실히 단순히 0~100개의 사이의 값만 더해도 100개의 다른 파티션으로 분산시킬 수 있을테니, 하지만 읽을때는 그만큼 또 오버헤드가 발생한다.  
위에서 말한대로 0~100을 더하는 식으로 했다면 해당 키의 값을 읽으려할때 키에 0~100을 더한값으로 전부 쿼리를 해주고 취합해줘야할테니 말이다.  


## Rebalancing
리밸런싱을 하는데에는 여러 이유가 있다.  
데이터의 양이 많아져서 설정해놓은 파티션당 최대 데이터량을 초과했다거나, 특정 파티션으로 부하가 쏠려서 노드가 overload되거나 등등  
이런 리밸런싱은 어드민이 직접 파티션을 쪼개고 옮기는 수동방법과 알고리즘에 따라 자동으로 분할돼 분산되는 자동방법이 있다.  
당연히 자동화하는 쪽이 더 좋아보이겠지만, 리밸런싱을 완전히 자동화 시킬 경우 예상치 못한 문제가 생길 수 있다.  
리밸런싱 작업은 
- 요청 rerouting
- 대용량 데이터 전송

의 작업을 동반하기에 네트워크와 다른 노드에 높은 부하를 줄 수 있기 때문이다.  
때문에 수동과 작업의 비율을 적절히 맞추는 것이 좋다.  
예로 Couchbase, Riak, Voldemort은 리밸런싱된 파티션을 제안하지만 해당 작업을 허가하는 것은 어드민이 직접 해줘야한다.  




