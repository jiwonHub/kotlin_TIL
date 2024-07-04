# Retrofit2?

Retrofit2는 Android및 Java에서 사용되는 Type-Safty HTTP 클라이언트 라이브러리다.

## Retrofit2의 특징

1. 타입 안전
    - Retrofit은 인터페이스와 어노테이션을 사용하여 HTTP API를 설명한다.
    - 이를 통해 컴파일 시간에 오류를 감지할 수 있다.
2. 데이터 변환기
    - Retrofit은 다양한 데이터 컨버터를 지원한다.
    - Gson, Moshi, Jackson 등의 라이브러리와 함께 사용하여 JSON 응답을 객체로 자동 변환할 수 있다.
3. 동기 및 비동기 요청
    - Retrofit은 동기 및 비동기 방식으로 네트워크 요청을 처리할 수 있다.
    - 비동기 요청은 Call 객체의 enqueue 메서드를 사용하여 처리된다.
4. 어노테이션 기반의 API 정의
    - Retrofit은 HTTP메서드(@GET, @POST, @PUT 등)와 요청 매개변수(@Query, @Path, @Body 등)를 지정하기 위한 다양한 어노테이션을 제공한다.
5. OkHttp통합
    - Retrofit은 내부적으로 OkHttp 라이브러리를 사용하여 네트워크 요청을 처리한다.
    - 이는 효율적인 연결 재사용, 요청/응답 인터셉터, 캐싱 등의 기능을 제공한다
6. 커스텀마이징
    - Retrofit은 인터셉터, 컨버터, 콜 어댑터 등을 사용하여 동작을 쉽게 커스터마이징 할 수 있다.

## Retrofit을 굳이 쓰는 이유?

### AsyncTasks vs Volley vs Retrofit

Instructure 팀은 안드로이드 앱의 속도 문제를 해결하기 위해 네트워킹 라이브러리를 변경하는 방안을 검토했다. 기존에는 AsyncTasks를 사용했지만, 여러 단점(방향 전환 지원 부족, 네트워크 호출 취소 불가, 병렬 API 호출 어려움 등)으로 인해 Volley와 Retrofit을 대안으로 고려했다.

### 기존 방식: AsyncTasks

- **문제점**:
    - 방향 전환 지원 부족
    - 네트워크 호출 취소 불가
    - 병렬 API 호출 어려움 (AsyncTasks는 직렬 방식으로 실행)
    - 여러 API 호출이 필요한 뷰에서 속도 저하
    

### 비교 평가

- **Volley**와 **Retrofit**은 모두 비동기 네트워크 호출을 지원하고 콜백 인터페이스를 제공하여 성공 및 실패 시 처리를 할 수 있다.
- **Volley**는 API 호출 시 전체 엔드포인트를 동적으로 지정하며, 기본적으로 JSONObject 또는 JSONArray를 반환한다.
- **Retrofit**은 기본 엔드포인트 URL을 설정하고 Java 어노테이션을 사용하여 정적 인터페이스를 작성한다. Retrofit은 GSON을 사용하여 JSON 파싱을 자동으로 수행한다.

### 성능 테스트

|  | One Discussion | Dashboard (7 requests) | 25 Discussions |
| --- | --- | --- | --- |
| AsyncTask | 941 ms | 4,539 ms | 13,957 ms |
| Volley | 560 ms | 2,202 ms | 4,275 ms |
| Retrofit | 312 ms | 889 ms | 1,059 ms |

### 성능 테스트 결과

벤치마크 테스트 결과, 두 라이브러리 모두 AsyncTasks보다 훨씬 빠르고, 성능 면에서 Retrofit이 가장 빠르다는 결론을 얻었다.

### 결론

- **Retrofit**을 선택한 이유는 속도가 매우 빠르고 기존 아키텍처와 잘 맞으며, 오류 처리, 캐싱, 페이징 등을 쉽게 처리할 수 있기 때문이다.
- Retrofit으로의 전환 작업은 변수 이름 변경, 인터페이스 작성, AsyncTasks 제거 등의 작업을 포함하여 비교적 간단하게 진행된다.

## Retrofit2 기본 사용법

### 1. Retrofit 인스턴스 생성

Retrofit을 사용하기 위해서는 먼저 인스턴스를 생성해야한다. Retrofit 인스턴스는 내부적으로 
OkHttp를 사용한다.

OkHttp 클라이언트는 커넥션 풀, 캐시, Request/Response/Intercepter 등을 관리하는데, 이런 리소스들을 효율적으로 관리하기 위해서는 초기화 되지 않고 재사용되어야한다.

