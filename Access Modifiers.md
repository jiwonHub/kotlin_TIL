# Kotlin 접근 제어자

### public
기본 접근제어자. 모든 클래스에서 접근 가능하다.

### private 
해당 클래스 내부에서만 접근 가능하다.

### protected 
해당 클래스 내부와 하위 클래스에서 접근 가능하다.

### internal
같은 모듈 내에서만 접근 가능하다.

```kotlin
class MyClass {

    // public 멤버 변수
    var publicProperty: String = "Public"

    // private 멤버 변수
    private var privateProperty: String = "Private"

    // protected 멤버 변수
    protected var protectedProperty: String = "Protected"

    // internal 멤버 변수
    internal var internalProperty: String = "Internal"

    // public 메서드
    fun publicMethod() {
        println(publicProperty)
        println(privateProperty)
        println(protectedProperty)
        println(internalProperty)
    }

    // private 메서드
    private fun privateMethod() {
        println("private")
    }

    // protected 메서드
    protected fun protectedMethod() {
        println("protected")
    }

    // internal 메서드
    internal fun internalMethod() {
        println("internal")
    }
}

// 같은 파일 내 다른 클래스에서 접근
fun main() {
    val myClass = MyClass()

    // public 접근 가능
    println(myClass.publicProperty)
    myClass.publicMethod()

    // private 접근 불가
    // println(myClass.privateProperty) // 오류
    // myClass.privateMethod() // 오류

    // protected 접근 불가
    // println(myClass.protectedProperty) // 오류
    // myClass.protectedMethod() // 오류

    // internal 접근 가능 (같은 모듈 내에서)
    println(myClass.internalProperty)
    myClass.internalMethod()
}
```
