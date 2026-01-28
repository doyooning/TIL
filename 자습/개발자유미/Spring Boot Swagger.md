#Java 
#Spring 

---
https://cafe.naver.com/xxxjjhhh/94
개발자 유미 강의 참조


# API 명세
**엔드포인트 공개**
서버에 존재하는 자원은 API를 통해 End User 또는 다른 개발자들에게 제공되는데
이런 API를 사용하기 위해선 “기능, 요청 방법 (세부), 응답 형식”을 파악해야 함

따라서 개발한 API를 잘 활용하도록 API 명세서 (스펙)를 작성해야 함

**API 명세서 획일화**
하지만 전세계에는 수많은 API가 존재하고 ..
각각의 개발자들이 자신만의 형태로 명세서를 작성하게 된다면 
새로운 API를 사용할 때 마다 명세서의 형태를 파악해야 하는 어려움이 있음

이 문제를 해결하기 위해 국제적으로 API 명세 표준을 몇 가지 만들어 두었음

- OpenAPI (OAS)
- RAML
- API Blueprint

이 중 가장 많이 사용되는 명세서 작성 형식은 OpenAPI이며 내부적으로 JSON/YAML 형태의 포맷이 있음

**OAS**
OpenAPI는 버전이 존재
현재 3.0 이라 불리는 3.X 번대의 버전이 stable하고 많이 사용됨

**OAS를 기반으로 시각화**
작성한 OAS를 Swagger UI에 넣는다면 흔히 Swagger라 부르는 시각화를 진행할 수 있음

![[Pasted image 20260128120410.png]]

# 엔드포인트 OAS 생성
**의존성 추가**
스프링 부트 컨트롤러 기반으로 OpenAPI 명세서를 생성하기 위해 관련된 의존성을 추가하겠습니다.

build.gradle
```
implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.8.8'
```
주로 사용되는 버전은 2.2.0, 2.3.0

**엔드포인트**
OAS
- JSON : /v3/api-docs
- YAML : /v3/api-docs.yaml

Swagger UI
- /swagger-ui/index.html

**OAS 설명 설정**
'우리 서버는 이러한 서버다'라고 OAS에 명시할 수 있음 
application.properties 또는 Config 클래스를 통해 진행

OpenApiConfig
```java
import io.swagger.v3.oas.models.OpenAPI;  
import io.swagger.v3.oas.models.info.Info;  
import io.swagger.v3.oas.models.servers.Server;

@Configuration  
public class OpenApiConfig {  
  
    @Bean  
    public OpenAPI openAPI() {  
  
        return new OpenAPI()  
                .info(new Info()  
                        .title("개발자 윤도 API 목록")  
                        .description("개발자 윤도 스프링 부트 Swagger 시리즈 실습을 위한 API 목록입니다.")  
                        .version("v1.0.0"))  
                .servers(List.of(  
                        new Server()  
                                .url("http://localhost:8080")  
                                .description("개발용 서버")  
                ));  
    }  
}
```

**샘플 엔드포인트**
ContentController
```java
@RestController  
@RequestMapping("/api/v1")  
public class ContentController {  
  
    @GetMapping("/content/{id}")  
    public ResponseEntity<?> contentGet(  
            @PathVariable("id")Long id  
    ){  
  
        Map<String, Object> resultBody = Map.of(  
                "id", id,  
                "title", "제목" + id,  
                "content", "내용" + id  
        );  
  
        HttpHeaders httpHeaders = new HttpHeaders();  
        httpHeaders.setContentType(new MediaType("application", "json"));  
  
        return new ResponseEntity<>(resultBody, httpHeaders, HttpStatus.OK);  
    }  
  
    @PostMapping("/content")  
    public ResponseEntity<?> contentPost(  
            @RequestBody ContentRequestDTO dto  
    ) {  
  
        Map<String, Object> resultBody = Map.of("id", 1L);  
  
        HttpHeaders httpHeaders = new HttpHeaders();  
        httpHeaders.setContentType(new MediaType("application", "json"));  
  
        return new ResponseEntity<>(resultBody, httpHeaders, HttpStatus.OK);  
    }  
  
    @DeleteMapping("/content/{id}")  
    public ResponseEntity<?> contentDelete(  
            @PathVariable("id") Long id  
    ) {  
  
        HttpHeaders httpHeaders = new HttpHeaders();  
        httpHeaders.setContentType(new MediaType("application", "json"));  
  
        return new ResponseEntity<>(httpHeaders, HttpStatus.OK);  
    }  
}
```


# 엔드포인트 그룹화
**엔드포인트 도메인**
서비스가 가지는 엔드포인트는 도메인별로 특징이 있음
게시글 관련 CRUD, 회원 관련 CRUD, ...
따라서 이런 엔드포인트들을 그룹화 하는 방법이 필요

**@Tag**
ContentController
```java
@Tag(name = "Content API", description = "게시글 도메인 API") 
@RestController 
@RequestMapping("/api/v1") 
public class ContentController {

}
```

**여러 클래스에 대한 동일 그룹화**
동일한 @Tag를 각 클래스별로 부여하면 적용


