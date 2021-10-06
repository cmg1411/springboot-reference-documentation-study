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
  - 객체를 목킹하는 기본적인 방법
- @Mock
  - 위외 같은 일을 에너테이션으로 할 수 있다.
- @InjectMock
  - 목킹된 객체를 의존주입한 객체를 생성한다.
- @Spy
  - 목킹을 사용하되, 일부 스텁은 원래의 기능을 사용하고 싶을때 쓴다.
  - when().then() 처럼 스텁을 명시적으로 지정하지 않은 부분에 대해서는 실제 구현을 사용한다.
- @MockBean
  - @SpringbootTest 로 서버를 띄운 테스트에서 사용한다.
  - 목킹된 빈이 스프링 빈에 등록된다.
  - @InjectMock 객체가 아닌 @Autowired 객체에 의존주입을 할 때 사용.
- @SpyBean
  - @SpringbootTest 에 사용된다.
  - Spy 객체가 빈으로 등록된다.
  
## JSON Test
```java
data class Car(
    val name: String,
    val price: Int,
    val driven: Int
)
```

```java
@RunWith(SpringJUnit4ClassRunner::class)
@JsonTest
class CarTest {

  @Autowired
  private var json: JacksonTester<Car>? = null

  @Test
  fun serialize() {
    val car = Car(name = "Benz", driven = 10000, price = 50000)

    // car.json 은 resource 에 위치.
    assertThat(json!!.write(car)).isEqualToJson("/car.json")

    assertThat(json!!.write(car)).hasJsonPathStringValue("@.name")
    assertThat(json!!.write(car)).hasJsonPathNumberValue("@.driven")
    assertThat(json!!.write(car)).hasJsonPathNumberValue("@.price")

    assertThat(json!!.write(car)).extractingJsonPathStringValue("@.name").isEqualTo("Benz")
    assertThat(json!!.write(car)).extractingJsonPathNumberValue("@.driven").isEqualTo(10000)
    assertThat(json!!.write(car)).extractingJsonPathNumberValue("@.price").isEqualTo(50000)

    assertThat(json!!.write(car)).extractingJsonPathNumberValue("@.price")
            .satisfies { price -> assertThat(price.toFloat()).isCloseTo(49999f, within(5f)) }
  }


  @Test
  fun deSerialize() {
    val content = "{\"name\":\"Benz\",\"price\":\"50000\",\"driven\":\"10000\"}"
    assertThat(json!!.parse(content)).isEqualTo(Car(name = "Benz", driven = 10000, price = 50000))

    assertThat(json!!.parseObject(content).name).isEqualTo("Benz")
    assertThat(json!!.parseObject(content).driven).isEqualTo(10000)
    assertThat(json!!.parseObject(content).price).isEqualTo(50000)
  }
}
```