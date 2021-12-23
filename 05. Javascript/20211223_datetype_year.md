# 연도 datetype 을 yyyy 에서 uuuu 로 써야 하는 이유
elasticsearch 로그를 teams 로 받아보도록 data pipeline 을 구축하자마자 온갖 elasticsearch warning 이상의 로그가 날아들게 되었는데...  
그 와중에 전혀 인지하고 있지도 못했던 로그 메시지가 등장했다.
```
'y' year should be replaced with 'u'. Use 'y' for year-of-era. Prefix your date format with '8' to use the new specifier.
```
여지껏 logstash pipeline 이든 elasticsearch query dsl 이든 잘만 yyyy 로 쓰고 있었는데 갑자기 웬 uuuu 죠(...)

## JAVA8 이상부터 DateTimeFormatter 에서는 yyyy -> uuuu 로 작성
...하라고 한다! 그 이유인즉,  
* **yyyy** : year-of-era. BC/AD 에 대한 구분을 하기 어렵다
* **uuuu** : year. BC/AD 구분을 -/+ 로 한다

yyyy 의 경우 signed 로 표시되기 때문에 기원전 연도를 양수로 표기한다.  
예) yyyy 패턴으로 출력한 결과가 3 일 때, 기원후 3년인지 기원전 2년인지 별도 표기할 방법이 없다.  
그렇기 때문에 yyyy 보다 uuuu 를 사용해서 기원전 연도에 대한 표기를 확실히 해 둘 필요가 있다고 한다.  
~~(...는 로그 다루는 데 기원전 연도까지 신경써야 하는 것인가...)~~

## 그럼에도 yyyy 를 쓰고 싶다면?
1. 'G yyyy*' 로 표기
2. '8yyyy*' 로 표기 

## 참고
* https://blog.voidmainvoid.net/288
* https://bufferings.hatenablog.com/entry/2017/10/23/000057
