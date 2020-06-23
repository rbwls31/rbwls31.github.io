## REST API 문서 자동화 하기

 #### 웹 서비스 구조
 <br>
<img src="https://raw.githubusercontent.com/rbwls31/rbwls31.github.io/master/images/WEB.png" width="85%">
<br><br>

- API서버가 어떤 Spec을 가진 데이터를 주고 받는지에 대한 문서 필요
- 엑셀 기반의 문서 관리 -> **비효율적**
	- 작성에 시간이 많이 걸림
	- 기능이 추가되거나 변경되면 따로 작업 필요

## Swagger
Swagger는 간단한 설정으로 프로젝트에서 지정한 URL들을 HTML 화면으로 확인할 수 있게 해주는 프로젝트로, Java뿐만 아니라 Nodejs, Python 등 다양한 언어를 지원

### 1. 의존성 추가
maven pom.xml
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

### 2. Config 설정
Swagger 설정을 위한 SwaggerConfig 클래스를 생성
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
	.paths(PathSelectors.any())  
	.build()  
}  
```
- Swagger는 @EnableSwagger2 어노테이션을 통해 사용 가능함
- Docket빈을 정의 후, `select()` 메소드를 통해 `ApiSelectorBuilder` 인스턴스를 리턴받아 Swagger에 의해 노출되는 끝단(endpoint)을 제어
- RequestHandlers의 selection을 위한 서술부는 `RequestHandlerSelectors` 와 `PathSelectors`  를 통해 설정함. 둘 다 `any()` 를 쓰면 사용자의 전체 API가 Swagger를 통해 문서화 됨
- 상세설정 : [https://springfox.github.io/springfox/docs/current/#configuration-explained](https://springfox.github.io/springfox/docs/current/#configuration-explained)

### 3. API 어노테이션 명세
Controller에 어노테이션을 이용하여 API를 명세
```java
@Api(tags = {TAG_EVENT_BY_USER})  
@RestController  
@RequestMapping("events")  
public class EventUserController {  
  
  private EventService eventService;  
  public EventUserController(EventService eventService) {  
	  this.eventService = eventService;  
  }  
  @ApiOperation(value = "이벤트 생성")  
  @PostMapping("")  
  @ResponseStatus(code = HttpStatus.CREATED)  
  public void createEvent(@RequestBody CreateEventRequest createEventRequest, Principal principal) {  
	  eventService.createEventAndDefaultProgramAndSampleActivityAndParticipantManager(createEventRequest, principal.getName());  
  }  
  
  @ApiOperation(value = "이벤트 정보 조회")  
  @GetMapping("{eventId}")  
  @ResponseStatus(code = HttpStatus.OK)  
  public EventInformationResponse retrieveEvent(@PathVariable String eventId) {  
	  return eventService.retrieveEvent(eventId);  
  }  
  
  @ApiOperation(value = "이벤트 목록 조회")  
  @GetMapping("")  
  @ResponseStatus(code = HttpStatus.OK)  
  public EventInformationListResponse retrieveEvents(@ApiParam(value = "이벤트 코드", required = true) @RequestParam String code) {  
	  return new EventInformationListResponse();  
  }  
}
```

- **@Api**
	- 해당 클래스가 Swagger 리소스라는 것을 명시
		- description : api 설명
		- tags : api 그룹명
- **@ApiOperation**
	- API URL과 Method 선언
		- value : API에 대한 간략한 설명
		- notes : 상세 설명	
- **@ApiParm**
	- API 호출 시 전달되는 파라미터에 대한 정보 명시
		- value : 파라미터명
		- required : 필수 여부
		- example : 파라미터값 예시

### 4. API Model 명세
DTO 클래스에 어노테이션을 이용하여 API Model을 명세
```java
@Getter @Setter  
@ApiModel(value = "이벤트 생성", description = "이벤트 생성 요청 객체")  
public class CreateEventRequest {  
  //  
  @ApiModelProperty(value = "이벤트 제목", required = true)  
  @NotBlank(message = ValidatorMessage.EVENT_TITLE_BLANK_ERR)  
  private String title;  
  
  @ApiModelProperty(value = "이벤트 시작 일자", required = true)  
  @JsonFormat(pattern = "yyyy-MM-dd")   
  @NotNull(message = ValidatorMessage.EVENT_START_DATE_TIME_BLANK_ERR)  
  private LocalDate startDate;  
  @ApiModelProperty(value = "이벤트 종료 일자", required = true)    
  @JsonFormat(pattern = "yyyy-MM-dd")  
  @NotNull(message = ValidatorMessage.EVENT_END_DATE_TIME_BLANK_ERR)
  private LocalDate endDate;  
  
