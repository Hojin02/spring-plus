# SPRING PLUS
#### 바로가기
1️⃣ [ 코드 개선 퀴즈 - @Transactional의 이해](#a1)  
2️⃣ [ 코드 추가 퀴즈 - JWT의 이해](#a2)  
3️⃣ [ 코드 개선 퀴즈 - AOP의 이해](#a3)  
4️⃣ [ 테스트 코드 퀴즈 - 컨트롤러 테스트의 이해](#a4)  
5️⃣ [ 코드 개선 퀴즈 -  JPA의 이해](#a5)  
6️⃣ [ JPA Cascade ](#a6)  
7️⃣ [ N+1 ](#a7)  
8️⃣ [ QueryDSL ](#a8)  
9️⃣ [ Spring Security ](#a9)  


<a id="a1"></a>
## 1. 코드 개선 퀴즈 - @Transactional의 이해 🔍
### 문제 원인🐛
- TodoService 클래스에   @Transactional(readOnly = true) 어노테이션이 적용되어있었다.
-  이렇게 되면 모든 메소드는 읽기 전용속성으로 트렉젠션이 적용된다.
- 조회가 아닌 저장로직의 메소드에서 읽기 전용 트렌젝션이 전용되므로 문제가 발생한다.⚠️
### 해결 방법
- 클래스에서 트렌젝션을 지운 후 각 메소드에서 트렌젝션을 관리하여 메소드별 속성을 적용하거나,
- 클래스의 적용된 l(readOnly = true) 읽기전용 속성을 지운 후, 조회 메소드에서만 적용을 해주면 된다.✅
<br>
<hr>
<br>

<a id="a2"></a>
## 2. 코드 추가 퀴즈 - JWT의 이해 🔍
### 변경된 요구사항
- User정보에 nickname이 필요해짐.
-  JWT에서 유저의 닉네임을 꺼내 사용해야함.

### 해결 방법
#### users테이블에 nickname컬럼 추가
- User클래스에서 nickname필드 추가 후 생성자도 nickname을 넣어 수정

#### 회원가입 시 사용자에게 닉네임 정보를 함께 받아 회원가입
- SignupRequest에 nickname필드를 추가

#### 토큰 발급을 로직 수정
- JwtUtil클래스에서 createToken 메서드에 인자값으로 닉네임을 받고, 클레임에 닉네임을 추가해줌.

#### 회원가입 로직 수정
- User객체 생성시 SignupRequest에서 닉네임을 꺼내 생성자에 함께 넣어 User객체 생성
-  회원가입 토큰발급을 위해 createToken에 유저정보 닉네임과 함께 넣어줌.

#### 로그인 로직 수정
- 토큰생성 로직 수정됨에 따라 로그인 시 로그인 토큰 발급에 필요한  정보를 수정 해줌.(닉네임 추가)

#### 클라이언트에게 유저의 정보를 넘기기(토큰)
- JwtFilter클래스에서  유효한 토큰일 경우 토큰의 클레임에서 닉네임정보를 꺼내 setAttribute해줌
- AuthUserArgumentResolver에서 resolveArgument로 getAttribute하여 필요한 정보(닉네임)을 꺼내 반환해줌.
<br>
<hr>
<br>

<a id="a3"></a>
## 3. 코드 개선 퀴즈 - AOP의 이해
### 문제 원인🐛
- changeUserRole()의 메소드가 실행전에 동작해야하지만, After로 실행 후에 동작 되도록 되어있다.
-  또 execution경로가 다른 클래스의 다른 매서드로 지정되어있다.
- 수정 전 코드에서는 changeUserRole()메소드가 실행 된 후 동작되도록 되어있어 요구사항에 맞지 않다.⚠️
### 해결 방법
- 먼저 @Before로 실행전에 동작되도록 바꿔준 후
- 경로를 요구사항에 맞게 UserAdminController 클래스의 changeUserRole() 매소드로 지정해준다.
- 마지막으로 메소드명도 맞게 변경해주면 요구사항에 맞게 잘 작동한다.✅
<br>
<hr>
<br>

<a id="a4"></a>
## 4. 테스트 코드 퀴즈 - 컨트롤러 테스트의 이해
### 문제 원인🐛
- todo_단건_조회_시_todo가_존재하지_않아_예외가_발생한다() 메소드의 실패원인은 아래 하나다.
- InvalidRequestException시 글로벌예외처리에서 400 배드리퀘스트로 반환한다.
```java
    @ExceptionHandler(InvalidRequestException.class)
    public ResponseEntity<Map<String, Object>> invalidRequestExceptionException(InvalidRequestException ex) {
        HttpStatus status = HttpStatus.BAD_REQUEST;
        return getErrorResponse(status, ex.getMessage());
    }
```
- 예외가 발생하면 400 BAD_REQUEST가 발생하는데 
```
MockHttpServletResponse:
           Status = 400
    Error message = null
          Headers = [Content-Type:"application/json"]
     Content type = application/json
             Body = {"code":400,"message":"Todo not found","status":"BAD_REQUEST"}
    Forwarded URL = null
   Redirected URL = null
          Cookies = []

```
- Http 상태 OK를 기대하고 있기 때문이다.
```java
 // then
        mockMvc.perform(get("/todos/{todoId}", todoId))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.status").value(HttpStatus.OK.name()))
                .andExpect(jsonPath("$.code").value(HttpStatus.OK.value()))
                .andExpect(jsonPath("$.message").value("Todo not found"));
```

### 해결 방법
- Http status OK를 400 BAD_REQUEST로 바꿔주면된다.
```java
  // then
        mockMvc.perform(get("/todos/{todoId}", todoId))
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.status").value(HttpStatus.BAD_REQUEST.name()))
                .andExpect(jsonPath("$.code").value(HttpStatus.BAD_REQUEST.value()))
                .andExpect(jsonPath("$.message").value("Todo not found"));
```
<br>
<hr>
<br>

<a id="a5"></a>
## 5. 코드 개선 퀴즈 -  JPA의 이해
### 요구 사항 ➕
-  기존 일정 전체 조회에서 날씨와, 수정일을 기준으로 기간 검색 
---
### 해결 방법
- TodoController 컨트롤러 수정(날씨와 기간을 입력 받도록 수정)
``` 조건 : 날씨, 기간의 시작과 끝 파라미터는 필수 입력이 아님.```
```java
@RequestParam(required = false) String weather,
@RequestParam(required = false) String startDate,
@RequestParam(required = false) String endDate
```
를 추가해 입력값이 필수가 아닌 파라미터를 받는다. 그 후 서비스에 값들을 넘긴다. (입력값이 없다면 null)
```java
todoService.getTodos(page, size,weather,startDate,endDate)
```
- TodoService 서비스수정(날짜 형변환 및 기본값 초기화 후 repository로 넘김)
기간의 시작 과 끝 중 어느 하나라도 null이면 최소 최대의 기본값을 설정
```(이걸 적으면서 생각해보니 컨트롤러에서  @RequestParam(defaultValue=..)로 하면 될껄... 라고 늦은 후회)```

```java

LocalDateTime mysqlMinDate = LocalDateTime.parse("1000-01-01T00:00:00");
LocalDateTime mysqlMaxDate = LocalDateTime.parse("9999-12-31T23:59:59");

LocalDateTime start = startDate==null ? mysqlMinDate : LocalDate.parse(startDate).atStartOfDay();
LocalDateTime end = endDate==null ? mysqlMaxDate : LocalDate.parse(endDate).atTime(23, 59, 59);
```
null이면 mysql의 날짜 최소 최대값으로 초기화 아니면 입력값을 LocalDateTime형식으로 형변환
그 후 모든 값들을 repository에 넘긴다 
```java
Page<Todo> todos = todoRepository.findTodosByFiltersOrderByModifiedAtDesc(pageable,weather,start,end);
```
- TodoRepository 수정(쿼리문 수정)
날씨는 null이 넘어올 수 있기 때문에 null이면 전체 날씨 아니면 해당 날씨 기준으로 조회,  
날짜는 service에서 기본값을 설정해줬기 때문에 null은 안넘어옴, 수정일 기준으로 해당 기간 내에 데이터 조회 하도록 변경
```java
 @Query("SELECT t FROM Todo t " +
            "LEFT JOIN FETCH t.user u " +
            "WHERE (:weather IS NULL OR t.weather LIKE :weather) " +
            "AND (t.modifiedAt BETWEEN :startDate AND :endDate) " +
            "ORDER BY t.modifiedAt DESC")
    Page<Todo> findTodosByFiltersOrderByModifiedAtDesc(
            Pageable pageable,
            @Param("weather") String weather,
            @Param("startDate") LocalDateTime startDate,
            @Param("endDate") LocalDateTime endDate
    );
```
```메소드명도 조금 더 어울리게 변경하였다.```
<br>
<hr>
<br>

<a id="a6"></a>
## 6. JPA Cascade
### 문제 원인🐛
- 일정 추가 시 매니저도 같이 추가 되도록 해야함.

### 해결 방법
- cascade를 추가하여 부모 엔티티를 저장할 때 연관된 엔티티도 저장되도록 한다.
```부모 엔티티 삭제 시 연관된 데이터도 삭제 되도록 하려면 CascadeType.ALL를 사용하는게 좋을 것같다```
```java
cascade = CascadeType.PERSIST
```
### Cascade 설정 종류
CascadeType.PERSIST: 부모 엔티티를 저장할 때 연관된 엔티티도 저장.
CascadeType.MERGE: 부모 엔티티를 병합(업데이트)할 때 연관된 엔티티도 병합.
CascadeType.REMOVE: 부모 엔티티를 삭제할 때 연관된 엔티티도 삭제.
CascadeType.ALL: 위 모든 작업(PERSIST, MERGE, REMOVE 등)을 전파.
<br>
<hr>
<br>

<a id="a7"></a>
## 7. N+1
### 문제 원인🐛
- 댓글 전체 조회 시 N+1문제 발생
- 원인 CommentRepository에서 댓글전체 조회하는 findByTodoIdWithUser매서드의 쿼리에서 단순 JOIN하고 있기 때문
### 해결 방법
- JOIN FETCH로 바꿔 데이터를 한꺼면에 가져올 수 있도록 수정해주면된다.
```java
@Query("SELECT c FROM Comment c JOIN FETCH c.user WHERE c.todo.id = :todoId")
```
<br>
<hr>
<br>


<a id="a8"></a>
## 8. QueryDSL
### 요구사항🐛
- todo_단건조회 시 JPQL에서 QueryDSL로 변경
```N+1 문제 발생하지 않도록!```

### 해결 방법
- 우선 bulid.gradle에 쿼리DSL의존성 추가해준다
```gradle
    implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'
    annotationProcessor 'com.querydsl:querydsl-apt:5.0.0:jakarta'
    annotationProcessor "jakarta.annotation:jakarta.annotation-api"
    annotationProcessor "jakarta.persistence:jakarta.persistence-api"
```
- 그 후 쿼리DSL설정 파일 ```QueryDslConfig.java``` 을 추가해준다.
```java
public class QueryDslConfig {

    @PersistenceContext
    private EntityManager entityManager;

    @Bean
    public JPAQueryFactory jpaQueryFactory() {
        return new JPAQueryFactory(entityManager);
    }
}
```
- 쿼리DSL레파지토리 클래스를 추가하여 기존JPQL을 쿼리DSL로 바꿔준다.
```TodoQueryRepository```
```java
@Repository
@RequiredArgsConstructor
public class TodoQueryRepository {
    private final JPAQueryFactory jpaQueryFactory;

    public Optional<Todo> findByIdWithUser(Long todoId) {
        return Optional.ofNullable(
                jpaQueryFactory.selectFrom(todo)
                        .leftJoin(todo.user, user).fetchJoin()
                        .where(
                                todo.id.eq(todoId)
                        )
                        .fetchOne()
        );
    }
```
.fetchJoin()를 사용하여 n+1문제 해결하였다!
<br>
<hr>
<br>


<a id="a9"></a>
## 9. Spring Security
### 요구 사항🐛
- 프로젝트에 Spring Security 적용하기
- 기존 Filter와 Argument Resolver를 사용하던 코드들을 Spring Security로 변경하
- 토큰 기반 인증 방식은 유지할 거예요. JWT는 그대로 사용

### 구현 과정
#### 먼저  Spring Security를 사용하기위해 build.gradle 관련 의존성 추가해주기!
```gradle
implementation 'org.springframework.boot:spring-boot-starter-security'
```
크게 수정할 부분은 JwtFillter이고, SecurityConfig파일을 추가해준다.
그 후 AuthUser클래스를 스프링 시큐리티에서 사용할 수 있도록 수정해준다.   

```java

@Getter
public class AuthUser implements UserDetails {

    private final Long id;
              :
    public AuthUser(Long id, String email, String nickname, UserRole userRole) {
       :
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return Collections.singletonList(userRole::getRole);
    }

    @Override
    public String getPassword() {
        return "";
    }

    @Override
    public String getUsername() {
        return this.email;
    }

```
기존 AuthUser를 시큐리티에서 사용하기 위해 UserDetails 를 임플리먼츠 해준후 필요한 메소드를 오버라이딩 해준다.
JwtFilter에서는 토큰의 유효성 검사만 한 후 토큰에서 유저정보를 꺼내 시큐리티에 유저 정보를 넘긴다!
```java
Long userId =  Long.parseLong(claims.getSubject());
            UserRole userRole = UserRole.valueOf(claims.get("userRole", String.class));
            String userEmail = (String) claims.get("email");
            String userNickName = (String) claims.get("nickname");

            UserDetails authUser = new AuthUser(userId,userEmail,userNickName,userRole);
            SecurityContextHolder
                    .getContext()
                    .setAuthentication(
                            new UsernamePasswordAuthenticationToken(
                                    authUser, null, authUser.getAuthorities()));

```
```기존 JwtFilter에서 url검증 부분 지우고, Argument Resolver에 값을 넘겨주는 부분을 지워줌```
시큐리티에서는 JwtFilter에서 받은 유저 정보를 url검증과 권한 인증인가를 실시한다.
```java
 @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
                .csrf(AbstractHttpConfigurer::disable)  // .csrf().disable() 방식은 더 이상 사용 안함.
                .httpBasic(AbstractHttpConfigurer::disable) // BasicAuthenticationFilter 비활성화
                .formLogin(AbstractHttpConfigurer::disable) // UsernamePasswordAuthenticationFilter,
                // DefaultLoginPageGeneratingFilter 비활성화
                .addFilterBefore(jwtFilter, SecurityContextHolderAwareRequestFilter.class)
                .authorizeHttpRequests(auth -> auth
                                .requestMatchers("/*","/auth").permitAll()
                                .requestMatchers("/admin/**").hasRole("ADMIN")
                                .anyRequest().authenticated()
                )
                .exceptionHandling(exceptions -> exceptions
                        .accessDeniedHandler((request, response, accessDeniedException) -> {
                            response.sendError(HttpServletResponse.SC_FORBIDDEN, "관리자 권한이 없습니다.");
                        })
                )
                .build();
    }
```
컨트롤러에서는 @Auth로 유저정보를 가져오던 부분을 @AuthenticationPrincipal로 받아와 사용할 수 있다.

생각해보니 @Auth 어노테이션을 만들었는데 안쓰게 되니 AuthenticationPrincipal를 @Auth에서 작동되게 할 수 있지 않나 싶다..
(컨트롤러 하나하나 AuthenticationPrincipal로 바꿔줌)
