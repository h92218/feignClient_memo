
RestTemplate
동기식 => 대량의 요청을 처리할 경우에는 성능이슈 발생 가능 , 스프링4부터 AsyncRestTemplate으로 비동기 처리 가능 

WebClient
비동기 및 리액티브 처리시에 유용
논블로킹 I/O를 사용하여 병렬 처리 및 높은 확장성
함수형 프로그래밍 스타일, 람다 표현식을 사용하여 요청 및 응답을 정의할 수 있음


feign
동기식
마이크로 서비스 간 통신을 쉽게 구현할 수 있는 라이브러리
선언적 API정의 => 코드가 간결하고 읽기 쉬움
인터셉터 제공 => 보안, 로깅 및 기타 요구사항 처리
스프링 부트, 스프링 클라우드와 함께 사용하면 효과적으로 작동


*****

*구성*

1. @EnableFeignClients 를 붙여야 됨
@SpringBootApplication이 붙은 파일에 붙여줌 

2. feign client 파일을 만든다.
<pre><code class="java">

@FeignClient(name="feignClientUtil",url="${app.service.common}", configuration = FeignClientConfiguration.class)
public interface FeignClientUtil {
    
    @GetMapping(value="{servicePath}")
    JSONObject get(@PathVariable String servicePath);

    @GetMapping(value="{servicePath}")
    JSONObject getWithParam(@PathVariable String servicePath,@SpringQueryMap Map<String,Object> queryParam);

    @PostMapping(value="{servicePath}")
    JSONObject post(@PathVariable String servicePath, @RequestBody Object param);
   
}
</code></pre>

3. 설정파일은 필요하면 만든다.


*****

*Feign Client 설정내용*
<pre><code class="yaml">

connectTimeout: 5000   
#서버에 연결을 시도하는 최대 시간. 이 시간 내에 서버에 연결하지 못하면 예외가 발생   


readTimeout: 5000          
#서버로부터 응답을 받는 최대 시간. 이 시간 내에 응답이 도착하지 않으면 예외가 발생              
                    
loggerLevel: FULL     
#로깅레벨. NONE, BASIC, HEADERS, FULL
                          
errorDecoder: com.example.SimpleErrorDecoder 
#서버 응답에서 발생하는 오류를 해석하는 데 사용되는 커스텀 에러 디코더를 지정.   
#Feign은 가장 기본적으로 4xx, 5xx에 대해 ErrorDecoder.default를 이용하며 FeignException 를 반환함

retryer: feign.Retryer$Default
#요청이 실패할 경우 재시도 동작정의
#설정 하지 않으면 기본 Retyer은 NEVER.RETRY로 설정되므로 재시도를 안 함
#Default retryer은 5번까지 함 

requestInterceptors:
#Feign 클라이언트에서 발생하는 각 요청에 헤더 추가, 요청 수정 등의 작업을 수행하는 인터셉터 정의 

decode404: false
# 404응답이 올 때 FeignExeption을 발생시킬지, 아니면 응답을 decode할 지 여부

encoder: com.example.SimpleEncoder
decoder: com.example.SimpleDecoder
#응답의 데이터 형식을 변환 
#예) JSON 데이터 -> DTO 디코더 & DTO -> JSON 데이터 인코더 구성 

contract: com.example.SimpleContract
#Feign 인터페이스의 메서드를 서버 요청으로 변환하는 방법을 정의
#기본적으로 Feign은 Spring Cloud 프로젝트의 SpringMvcContract를 사용하여 Spring MVC 애노테이션을 해석합
</code></pre>


*****



설정하기
클래스파일 설정과 yml파일에 특정 설정이 중복으로 존재하는 경우 yml 설정을 따라가는것으로 보임

1. 설정 클래스 파일
<pre><code class="java">
public class FeignClientConfiguration {
   @Bean //헤더달기
    public RequestInterceptor requestInterceptor(){
        return requestTemplate -> {
            requestTemplate.header("Authorization", "Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJoaGgxMjMiLCJyb2xlIjoiUk9MRV9VU0VSIiwiZXhwIjoxNzEyMTI1NDg5fQ.vXAG7uRaEgSW6fCgpSjZt02rY5kYS9RWYoAh6JAdmZE");
        };
       
    }
   
  @Bean //재시도 설정 
    Retryer.Default retryer(){
        return new Retryer.Default(1000L,TimeUnit.SECONDS.toMillis(3L),3);
        //return new Retryer.Default();
    }

  @Bean //로깅 내용 설정
    Logger.Level feignLoggerLevel(){
        return Logger.Level.FULL;
        /*
        NONE    : no logging(디폴트)
        BASIC   : request 메소드, request URL, response status, execution time
        HEADERS : request 헤더, response헤더, 기본적인 정보
        FULL    : request와 response의 헤더, 바디, 메타데이터 
        */
    }

  @Bean
    ErrorDecoder.Default errorDecoder(){
        return new ErrorDecoder.Default();
    }


}
</code></pre>




2. application.yml
<pre><code class="yaml">
spring:
  cloud:
    openfeign:
      client:
       config:
        default:  # fegin 클라이언트 이름 
         loggerLevel: FULL
         connectTimeout: 5000
         readTimeout: 5000
         retryer: kr.co.dominos.basketorder.global.config.FeignRetryer   # Retryer 클래스 만들어줘야됨
</code></pre>

로깅설정
feign 의 logging 설정의 경우, DEBUG level 에서만 작동함
<pre><code class="yaml">
logging:
  config: classpath:log4j2.xml
  level:
    com:
      querydsl:
        sql: DEBUG
    kr.co.dominos.basketorder.global.utils.FeignClientUtil: DEBUG
</code></pre>

어떤 경우에 Retry를 하는거얏


