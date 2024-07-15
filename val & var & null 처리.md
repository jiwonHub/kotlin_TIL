# val & var & null 처리

## val & var의 차이

코틀린은 자바와 달리 변수선언시 val과 var를 사용하며 타입 선언 없이 값을 할당할 수 있다.

1. val
    - Immutable(불변) 변수를 선언할 때 사용한다.
    - 한 번 초기화된 이후에는 값을 변경할 수 없다.
    - Java에서의 final 키워드와 비슷하다.
2. var
    - Mutable(불변) 변수를 선언할 때 사용한다.
    - 초기화된 이후에도 값을 변경할 수 있다.

## 요약

val과 var는 변수의 시작을 알리면서 변수가 불변인지 가변인지를 나타낸다.

공통적으로 초기화 시 값을 할당하지 않는다면 반드시 type을 명시해야 한다.

하지만 컴파일러가 변수 타입을 추론할 수 있는 경우에는 타입을 생략해도 된다.

## null 처리 방법

1. Nullable 타입
    
    코틀린에는 변수 타입에 ‘?’를 붙여서 nullable 타입을 명시할 수 있다. 이는 해당 변수가 null값을 가질 수 있음을 나타낸다.
    
    ```kotlin
    var nullableString: String? = "Hello"
    nullableString = null // null 허용됨
    ```
    
2. 안전한 호출 연산자 (Safe Call Operator)
    
    ‘?.’ 연산자를 사용하여 null 체크를 간단하게 할 수 있다. 만약 객체가 null이라면, 해당 호출은 null을 반환한다.
    
    ```kotlin
    val length: Int? = nullableString?.length
    ```
    
3. 엘비스 연산자 (Elvis Operator)
    
    ‘?:’ 연산자를 사용하여 null일 경우 대체 값을 제공할 수 있다.
    
    ```kotlin
    val length: Int = nullableString?.length ?: 0
    ```
    
4. 널 가능성 확인 연산자 (Null-assertion Operator)
    
    ‘!!’ 연산자를 사용하여 변수나 값이 null이 아님을 보장할 수 있다. 그러나 만약 null값이 있을 경우, ‘NullPointerException’이 발생한다.
    
    ```kotlin
    val length: Int = nullableString!!.length
    ```
    
5. 안전한 형 변환 (Safe Cast Operator)
    
    ‘as?’ 연산자를 사용하여 안전하게 형 변환을 할 수 있다. 만약 변환이 불가능하다면 null을 반환한다.
    
    ```kotlin
    val nullableInt: Int? = nullableString as? Int
    ```
    
6. let 함수
    
    ‘let’함수를 사용하여 null이 아닌 경우에만 특정 블록을 실행할 수 있다.
    
    ```kotlin
    nullableString?.let {
        println(it.length)
    }
    ```