```kotlin
val retrofit = Retrofit.Builder()
		.baseUrl("http://api.example.com/")
		.addConverterFactory(GsonConverterFactory.create())
		.build()
```

### 2. API 인터페이스 정의

API 호출을 정의하기 위해 인터페이스를 사용한다.

각 메서드에는 HTTP 동작을 나타내는 어노테이션아 포함되어야 한다.

```kotlin
interface ApiService {
		@GET("users/{user}")
		fun getUser(@Path("user") user: String): Call<User>
}
```

### 3. 어노테이션을 사용한 HTTP 메서드와 요청 매개변수 저장

Retrofit에서는 다양한 어노테이션을 사용하여 요청의 종류와 매개변수를 저장할 수 있다.

이를 통해 개발자는 복잡한 코드 없이 API의 endpoint와 파라미터를 정의할 수 있다.

- **HTTP 메서드**:
    - `@GET` - GET 요청, 주로 데이터를 검색할 때 사용한다.
    - `@POST` - POST 요청, 주로 데이터를 서버에 전송할 때 사용한다.
    - `@PUT` - PUT 요청, 서버의 데이터를 갱신할 때 사용한다.
    - `@DELETE` - DELETE 요청, 서버의 데이터를 삭제할 때 사용한다.
- **URL 매개변수**:
    - `@Path` - URL 일부를 동적으로 변환하는데 사용된다.
        - 예를 들어, `@GET("users/{id}")`와 같이 경로에 변수를 지정하고, 메서드 파라미터에서 `@Path("id") int id`로 값을 전달할 수 있다.
- **쿼리 매개변수**:
    - `@Query` - URL에 쿼리 파라미터를 추가하는데 사용된다.
        - 예: `@Query("sort") String order`는 `sort=orderValue` 형태로 URL에 추가된다.
    - `@QueryMap` - 여러 쿼리 파라미터를 한 번에 전달하기 위해 사용된다. Map 객체를 사용해 key-value 쌍의 파라미터를 전달 가능하다.
- **헤더**:
    - `@Headers` - 요청에 정적 헤더를 추가한다. 여러 헤더를 배열 형태로 전달할 수 있다.
    - `@Header` - 요청에 동적 헤더를 추가한다. 메서드의 파라미터로 전달된 값을 헤더에 추가한다.
- **요청 본문**:
    - `@Body` - POST나 PUT 요청에서 요청 본문을 전송할 때 사용한다. 주로 객체를 JSON 형식으로 변환한 뒤 서버에 전송하는 데 사용한다.

### 4. 응답 처리 및 데이터 변환

4-1) Call을 사용해 동기 및 비동기 방식으로 응답 처리하기

- **동기 호출(Synchronous call)**:
    - `Call.execute()`를 사용하여 동기적으로 API 호출.
    - 이 방식은 메서드가 호출되었을 때 해당 메서드의 실행이 완료될 때까지 호출한 쪽에서 대기하는 방식이다.
    - 즉, 호출된 메서드가 끝날 때까지 코드는 다음으로 안넘어간다.
    - 때문에 UI스레드(= Main 스레드)에서 호출하면 안되고 백그라운드 스레드(I/O를 권장)에서 실행되어야 한다.
- **비동기 호출(Asynchronous call)**:
    - `Call.enqueue()` 메서드를 사용하여 비동기적으로 API를 호출한다.
    - 이 방식은 메서드를 호출하고 바로 다음 코드로 넘어간다. API 호출 결과는 Callback 메커니즘을 통해 전달받는다.

```kotlin
interface ApiService {
    @GET("users/{id}")
    fun getUser(@Path("id") userId: Int): Call<User>
```

이렇게 정의된 인터페이스를 사용하여 API를 호출할 때는 다음과 같이 처리한다.

```kotlin
val apiService = retrofit.create(ApiService::class.java)
val call = apiService.getUser(1)
call.enqueue(object : Callback<User> {
    override fun onResponse(call: Call<User>, response: Response<User>) {
        if (response.isSuccessful) {
            val user = response.body()
            // TODO: 사용자 정보 처리
        } else {
            // TODO: 에러 처리
        }
    }

    override fun onFailure(call: Call<User>, t: Throwable) {
        // TODO: 네트워크 오류 또는 요청 실패 처리
    }
})
```

`Call<T>`를 사용하면 콜백 메커니즘을 통해 비동기적으로 API응답을 처리해야 한다.

### 4-2) Coroutine을 사용해 응답 처리하기

코루틴은 동시성을 쉽게 처리하기 위한 기능이다.

Retrofit과 코루틴을 함께 사용하면 동기적인 방식의 코드임에도 비동기적인 효과를 얻을 수 있다.

코루틴을 Retrofit과 사용할 때의 장점은 다음과 같다:

