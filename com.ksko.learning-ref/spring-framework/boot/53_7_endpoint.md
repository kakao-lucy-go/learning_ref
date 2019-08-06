53.7 커스텀 엔드포인트 확장
====
   
! 엔드포인트란 앱을 **모니터링** 하고 **상호작용** 할 수 있도록 해주는 것. 모니터링 뿐 아니라 트래픽, DB의 상태도 확인할 수 있다.  스프링 부트는 여러 빌트인된 엔드포인트들이 있다. 이 기능을 사용하려면 actuator 프로젝트를 추가해주고 속성을 true로 변경해주면 된다.
인증이 기본이긴 한데 없이 사용할 수도 있다.

! @Endpoint 어노테이션으로 엔드포인트를 지정해주고(예제에서는 클래스 위에 선언하던데..!!) 지정된 url로 요청하면 노출된다. [엔드포인트 사용 예제](https://howtodoinjava.com/spring-boot/actuator-endpoints-example/).

! JMX는 앱 모니터링 표준. 
![jmx_architecture](https://github.com/mychum1/learning_ref/blob/master/com.ksko.learning-ref/imgs/jmx.png?raw=true)  
자바 앱이랑 jmx 모니터링 툴이랑 연결해서 데이터를 본다고 하면 되고, 실행시킬 때 옵션을 사용해주면 된다. java -jar com.sun.management.jmxremote=true 이런식으로!

@Endpoint 어노테이트된 @Bean을 추가한다면, @ReadOperation, @WriteOperation, @DeleteOperation과 같이 어노테이트된 메소드들은 자동적으로 JMX와 웹앱에서 HTTP를 통해 노출된다. 엔드포인트들은 Jersey, 스프링 MVC나 Spring WebFlux를 사용해서 노출될 수 있다.   
   
@WebEndpoint, @JmxEndpoint를 사용해서 특정 기술 엔드포인트를 쓸 수 있다. 이 엔드포인트들은 해당 기술로 제한된다. 예를 들어서, @WebEndpoint는 JMX가 아니라 오직 HTTP를 통해서 노출된다.  
  
@EndpointWebExtension과 @EndpointJmxExtension을 사용해서 특정 기술 엔드포인트를 쓸 수 있다. 이 어노테이션들은 존재하는 엔드포인트를 보완하는 특정 기술 동작을 제공하게 한다.   
  
웹 프레임 워크 특정 기능에 액세스해야하는 경우 JMX를 통해 사용할 수 없거나 다른 웹 프레임 워크를 사용할 때 Servlet 또는 Spring @Controller 및 @RestController 끝점을 구현할 수 있다.  
  

53.7.1 인풋 받기
----
  
엔드 포인트에서의 조작은 매개 변수를 통해 입력을 받다. 웹을 통해 노출되면 이러한 매개 변수의 값은 URL의 쿼리 매개 변수 및 JSON 요청 본문에서 가져온다. JMX를 통해 노출되면 매개 변수가 MBean의 작업 매개 변수에 매핑된다. 매개 변수는 기본적으로 필요하다. @org.springframework.lang.Nullable 과 어노테이트해서 옵션화할 수 있다.  
JSON 요청 바디에서각 루트 프로퍼티는 엔드포인트의 파라미터로 매핑될 수 있다.
  
<pre><code>
{ 
	"name" : "test" , 
	"counter" : 42 
}
</code></pre>
  
String name과 int counter 파라미터를 사용하는 쓰기 동작을 호출하는데 사용할 수 있다.  
> 엔드 포인트는 기술에 독립적이므로 메소드 시그니처에 단순 유형 만 지정할 수 있다. 특히 name 및 counter속성을 정의하는 사용자 정의 유형이 있는 단일 매개 변수 선언하는 것은 지원되지 않는다.
! 메소드 시그니처란? (Method Signature. 메소드 생성 규칙들 중에서 메소드의 이름, 파라미터를 메소드의 시그니처라고한다. 따라서 위에서 하는 말은 name이랑 counter를 갖는 오브젝트를 하나 만들어서 그걸 인자로 받는 커스텀한 경우는 지원되지 않는다는 말!!
> 입력을 조작 메소드의 매개 변수에 맵핑하려면 엔드 포인트를 구현하는 Java 코드를 -parameters로 컴파일해야하며 엔드 포인트를 구현하는 Kotlin 코드는 -java-parameters로 컴파일해야한다. 이것은 Spring Boot의 Gradle 플러그인을 사용하거나 Maven과 spring-boot-starter-parent를 사용하는 경우 자동으로 발생한다.
  
### 입력 유형 변환
  
엔드포인트 동작 메소드에 전달된 매개 변수는 필요한 경우 자동으로 필수 유형으로 변환된다. 동작 메소드를 호출하기 전에, JMX 또는 HTTP 요청을 통해 수신된 입력은 **ApplicationConversionService 인스턴스를 사용해서 요구되는 타입으로 변환된다.**

53.7.2 커스텀 웹 엔드포인트
----
   
@Endpoint, @WebEndpoint, @EndpointWebExtension 동작은 자동적으로 Jersey, 스프링 MVC, 스프링 Webflux를 사용해서 HTTP 로 노출된다.

!Jersey란? Resful 웹서비스용 자바 API. JSR 311에서 구현됨.

### 웹 엔드포인트 요청 Predicates(동작들)
   
요청 Predicates는 자동적으로 웹에 노출되는 엔드포인트에 각 동작을 일반화한다.

???

### Path
   
prediacate의 path는 웹에 노출된 엔드포인트의 기본 path와 엔드포인트의 ID에 의해 결정된다. 기본 path는 **/actuator**이다. 예를 들어서 sessions라는 ID를 가진 엔드포인트는 /actuator/sessions을 사용할 것이다.  
@Selector를 사용하여 하나 이상의 조작 메소드 매개 변수에 주석을 달아 경로를 추가로 사용자 정의 할 수 있다. 이러한 매개 변수는 경로 변수로 경로 조건 자에 추가됩니다. 변수의 값은 엔드 포인트 조작이 호출 될 때 조작 메소드에 전달된다.
   
### HTTP  메소드
   
predicate 의 HTTP메소드는 동작 타입에 의해 결정된다. 아래 테이블을 보자.  

| Operation       | HTTP method |
| :--------------- | :---------- |
| @ReadOperation   | GET         |
| @WriteOperation  | POST        |
| @DeleteOperation | DELETE      |
  
### Consumes
  
@WriteOperation(HTTP POST) 를 request body 에 사용하면 predicate의 consumes 구문은 application/vnd.spring-boot.actuator.v2+json, application/json이다. consume구문의 다른 모든 동작은 비어있다.  
  
### Produces

predicate의 produces 구문은 @DeleteOperation, @ReadOperation, @WriteOperation 어노테이션의 produces 속성에 의해 결정될 수 있다. 그 옵션은 선택적이다. 사용하지 않는다면, produces 구문은 자동적으로 결정된다.  
  
오퍼레이션 메소드가 void 또는 Void를 리턴하면 produce 절이 비어 있습니다. 조작 메소드가 org.springframework.core.io.Resource를 리턴하면 produce 절은 application / octet-stream입니다. 다른 모든 작업의 ​​경우 produce 절은 application / vnd.spring-boot.actuator.v2 + json, application / json이다.  
  
### 웹 엔드포인트 응답 스테이터스
  
엔드포인트 동작을 위한 기본 응답 스테이터스는 타입(read, write, delete)과 리턴 값에 의존한다. @ReadOperation 은 값을 리턴하면 응답코드를 200 OK로 응답한다. 값은 반환하지 않으면 404(Not found)를 응답할 것이다. @WriteOpearion이나 @DeleteOperation이 값을 반환하면 200 OK를 반환할 것이다. 값을 반환하지 않으면 204 (No Content)를 반환할 것이다.  
동작이 요구된 파라미터 없이 호출하거나 요구되는 타입으로 컨버팅될 수 없다면 메소드는 불리지 않을 것이고 응답 스테이터스도 400(bad request) 일 것이다.  
  
### 웹 엔드포인트 범위 요청

HTTP 범위 요청은 HTTP 리소스의 요청 부분에 사용될 수 있다. 스프링 MVC나 스프링 웹 플럭스를 사용할 때 **org.springframework.core.io.Resource** 를 리턴하는 동작은 자동적으로 범위 요청을 지원한다.  
>범위 요청은 Jersey를 사용할 때는 지원하지 않는다.  
  
### 웹 엔드포인트 시큐리티  
웹 엔드 포인트 또는 웹 특정 엔드 포인트 확장에 대한 조작은 현재 java.security.Principal 또는 org.springframework.boot.actuate.endpoint.SecurityContext를 메소드 매개 변수로 수신 할 수 있다. 전자는 일반적으로 @Nullable과 함께 사용되어 인증 된 사용자와 인증되지 않은 사용자에 대해 다른 동작을 제공한다. 후자는 일반적으로 isUserInRole (String) 메소드를 사용하여 권한 부여 검사를 수행하는 데 사용다.  
  
53.7.3 서블릿 엔드포인트
----
  
공급자 <EndpointServlet>도 구현하는 @ServletEndpoint로 주석이 달린 클래스를 구현하여 서블릿을 엔드 포인트로 노출 할 수 있다. 서블릿 엔드 포인트는 서블릿 컨테이너와의 긴밀한 통합을 제공하지만 이식성을 희생한다. 기존 서블릿을 엔드 포인트로 노출하는 데 사용한다. 새로운 엔드포인트의 경우 가능한 @Endpoint 및 @WebEndpoint 어노테이션을 선호해야 한다.  
  
53.7.4 컨트롤러 엔드포인트
----
  
@ControllerEndpoint와 @RestControllerEndpoint는 Spring MVC 또는 Spring WebFlux에서만 노출되는 엔드 포인트를 구현하는 데 사용할 수 있다. 메소드는 @RequestMapping 및 @GetMapping과 같은 Spring MVC 및 Spring WebFlux의 표준 어노테이션을 사용하여 맵핑되며 엔드 포인트의 ID는 경로의 접두부로 사용된다. 컨트롤러 엔드 포인트는 Spring의 웹 프레임 워크와 보다 긴밀한 통합을 제공하지만 이식성을 희생한다. @Endpoint 및 @WebEndpoint 어노테이션은 가능할 때마다 선호되어야 한다.  
  
53.8 health 정보
====
  
상태 정보를 사용하여 실행중인 응용 프로그램의 상태를 확인할 수 있다. 프로덕션 시스템이 다운 될 때 경고하기 위해 소프트웨어를 모니터링하여 자주 사용된다. health 엔드포인트에 의해 노출되는 정보는 다음 값 중 하나로 구성 할 수있는 management.endpoint.health.show-details 속성에 따라 다르다.  
| Name            | Description                                                                   |
| :-------------- | :---------------------------------------------------------------------------- |
| never           | 디테일은 절대 보여지지 않는다.                                                             |
| when-authroized | 디테일은 허가된 사용자에게만 보여준다. 인증 역할은 management.endpiont.health.roles를 사용해서 설정할 수 있다. |
| always          | 디테일은 모든 유저에게 보여진다.                                                            |
   
기본 값은 **never**다. 사용자는 하나 이상의 엔드 포인트 역할에있을 때 권한이 부여 된 것으로 간주된다. 엔드 포인트에 구성된 역할이 없는 경우 (기본값) 모든 인증 된 사용자는 권한이 부여 된 것으로 간주된다. management.endpoint.health.roles 특성을 사용하여 역할을 구성 할 수 있다.  
  
> 앱이 secure 하고있고 always를 사용하길 원할수도 있는데, securiy config에 권한이 있는 유저와 없는 유저 모두에게 health 엔드포인트에 접근하는 것을 허가해주어야만 한다.  
  
health 정보는 **HealthIndicatorResigtry** 에서 수집된다. (기본적으로 모든 HealthIndicator 인스턴스는 ApplicationContext에 의해 정의된다.). 스프링 부트는 자동 config된 **HealthIndicators**들을 포함하고, 커스텀할 수도 있다. 기본적으로 최종 시스템 상태는 상태 목록에 따라 각 HealthIndicator에서 상태를 정렬하는 HealthAggregator에 의해 파생된다. 정렬 된 목록의 첫 번째 상태는 전체 상태로 사용된다. HealthIndicator가 HealthAggregator에 알려진 상태를 리턴하지 않으면 UNKNOWN 상태가 사용된다.
  

> HealthIndicatorRegistry는 런타임에 Health 표시기를 등록 및 등록 해제하는 데 사용될 수 있다.

53.8.1 자동 설정 HealthIndicators
----

**HealthIndicators**는 스프링 부트에 의해 자동 설정될 수 있다.

| Name                         | Description               |
| :--------------------------- | :------------------------ |
| CassandraHealthIndicator     | 카산드라 DB를 체크한다.            |
| CouchbaseHealthIndicator     | couchbase 클러스터를 체크한다.     |
| DiskSpaceHealthIndicator     | 디스크 공간을 체크한다.             |
| DataSourceHealthIndicator    | **DataSource** 커넥션을 체크한다. |
| ElasticsearchHealthIndicator | 엘라스틱서치 클러스터를 체크한다.        |
| InfluxDbHealthIndicator      | InfluxDB를 체크한다.           |
| JmsHealthIndicator           | JMS broker를 체크한다.         |
| MailHealthIndicator          | mail server를 체크한다.        |
| MongoHealthIndicator         | 몽고 DB를 체크한다.              |
| Neo4jHealthIndicator         | Neo4j를 체크한다.              |
| RabbitHealthIndicator        | Rabbit 서버를 체크한다.          |
| RedisHealthIndicator         | Redis를 체크한다.              |
| SolrHealthIndicator          | Solr 서버를 체크한다.            |

> management.health.defaults.enabled 프로퍼티를 세팅해서 안보이게 할 수 있다.

53.8.2 커스텀 HealthIndicator 작성하기
----

사용자 정의 health 정보를 제공하기 위해 HealthIndicator 인터페이스를 구현하는 Spring 빈을 등록 할 수 있다. health () 메서드의 구현을 제공하고 Health 응답을 반환해야 한다. Health 응답에는 상태가 포함되어야하며 표시 할 추가 세부 사항을 선택적으로 포함 할 수 있다. 다음 코드는 샘플 HealthIndicator 구현을 보여준다.

<pre><code>
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class MyHealthIndicator implements HealthIndicator {

	@Override
	public Health health() {
		int errorCode = check(); // perform some specific health check
		if (errorCode != 0) {
			return Health.down().withDetail("Error Code", errorCode).build();
		}
		return Health.up().build();
	}

}
</code></pre>

> 주어진 HealthIndicator의 식별자는 존재하는 경우 HealthIndicator 접미어가 없는 Bean의 이름이다. 앞의 예에서 health 정보는 **my**라는 항목에서 사용할 수 있습니다.

Spring Boot의 사전 정의 된 상태 유형 외에도 Health는 새로운 시스템 상태를 나타내는 사용자 정의 상태를 반환 할 수도 있다. 이러한 경우 HealthAggregator 인터페이스의 사용자 정의 구현도 제공하거나 management.health.status.order 구성 특성을 사용하여 기본 구현을 구성해야 한다.

예를 들어 HealthIndicator 구현 중 하나에서 코드가 FATAL 인 새 Status가 사용되고 있다고 가정한다. 심각도 순서를 구성하려면 응용 프로그램 속성에 다음 속성을 추가하십시오.

<pre><code>
management.health.status.order=FATAL, DOWN, OUT_OF_SERVICE, UNKNOWN, UP
</code></pre>

응답의 HTTP 상태 코드는 전반적인 상태를 반영한다 (예 : UP은 200으로 매핑되고 OUT_OF_SERVICE 및 DOWN은 503으로 매핑 됨). 또한 HTTP를 통해 상태 관리 끝점에 액세스하는 경우 사용자 지정 상태 매핑을 등록 할 수도 있다. 예를 들어 다음 속성은 FATAL을 503 (서비스를 사용할 수 없음)으로 매핑한다.

<pre><code>
management.health.status.http-mapping.FATAL=503
</code></pre>

> 더 컨트롤이 필요하면 **HealthStatusHttpMapper** 빈을 정의할 수 있다.

다음 표에서는 기본 제공 상태에 대한 기본 상태 매핑을 보여준다.

| Status         | Mapping                       |
| :------------- | :---------------------------- |
| DOWN           | SERVICE_UNAVAILABLE(503)      |
| OUT_OF_SERVICE | SERVICE_UNABLABLE(503)        |
| UP             | 기본 매핑 값이 없어서 http status는 200 |
| UNKNOWN        | 기본 매핑 값이 없어서 http status는 200 |

53.8.3 반응성 health indicators
----

ReactiveHealthIndicator는 Spring WebFlux를 사용하는 것과 같이 반응이있는 응용 프로그램의 경우 응용 프로그램 상태를 가져 오는 비 차단 계약을 제공한다. 전통적인 HealthIndicator와 유사하게, 건강 정보는 ReactiveHealthIndicatorRegistry의 컨텐츠 (기본적으로 ApplicationContext에 정의 된 모든 HealthIndicator 및 ReactiveHealthIndicator 인스턴스)에서 수집된다. 반응 API에 대해 확인하지 않는 Regular HealthIndicator는 탄력적 스케줄러에서 실행된다.

> 반응 형 응용 프로그램에서 ReactiveHealthIndicatorRegistry를 사용하여 런타임시 상태 표시기를 등록 및 등록 취소 할 수 있다.

반응 형 API에서 사용자 정의 상태 정보를 제공하기 위해 ReactiveHealthIndicator 인터페이스를 구현하는 Spring Bean을 등록 할 수 있다. 다음 코드는 샘플 ReactiveHealthIndicator 구현을 보여준다.

<pre><code>
@Component
public class MyReactiveHealthIndicator implements ReactiveHealthIndicator {

	@Override
	public Mono<Health> health() {
		return doHealthCheck() //perform some specific health check that returns a Mono<Health>
			.onErrorResume(ex -> Mono.just(new Health.Builder().down(ex).build())));
	}

}
</code></pre>

> 오류를 자동으로 처리하려면 AbstractReactiveHealthIndicator에서 확장하는 것이 좋다.

53.8.4 자동 설정된 ReactiveHealthIndicators
----

다음 ReactiveHealthIndicators는 적절한 경우 Spring Boot에 의해 자동 구성된다.

| Name                             | Description       |
| :------------------------------- | :---------------- |
| CassandraReactiveHealthIndicator | 카산드라 db 체크        |
| CouchbaseReactiveHealthIndicator | Couchbase 클러스터 체크 |
| MongoReactiveHealthIndicator     | 몽고db 체크           |
| RedisReactiveHealthIndicator     | Redis 서버 체크       |

> 필요한 경우 반응성 표시기가 일반 표시기를 대체한다. 또한 명시 적으로 처리되지 않은 HealthIndicator는 자동으로 랩핑된다.
