데이터를 표현하는데에는 크게 봤을때 두가지 방법이 있다.  
- in-memory
- file(disk)

in-memory의 경우 포인터를 통해 해당 프로그램에서는 접근하는데에 최적화가 돼있지만  
만약 다른 컴퓨터로 이 정보가 그대로 넘어가면 아무 쓸모가 없어진다.  
때문에 다른 컴퓨터에서도 해당 객체를 이해하기 위해 encoding(a.k.a serialization, marshalling)작업이 필요하다.  

프로그래밍 언어에서는 대부분 그런 인코딩 기능을 지원하는 표준 라이브러리를 탑재한 채로 출시된다.  
하지만 adhoc 작업이 아닌 영구적으로 쓰일 기능에서 이런 언어의 인코딩 기능을 활용하면 해당 기능은 해당 언어에만 종속되버린다.  
때문에 language independent한 format들이 존재한다.(e.g. json, csv, xml)
이런 language independent foramt도 만능은 아니고 여러 단점이 있지만 비교적 훨씬 범용적으로 쓰일 수 있기에 그럼에도 인기가 많다.  

데이터를 binary형태로 표현하는거는 효율성 측면에서 중요한 부분이다.  
때문에 protobuf, thrift등 json을 이진 형태로 표현하는 다향한 기법이 있다.  

protobuf는 스키마 정보가 따로 있는데  
이 스키마 정보에는 field_name 과 tag가 매핑돼있고  
실제 이진형태 데이터에서는 tag값을 사용한다.  
이 field_tag는 추가 될 수 있지만 수정 될 수는 없다.  
이진 형태 데이터에서 태그값이 1로 돼있는데 스키마 정보에서 1이었던 태그값을 2로 바꾸면 기존 데이터에서 태그값1 이란 정보는 의미가 없어지기 때문이다.  

덕분에 forward compatibility가 지켜질 수 있다.  
old code에서 변경된 데이터를 읽으려하면 태그값이 추가된건 있어도 수정 된건 없을테고  
old code에서는 변경되지 않았을 기존의 태그값을 바탕으로 데이터를 읽으면되고 추가된 필드는 단순히 무시되기 때문이다.  

필드 삭제의 경우에는 optional field만 삭제가 가능하다


avro는 thrift가 Hadoop에 작 적용되지 못해 시작된 프로젝트이다.  
protobuf보다 더 compact하게 데이터를 인코딩한다.  
avro도 똑같이 스키마 정보를 따로 가지고 있다.  
avro로 인코딩된 데이터는 단순히 데이터의 길이와 값만 가지고 있다. 데이터 타입도 없다.  
필드 정보는 가지고 있지 않다.  
대신 데이터를 read할 때 writer의 스키마도 가지고 있어야한다.  
데이터를 read할 때 writer의 스키마와 reader의 스키마를 비교해 필드의 순서, 추가, 삭제 정보를 이해하고 데이터를 읽는다.  

근데 어떻게 reader가 writer의 스키마를 알 수 있을까?  
스키마가 진화함에 따라 한 저장소에 여러 버전의 스키마로 쓰인 데이터가 공존할 수 있고 모든 record에 스키마 정보를 포함시키는건 딱봐도 매ㅐㅐㅐ우 비효율적이다.  
여기엔 몇가지 상황이 있다.  
- 많은 레코드가 쓰인 매우큰 파일 하나를 저장하는 경우  

이 경우 간단하다.  
해당 파일에 레코드들의 스키마를 저장하면 된다.  

- 각각 데이터가 따로 쓰인 경우  

일반적인 RDB를 생각했을때 레코드들은 개별적으로 쓰인다.  
이 경우 가장 간단한 해결책은 스키마 버전 컬럼을 추가하는 것이다.  
그리고 스키마 버전 리스트를 기억해놔  
데이터를 읽을때 스키마 버전 컬럼값으로 스키마를 가져오는 방식이다.  

- 네트워크를 통해 데이터를 보낼때 

이 경우 두번째 방법으로 하기에는 상대측에서 내가 가지고 있는 스키마 정보가 없을 수 있다.  
때문에 연결을 setup할때 어떤 스키마를 사용하자고 이야기한 이후에 연결을 진행한다.

> 근데 그러면 데이터를 읽을때 다른 format에 비해 스키마를 비교하고 하는 과정에서 읽을때 오버헤드가 좀더 생기는거 아닌가?  
> 라고 생각했는데 한번만 하니까 상관없나


REST에 비해 SOAP가 deprecated 된 이유는  
너무 복잡해서 SOAP사용자는 도구의 지원을 받아야했다.  
하지만 SOAP 서포트 툴을 사용하지 않는 언어의 사용자는 SOAP를 사용하기 매우 어렵다.  
이러한 이유로 SOAP는 대기업에서는 여전히 쓰이지만, 최근 스타트업에서는 보통 쓰이지 않는다.  


마지막에 actor model이란게 나왔는데 이건 뭔지 잘 모르겠따..
