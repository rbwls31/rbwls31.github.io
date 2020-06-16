
# REST API 문서 자동화 하기
일반적으로 여러명이서 개발하는 웹 서비스에서 단일 프로젝트가 DB에서 데이터를 조회하고, 화면을 그려주고, 정보를 저장하는 등의 모든 것을 담당하지 않는다. 아무리 간단한 구조라도 아래와 같은 형태정도는 유지한다.

![web](https://raw.githubusercontent.com/rbwls31/rbwls31.github.io/master/images/WEB.png)

이런 구조로 처음 개발하거나 유지보수를 진행 중인 경우, 해당 API서버가 어떤 Spec을 가진 데이터를 주고 받는지에 대한 문서작업이 필요하다.
예전에는 엑셀에 메소드 URI, 요청 파라미터, 응답 status code별 설명, 응답 body 등을 컬럼으로 두고 내용을 채웠었다. 하지만 이런 방식은 굉장히 시간이 많이걸리고, 매번 기능이 추가되거나 변경될 때마다 작업을 따로 해줘야 했기 때문에 매우 비효율 적이었다. 그러다보니 API Spec 문서를 자동화는 도구들이 나오게 되었다. 
- Swagger
- Spring REST Docs

## Swagger
Swagger는 간단한 설정으로 프로젝트에서 지정한 URL들을 HTML 화면으로 확인할 수 있게 해주는 프로젝트로, Java뿐만 아니라 NodeJs, Python 등 다양한 언어를 지원해준다. 

> 아래의 내용은 솔루션 연구팀에서 개발하고 있는 행사 플랫폼(Lerni)에 실제 적용한 코드를 기반으로 작성하였다.
---
### 1. 의존성 추가
maven pom.xml
	- Spring은 Swagger2 명세서의 구현체인 Springfox를 제공한다.
```
<dependency>
<groupId>io.springfox</groupId>
<artifactId>springfox-swagger2</artifactId>
<version>2.9.2</version>
</dependency>

<dependency>
<groupId>io.springfox</groupId>
<artifactId>springfox-swagger-ui</artifactId>
<version>2.9.2</version>
</dependency>
```
---	
 ### 2. Swagger 설정
 Swagger 설정을 위해 SwaggerConfig 클래스를 생성한다.
 
 ```java
@Configuration  
@EnableSwagger2  
public class SwaggerConfig {  
  
  private static final String VERSION = "2.0";  
  
  @Bean  
  public Docket api() {  
  return new Docket(DocumentationType.SWAGGER_2)  
  .select()  
  .apis(RequestHandlerSelectors.any())  
  .paths(Predicates.not(PathSelectors.regex("/error.*")))  
  .paths(Predicates.not(PathSelectors.regex("/actuator.*")))  
  .build()  
  .useDefaultResponseMessages(false)  
  .apiInfo(apiInfo())  
  .tags(API_USER_ACCOUNT, API_USER_BY_USER)  
  .tags(API_ROLE_USER, API_ANY_AUTH, API_NO_AUTH)  
  .tags(API_EVENT_BY_USER, API_EVENT_BY_MANAGER)  
  .produces(Sets.newHashSet(MediaType.APPLICATION_JSON_VALUE))
  .consumes(Sets.newHashSet(MediaType.APPLICATION_JSON_VALUE));
  }  
  
  private ApiInfo apiInfo() {  
  String applicationName = "LERNI";  
  String applicationUrl = "https://app.lerni.kr/rest";  
  String description = "API Document";  
  Contact contact = new Contact(applicationName, applicationUrl, "admin@lerni.net");  
  String license = "";  
  String licenseUrl = applicationUrl;  
  
 return new ApiInfoBuilder()  
  .title(applicationName)  
  .termsOfServiceUrl(applicationUrl)  
  .description(description)  
  .contact(contact)  
  .license(license)  
  .licenseUrl(licenseUrl)  
  .version(VERSION)  
  .build();  
  }
  ```
 - select()
	 - ApiSelectBuilder 생성
 - apis()
	 - api 스펙이 작성되어 있는 패키지 지정
	 - RequestHandlerSelectors.any()는 프로젝트의 전체 API를 지정
 - paths()
	 - api()로 선택되어진 API중 특정 path 조건에 맞는 API들을 필터링
 - useDefaultResponseMessages()
	 - false로 설정하면, Swagger에서 제공해주는 응답코드(200, 401, 403, 404)에 대한 기본 메시지 제거
	 - 불필요한 응답코드와 메시지를 제거하기 위함이며, 컨트롤러에서 명시해 줌
  - apiInfo()
	 - Swagger API 문서에 대한 설명을 표기
 - tags()
	 - web ui에서 api문서를 정렬하기 위해 사용
 - consumes(), produces()
	 - Request Content-Type, Response Content-Type에 대한 설정
 - 
---
## Spring REST Docs
### 소개
### 설치
### 사용법(예제코드)

## Swagger vs Spring REST Docs

## 마무리





<!--stackedit_data:
eyJoaXN0b3J5IjpbMTIwOTc1Nzg0MCwtNjIxOTM4NDYyLC0zOT
U5MDI4MjIsMTg4OTE5NTU4LC0xNzI5OTk4MjIsLTEyNzIxNDEz
NTksMzUzODE1MDEyLC01MTQwOTU3MDAsMTg0NTA0MTg4NSw2ND
kyOTE0MjYsLTE0ODA5ODgzMjAsLTYzOTUxMTA5NSw2NDY3MTI0
MDksMTg1NTI5MTU4LDE3NTI3NTc5MjYsLTE3NjY3MjI4NDgsNT
A3ODk3NTc3LDY5NzAyNzYyLC00ODI3OTY5MzEsLTQ3NjMyODYx
OF19
-->