  @ApiModelProperty(value = "이벤트 시작 시간", required = true)    
  @JsonFormat(pattern = "HH:mm")  
  @NotNull(message = ValidatorMessage.EVENT_START_DATE_TIME_BLANK_ERR)  
  private LocalTime startTime;  
  @ApiModelProperty(value = "이벤트 종료 시간", required = true) 
  @JsonFormat(pattern = "HH:mm")   
  @NotNull(message = ValidatorMessage.EVENT_END_DATE_TIME_BLANK_ERR)  
  private LocalTime endTime;   
}
```
- **@ApiModel**
	- VO, DTO, Entity 등 모델에서 사용
		- value : 모델 이름을 지정
		- description : 상세 설명
- **@ApiModelProperty**
	- 모델 내의 필드를 설명
		- value : 필드 이름
		- required : 필수 여부
		- example : 필드값 예시
		- hidden : 필드 숨김 여부

### 5. web-ui
[http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html)

@Api 어노테이션에서 정의한 tags 순서로 API들이 나열됨
<img src="https://raw.githubusercontent.com/rbwls31/rbwls31.github.io/master/images/swagger-ui-api-2.PNG" width="85%">

## Spring REST Docs
Spring REST Docs은 테스트 코드 기반으로 RESTful 문서생성을 돕는 도구로 기본적으로 Asciidoctor를 사용하여 HTML을 생성

### 1. 의존성 추가
maven pom.xml
```
<dependencies>  
 ...  
 <!-- (1) -->  
	 <dependency>  
		 <groupId>org.springframework.restdocs</groupId>  
		 <artifactId>spring-restdocs-mockmvc</artifactId>  
		 <scope>test</scope>  
	 </dependency>  
</dependencies>  
  
<plugins>  
	<plugin>  
	 <!-- (2) -->  
	<groupId>org.asciidoctor</groupId>  
	<artifactId>asciidoctor-maven-plugin</artifactId>  
	<version>1.5.3</version>  
	<executions>  
		<execution>  
			<id>generate-docs</id>  
			<phase>prepare-package</phase>  
			<goals>  
				<goal>process-asciidoc</goal>  
			</goals>  
			<configuration>  
				<backend>html</backend>  
				<doctype>book</doctype>  
			</configuration>  
		</execution>  
	</executions>  
	<dependencies>  
	<!-- (3) -->  
		<dependency>  
			<groupId>org.springframework.restdocs</groupId>  
			<artifactId>spring-restdocs-asciidoctor</artifactId>  
			<version>2.0.2.RELEASE</version>  
		</dependency>  
	</dependencies>  
	</plugin>  
  
	 <plugin>  
	 <!-- (4) -->  
		 <artifactId>maven-resources-plugin</artifactId>  
		 <version>2.7</version>  
		 <executions>  
			 <execution>  
			 <id>copy-resources</id>  
			 <phase>prepare-package</phase>  
			 <goals>  
				 <goal>copy-resources</goal>  
			 </goals>  
			 <configuration>  
				 <outputDirectory>  
					 ${project.build.outputDirectory}/static/docs  
				 </outputDirectory>  
				 <resources>  
					 <resource>  
					 <directory>  
						 ${project.build.directory}/generated-docs  
					 </directory>  
					 </resource>  
				 </resources>  
			 </configuration>  
		 </execution>  
		 </executions>  
	 </plugin>  
</plugins>
```
- (1) `restdocs` 의존성을 추가, scope는 test로 지정.
- (2) `asciidoctor` 플러그인을 추가.
- (3) `spring-restdocs-asciidoctor`  의존성을 추가, 해당 의존성이 추가되면 `snippets`  이 자동으로 구성됨.
- (4) `Asciidoctor`  플러그인 설정.
    -   `outputDirectory`  문서가 출력되는 디렉토리 경로. 실제 /target/classes/static/docs/`  경로에 문서가 생성됨

### 2. 테스트 코드 설정
Spring Rest Docs는  `JUnit`  `TestNG`  등 다양한 테스트 프레임워크를 지원