**기타 : 대그룹화**
“/api/v1”, “/api/v2”와 같이 버전별 진행하는 방법

OpenApiConfig
```java
@Bean 
public GroupedOpenApi groupedOpenApiV1() { 
	return GroupedOpenApi.builder() 
		.group("v1") 
		.pathsToMatch("/api/v1/**") 
		.build(); 
} 

@Bean 
public GroupedOpenApi groupedOpenApiV2() {  
	return GroupedOpenApi.builder() 
		.group("v2") 
		.pathsToMatch("/api/v2/**") 
		.build(); 
}
```


# 엔드포인트 명세
**엔드포인트에 대한 명세**
명세는 “엔드포인트 역할”, “요청 방법”, “응답 종류”에 대해 어노테이션 기반으로 작성

**@Operation**
위 명세를 작성할 수 있는 어노테이션
`@Operation(...)`

엔드포인트에 대한 설명
```java
@Operation( 
	summary = "게시글 Read", 
	description = "게시글의 ID를 파라미터로 보내면 해당하는 게시글 조회"
)
```

**요청에 대한 명세**
- Query 파라미터 변수
```java
parameters = { 
	@Parameter( 
		name = "id", 
		description = "조회할 게시글 ID", 
		required = true, 
		in = ParameterIn.QUERY 
	) 
}
```

- Path 파라미터 변수
```java
parameters = { 
	@Parameter( 
		name = "id", 
		description = "조회할 게시글 ID", 
		required = true, 
		in = ParameterIn.PATH 
	) 
}
```

- JSON Body
```java
requestBody = @io.swagger.v3.oas.annotations.parameters.RequestBody(
    description = "게시글 JSON Body 데이터",
    required = true,
    content = @Content(
        mediaType = "application/json",
        schema = @Schema(implementation = ContentRequestDTO.class)
    )
),
```


**응답에 대한 명세**
- 상태 코드에 따른 응답 종류
```java
responses = {
    @ApiResponse(
        responseCode = "200",
        description = "성공"
    ),
    @ApiResponse(
        responseCode = "400",
        description = "실패"
    )
}
```

- JSON Body
```java
@ApiResponse(
    responseCode = "200",
    description = "성공",
    content = @Content(
        mediaType = "application/json",
        schema = @Schema(implementation = ContentResponseDTO.class)
    )
)
```

> 로직을 작성해두고 ChatGPT에게 시킨다...ㅎ

# 시큐리티 추가시
Swagger가 적용된 상태에서 스프링 시큐리티를 추가하게 된다면 몇가지 상황을 고려해야 함

**Swagger UI 경로 permitAll 적용시**
Swagger UI 경로인 “/swagger-ui/index.html”을 모든 사용자에게 공개하기 위해 permitAll을 설정한다면 OAS 경로도 함께 permitAll 해야 함

- SecurityConfig 예시
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {

    http
            .authorizeHttpRequests(auth -> auth
                    .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll());

    return http.build();
}
```

**요청 헤더에 JWT가 들어가는 경우 명세 작성**
시큐리티에 의해 요청 헤더에 JWT가 필요한 경우 명세서에도 표기해야 함 
(Swagger 테스트시도 필요)

- OpenApiConfig
```java
@Bean
public OpenAPI openAPI() {

    return new OpenAPI()
            .info(new Info()
                    .title("개발자 윤도 API 목록")
                    .description("개발자 윤도 스프링 부트 Swagger 시리즈 실습을 위한 API 목록입니다.")
                    .version("v1.0.0")
            )
            .servers(List.of(
                    new Server()
                            .url("http://localhost:8080")
                            .description("개발용 서버")
            ))
            .components(new Components()
                    .addSecuritySchemes("JWT", new SecurityScheme()
                            .type(SecurityScheme.Type.HTTP)
                            .scheme("bearer")
                            .bearerFormat("JWT")
                            .in(SecurityScheme.In.HEADER)
                            .name("Authorization")
                    )
            );
}
```

**- 특정 엔드포인트에 대해 : Operation()**
```java
security = @SecurityRequirement(name = "JWT"),
```
`addSecuritySchemes()`에 명시된 보안 스킴 name 설정으로 적용됨

**로그인/로그아웃 엔드포인트 문제**
스프링 시큐리티의 인증/인증 해제 엔드포인트는 모두 필터단에서 처리됨
따라서 Swagger 적용이 까다로움...

이 문제를 해결하기 위해 Swagger는 기본 로그인에 대해서 엔드포인트 활성화를 제공함

- application.properties
```java
springdoc.show-login-endpoint=true
springdoc.show-oauth2-endpoints=true
```
!! 문제는 로그인 필터를 커스텀하게 된다면 적용되지 않음

**- 편법**
로그인/로그아웃은 모두 필터단에서 처리됨
따라서 해당 엔드포인트에 대한 서블릿 Mapping을 만들어도 도달하지 않음

이를 바탕으로 로그인/로그아웃 가짜 컨트롤러 엔드포인트를 만든 뒤 (실제 동작은 수행하지 않는), 
Swagger 어노테이션을 붙여 명세가 완성되도록 설정하면 됨
