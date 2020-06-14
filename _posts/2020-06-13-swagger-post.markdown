# REST API 문서 자동화 하기
일반적으로 여러명이서 개발하는 웹 서비스에서 단일 프로젝트가 DB에서 데이터를 조회하고, 화면을 그려주고, 정보를 저정하는 등 모든 것을 담당하지 않는다. 아무리 간단한 구조라도 아래와 같은 형태정도는 유지한다.

![web](https://raw.githubusercontent.com/rbwls31/rbwls31.github.io/master/images/WEB.png)

이런 구조로 처음 개발하거나 유지보수를 진행 중인 경우, 해당 API서버가 어떤 Spec을 가진 데이터를 주고 받는지에 대한 문서작업이 필요하다.
예전에는 엑셀에 메소드 URI, 요청 파라미터, 응답 status code별 설명, 응답 body 등을 컬럼으로 두고 내용을 채웠었다.  하지만 이런 방식은 아래와 같은 문제가 있다.

 - **변경이력 관리**가 어렵다.
 - 엑셀과 같은 모던한 문서화 도구들은 중복을 추상화하기 어렵다. 
 - 가독성이 나쁘다. 비슷한 endpoint끼리 묶어서 카테고리화 시키고, 설명에  테이블이나 리스트와 같은 HTML 기반의 컨텐츠를 추가하는 등의 일이 힘들다.

## Swagger & Spring REST Docs




<!--stackedit_data:
eyJoaXN0b3J5IjpbMjMzNTg3MTA5LDE3NTI3NTc5MjYsLTE3Nj
Y3MjI4NDgsNTA3ODk3NTc3LDY5NzAyNzYyLC00ODI3OTY5MzEs
LTQ3NjMyODYxOF19
-->