### snapshot isolation
이전에 real mysql을 읽을 때, 만약 트랜잭션이 길어지게 되면 언두로그가 쌓여서 문제가 발생할 수 있다는 내용을 슥 읽고만 지나갔었는데, 이 책을 읽고 나니 어떤 이유에서 트랜잭션이 길어지면 언두로그가 쌓이는지 알 수 있게 됨

### locking 시점
여러 쿼리가 있는 트랜잭션에서 lock을 걸때 시작하기전에 먼저 접근할 데이터들을 탐색해서 lock을 걸고 시작하는건지 트랜잭션 수행중에 쿼리에서 특정 데이터에 접근하면 그때 lock을 거는 건지 궁금했는데, 
책에 아래와 같은 말이 있음
> If a transaction first reads and then writes an object, it may upgrade its shared lock to an exclusive lock. The upgrade works the same as getting an exclusive lock directly.

집적적으로 말하진 않지만 write를 시작하면 그때 lock을 건다는 걸 알 수 있음

