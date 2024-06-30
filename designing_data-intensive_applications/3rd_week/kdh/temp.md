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