> 본 예제는  `JUnit`  기반으로 테스트를 진행 하기 때문에  `JUnitRestDocumentation`  객체를
> 생성하였음
```java
@RunWith((SpringRunner.class))  
@SpringBootTest  
public class EventUserControllerTest {  
  
	@Rule  
	public JUnitRestDocumentation restDocumentation = new JUnitRestDocumentation();  
	@Autowired  
	private WebApplicationContext context;  
	private MockMvc mockMvc; //(1)  
	private RestDocumentationResultHandler document;  
	@Autowired  
	private ObjectMapper objectMapper;  
	@MockBean  
	private EventService eventService;
	
	//(2)
	@Before  
	public void setUp() {  
		this.document = document(  
			"{class-name}/method-name",
			preprocessRequest(prettyPrint()),  
			preprocessResponse(prettyPrint())  
		);  

		this.mockMvc = MockMvcBuilders.webAppContextSetup(this.context)  
			.apply(documentationConfiguration(this.restDocumentation))  
			.alwaysDo(document)  
			.build();  
		}  
}
```
-   (1)  `MockMvc`,  `WebTestClient`,  `Rest Assured`  등 다양한 방식으로 Controller에 대한 테스트를 진행 할 수 있음
-   (2)  `RestDocumentationResultHandler`  객체와  `MockMvc`  객체를 생성함
    -   `{class-name}/{method-name}`  설정은 해당 테스트 클래스의 이름, 메서드 이름 기반으로 디렉토리 경로를 설정해서  `snippets`을 생성
    -  `preprocessRequest(prettyPrint())`, `preprocessResponse(prettyPrint())`  설정을 통해 해당 문서가 이쁘게 출력됨
    -   `alwaysDo()`  메서드로 위에서 생성된  `RestDocumentationResultHandler`  객체를 의존성 주입함 (모든  `mockMvc`  테스트에 대한  `snippets`이 생성됨)

### 3. 이벤트 생성 테스트 코드 작성
```java
@Test  
public void createEvent() throws Exception {  
	CreateEventRequest createEventRequest = new CreateEventRequest();  
	createEventRequest.setTitle("솔루션연구팀 전사 세미나");  
	createEventRequest.setStartDate(LocalDate.of(2020, 6, 30));  
	createEventRequest.setStartTime(LocalTime.of(9, 0));  
	createEventRequest.setEndDate(LocalDate.of(2020, 7, 1));  
	createEventRequest.setEndTime(LocalTime.of(18, 0));  
  
	mockMvc.perform(post("/events")  
		.contentType(MediaType.APPLICATION_JSON)  
		.content(objectMapper.writeValueAsString(createEventRequest))  
		.accept(MediaType.APPLICATION_JSON))  
		.andDo(print())  
		.andExpect(status().isCreated())  
		.andDo(document.document(
			//(1)  
			requestFields(  
				fieldWithPath("title").description("이벤트 제목"),  
				fieldWithPath("startDate").description("이벤트 시작일자"),  
				fieldWithPath("startTime").description("이벤트 시작일시"),  
				fieldWithPath("endDate").description("이벤트 종료일자"),  
				fieldWithPath("endTime").description("이벤트 종료일시")  
			)
		));  
}  
```
- (1) `requestFields` 를 사용해서 파라미터를 정의

### 4. 이벤트 조회 테스트 코드 작성
```java
@Test  
public void retrieveEvent() throws Exception {  
	EventInformationResponse eventInformationResponse = new EventInformationResponse();  
	eventInformationResponse.setId("1");  
	eventInformationResponse.setTitle("솔루션연구팀 전사 세미나");  
	eventInformationResponse.setStartDate(LocalDate.of(2020, 6, 30));  
	eventInformationResponse.setStartTime(LocalTime.of(9, 0));  
	eventInformationResponse.setEndDate(LocalDate.of(2020, 7, 1));  
	eventInformationResponse.setEndTime(LocalTime.of(18, 0));  

	given(eventService.retrieveEvent("1")).willReturn(eventInformationResponse);  
   
	mockMvc.perform(RestDocumentationRequestBuilders.get("/events/{eventId}", "1")  
		.accept(MediaType.APPLICATION_JSON))  
		.andDo(print())  
		.andExpect(status().isOk())  
		.andDo(document.document(  
		(1)
		pathParameters(  
			parameterWithName("eventId").description("이벤트 아이디")  
		),
		(2)  
		responseFields(  
			fieldWithPath("id").description("이벤트 아이디"),  
			fieldWithPath("title").description("이벤트 제목"),  
			fieldWithPath("startDate").description("이벤트 시작일자"),  
			fieldWithPath("startTime").description("이벤트 시작일시"),  
			fieldWithPath("endDate").description("이벤트 종료일자"),  
			fieldWithPath("endTime").description("이벤트 종료일시")  
		) ))  
		.andExpect(jsonPath("id", is(notNullValue())))  
		.andExpect(jsonPath("title", is(notNullValue())))  
		.andExpect(jsonPath("startDate", is(notNullValue())))  
		.andExpect(jsonPath("startTime", is(notNullValue())))  
		.andExpect(jsonPath("endDate", is(notNullValue())))  
		.andExpect(jsonPath("endTime", is(notNullValue())));  
}
```
- (1) pathParameter 정의
- (2) `responseFields` 메서드로 response 정의