- **간결한 코드**: Callback을 사용하지 않고 직관적인 순차적 코드로 비동기 로직을 작성할 수 있다. 때문에 가독성이 좋다.
- **비블로킹**: 코루틴은 비블로킹이기 때문에, UI 스레드에서도 네트워크 요청과 같은 시간이 오래 소요되는 작업을 수행할 수 있다.
- **코루틴 빌더**: `launch`, `async`, `withContext` 등의 코루틴 빌더를 사용하여 다양한 동시성 패턴을 적용할 수 있다.
- **예외 처리**: `try-catch`를 사용하여 예외 처리를 쉽게 가능하다.

Retrofit 2.6.0 버전 이후로 코루틴을 직접 지원하기 때문에, API 인터페이스 메서드에서 `suspend` 키워드로 코루틴 함수를 정의 가능하다.

```kotlin
interface ApiService {
    @GET("users/{id}")
    suspend fun getUser(@Path("id") userId: Int): Response<User>
}
```

위처럼 Thread가 포함된 복잡한 코드나 별도의 동기/비동기 처리 없이 간결하게 API 호출과 응답을 처리할 수 있다.

저 Response 타입은 호출할 함수로<T>에 감싸진 User를 API호출 결과로 반환되는 데이터를 포함하게 하여 사용하는 데이터 클래스이다.

위 코드의 경우, getUser는 User 클래스의 인스턴스를 담고 있는 Response 객체를 반환한다.

### **4-3) Converter를 사용하여 JSON 응답을 객체로 변환하는 방법**

위의 기타 API 결과처럼, 대부분의 API 결과는 JSON 응답으로 내려온다.

그럼 왜 JSON 응답을 객체로 변환해서 사용해야 할까?

이유는 크게 세 가지로 요약할 수 있다.

1. **데이터 사용의 용이성**
    - JSON은 텍스트 기반의 데이터 포맷으로, 프로그래밍 언어와 독립적이다. 그래서 JSON 형식의 데이터를 직접 파싱하기 위해서는 복잡하고 시간이 많이 소요되는 비효율적인 일이다.
    - 객체로 변환하면, 개발할 때 쉽게 데이터에 접근하고 필요한 정보를 추출하거나 수정할 수 있게 된다.
    - 예를 들어 Kotlin에서는 클래스의 필드와 메서드를 사용해서 데이터를 쉽게 다룰 수 있다.
2. **타입 안정성과 오류 방지**
    - 객체 변환을 통해 데이터의 타입을 명확히 할 수 있고, 이는 잘못된 타입의 데이터 사용으로 발생하는 오류를 방지할 수 있다.
3. **유지보수가 쉽다**
    - 객체로 변환하면, 데이터 구조가 클래스나 구조체의 형태로 명확하게 정의된다. 이는 코드를 이해하기 쉽게 해주어 협업이 쉬워지며, 별도의 코드 설명 없이 데이터 구조를 빠르게 파악할 수 있도록 한다.
    - 또한, 데이터 구조에 변화가 있으면, 해당 객체의 클래스나 구조체만 수정하면 되므로 편하다.

그럼 어떻게 JSON을 객체로 변환할 수 있는가?

보통 안드로이드 개발 시에는, Gson Converter와 Moshi Converter를 사용한다.

각 라이브러리에 대한 설명은 다른 글에서 다루도록 하고, 이번 글에서는 Converter를 추가하는 방법만 파악하겠다.

여기서는 가장 많이 쓰이는 Gson을 예제로 설명한다.

우선 Retrofit 인스턴스에 GsonConverterFactory를 추가해야한다.

```kotlin
val retrofit = Retrofit.Builder()
    .baseUrl("<https://api.example.com/>")
    .build()  
```

이게 가장 기본적인 Retrofit 빌더의 모습이다.

여기서 .addConverterFactory(GsonConverterFactory.create()) 를 추가한다.

```kotlin
val retrofit = Retrofit.Builder()
    .baseUrl("<https://api.example.com/>")
    .addConverterFactory(GsonConverterFactory.create())
    .build()
```

다음으로 JSON 응답을 받은 데이터 클래스를 정의하고, API 인터페이스 메소드의 반환 타입을 Call로 설정하면, Retrofit과 Gson이 자동으로 JSON응답을 지정한 데이터클래스 객체로 변환한다.

```kotlin
interface BookSearchApi {
    @Headers("Authorization: KakaoAK \\$API_KEY")
    @GET("search/book")
    suspend fun searchBooks(
        @Query("query") query: String,
        @Query("sort") sort: String,
        @Query("page") page: Int,
        @Query("size") size: Int,
    ): Call<Book>
}
```
