# Test

- spring-boot-starter-test 를 통해 쉬운 단위테스트, 통합테스트를 가능하게 해준다.

## @SpringbootTest
- Spring boot 기능이 필요한 테스트에 사용
- JUnit5 는 상관없지만, JUnit4 는 @RunWith(SpringRunner.class) 를 추가해야한다.
    - 원래 JUnit 은 각 테스트마다 객체를 새로 만든다.
    - @RunWith(SpringRunner.class) 을 쓰면, ApplicationContext 를 생성하여 스프링부트처럼 싱글톤 객체를 만들어 모든 테스트에 사용하게 해준다.
- WebEnvironment 옵션
    - `MOCK(기본)` : 임베디드 톰켓이 뜨지 않음. `ApplicationContext` 을 로드하고 목킹된 서블릿으로 컨테이너가 뜸.
    - `RANDOM_PORT` : 랜덤한 포트로 임베디드 톰켓을 띄움.
    - `DEFINE_PORT` : application.properties 에 정의된 포트 번호로 띄움. 설정 안하면 8080
    - `NONE` : `ApplicationContext` 를 로드하지만, web 환경을 제공하지 않음.
```java
RANDOM_PORT,DEFINE_PORT 를 사용할 경우 @Transactional 에너테이션을 사용해도 롤백이 되지 않는다.
```
- properties 옵션
    - MVC 또는 WebFlux 중 하나만 사용한다면 자동으로 인식하지만, 둘 다 사용하는 경우 이 옵션으로 지정 가능.

## 실제 서버 테스트
- Spring MVC
```java
import org.jetbrains.annotations.TestOnly;
import org.springframework.boot.web.server.LocalServerPort;

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class RandomPortTestRestTemplateExampleTests {

   @LocalServerPort
   int port;

   @Autowired 
   TestRestTemplate testRestTemplate;
   
   @Test
   void test1() {
       assertThat(port).isEqualTo(8080);
   }

   @Test
   void test2() {
      String body = restTemplate.getForObject("/", String.class);
      assertThat(body).isEqualTo("Hello World");
   }
}
```

- WebFlux
```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class RandomPortWebTestClientExampleTests {

    @Test
    void exampleTest(@Autowired WebTestClient webClient) {
        webClient.get().uri("/").exchange().expectStatus().isOk().expectBody(String.class).isEqualTo("Hello World");
    }

}
```

## Mock 서버 테스트

- @WebMvcTest

```java
@WebMvcTest
public class WebMvcTest {

   @Autowired
   private MockMvc mockMvc;

   @Test
   void test() {
       mockMvc.perform(get("hello")).param("name", "tomas")
               .andExpect(status().isOk())
               .andDo(print());
   }
}
```

- @AutoConfigureMockMvc

```java
import org.springframework.beans.factory.annotation.Autowired;

@AutoConfigureMock
@SpringBootTest
public class AutoConfigureMock {

   @Autowired
   private MockMvc mockMvc;

   @Test
   void test() {
      mockMvc.perform(get("hello")).param("name", "tomas")
              .andExpect(status().isOk())
              .andDo(print());
}
```

- 차이점
    - @AutoConfigureMockMvc 는 @Controller, @RestController, @Service, @Repository 모두 사용.
    - @WebMvcTest 는 @Controller, @RestController 만 메모리에 올림.
    - @WebMvcTest 는 @SpringBootTest 기본 옵션과 같이 사용할 수 없다.
        - @WebMvcTest 도 목킹한 서블릿 컨테이너를 사용하기 때문에 서로 충돌이 난다.


## Mock
- Mock.mock()
- @Mock
- @InjectMock
- @MockBean
- @Spy
- @SpyBean

## JSON Test