### 5. web-ui
테스트 코드 빌드 시 아래와 같은 `snippets` 파일이 생성됨

<img src="https://raw.githubusercontent.com/rbwls31/rbwls31.github.io/master/images/snippets.PNG" width="20%">

web-ui를 위해 `src/main/asccidoc/` 경로에 `api-docs.adoc` 파일을 생성하고 테스트 코드를 통해 생성된 `snippets` 파일들을 아래와 같이 추가 함
```
= Lerni 2.0 API  
:doctype: book  
:icons: font  
:source-highlighter: highlightjs  
:toc: left  
:toclevels: 2  
:sectlinks:  
  
== Event  
  
=== Event 생성  
.request  
include::{snippets}/event-user-controller-test/create-event/http-request.adoc[]  
include::{snippets}/event-user-controller-test/create-event/request-fields.adoc[]  
  
.response  
include::{snippets}/event-user-controller-test/create-event/http-response.adoc[]  
include::{snippets}/event-user-controller-test/create-event/response-body.adoc[]  
  
=== Event 조회  
.request  
include::{snippets}/event-user-controller-test/retrieve-event/http-request.adoc[]  
include::{snippets}/event-user-controller-test/retrieve-event/path-parameters.adoc[]  
  
.response  
include::{snippets}/event-user-controller-test/retrieve-event/response-body.adoc[]  
include::{snippets}/event-user-controller-test/retrieve-event/http-response.adoc[]  
include::{snippets}/event-user-controller-test/retrieve-event/response-fields.adoc[]
```
최종적으로 `maven install`을 하게 되면 `target/classes/static/docs/` 경로에 html 파일이 생성됨

[http://localhost:8080/docs/api-docs.html](http://localhost:8080/docs/api-docs.html)

위에서 설정한 목차에 의해 api 들이 나열됨

<img src="https://raw.githubusercontent.com/rbwls31/rbwls31.github.io/master/images/rest-doc-web-ui.PNG" width="70%">

## Swagger vs Spring REST Docs
||Swagger|Spring Rest Docs|
|---|---|---|
|장점|API를 테스트 해 볼 수 있는 화면 제공  |제품 코드에 영향 없음|
||적용하기 쉬움|테스트가 성공해야 문서가 작성됨|	
|단점|코드에 어노테이션을 추가해야 됨|적용하기 어려움|
|||코드와 동기화 안될 수 있음|


## 마무리
Swagger와 Spring REST Docs을 비교해 봤을 때, 
코드와 문서의 일치성, API 문서의 형태 면에서 Spring REST Docs이 좀 더 우위에 있는 것 같다.
하지만 Spring REST Docs의 경우 모든 API에 대한 테스트 코드를 작성해야 되기 때문에 프로젝트 설계 일정을 고려해야 한다.


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTIwOTc1Nzg0MCwtNjIxOTM4NDYyLC0zOT
U5MDI4MjIsMTg4OTE5NTU4LC0xNzI5OTk4MjIsLTEyNzIxNDEz
NTksMzUzODE1MDEyLC01MTQwOTU3MDAsMTg0NTA0MTg4NSw2ND
kyOTE0MjYsLTE0ODA5ODgzMjAsLTYzOTUxMTA5NSw2NDY3MTI0
MDksMTg1NTI5MTU4LDE3NTI3NTc5MjYsLTE3NjY3MjI4NDgsNT
A3ODk3NTc3LDY5NzAyNzYyLC00ODI3OTY5MzEsLTQ3NjMyODYx
OF19
-->
