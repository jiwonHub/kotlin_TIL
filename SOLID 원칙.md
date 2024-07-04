# SOLID 원칙 이란

객체 지향 프로그래밍에서 소프트웨어 설계 원칙을 지칭하는 두문자어로, 다섯 가지 핵심 원칙을 포함하고 있다.

## 1. SRP (Single Responsibility Principle) 단일 책임 원칙

- 클래스는 단 하나의 책임만 가져야 한다. 이는 클래스가 하나의 책임을 수행하는데 집중되어야하며, 변경 사유가 하나뿐이여야 함을 의미한다.
- 단일 책임 원칙을 잘 수행하게 되면 무엇보다도 책임 영역이 확실해지기 때문에 한 책임의 변경에서 다른 책임의 변경으로의 연쇄작용에서 자유로워진다.
</br>

### 적용 방법

1. Extract Class를 통해 혼재된 각 책임을 각각의 개별 클래스로 분할하여 클래스 당 하나의 책임만을 맡도록 한다.
2. 책임만 분리하는 것이 아니라 분리된 두 클래스간의 관계의 복잡도를 줄이도록 설계하는 것이 포인트.
3. 산탄총 수술 : 산발적으로 여러 곳에 분포된 책임들을 한 곳에 모으면서 설계를 깨끗하게 한다. 즉, 응집성을 높이는 작업이다.

### SRP 적용 전 코드

```kotlin
class User(val username: String, val email: String) {

    fun getUsername(): String {
        return username
    }

    fun getEmail(): String {
        return email
    }

    fun saveToDatabase() {
        // 데이터베이스에 사용자 정보를 저장하는 로직
        println("Saving $username to the database.")
    }

    fun sendEmail(message: String) {
        // 이메일을 보내는 로직
        println("Sending email to $email: $message")
    }
}

```

- 위 코드는 사용자 정보를 저장 & 출력하는 User클래스이다. User클래스 내부에 이름과 이메일을 가져오는 메서드와 저장 & 출력하는 메서드도 포함되어있다.

### SRP 적용 코드

```kotlin
class User(val username: String, val email: String) {
    fun getUsername(): String {
        return username
    }

    fun getEmail(): String {
        return email
    }
}

// UserRepository 클래스: 사용자 정보를 데이터베이스에 저장하는 책임을 가짐
class UserRepository {
    fun saveToDatabase(user: User) {
        // 데이터베이스에 사용자 정보를 저장하는 로직
        println("Saving ${user.getUsername()} to the database.")
    }
}

// EmailService 클래스: 이메일을 보내는 책임을 가짐
class EmailService {
    fun sendEmail(user: User, message: String) {
        // 이메일을 보내는 로직
        println("Sending email to ${user.getEmail()}: $message")
    }
}

```

- 수정된 User클래스는 사용자 정보를 관리하는데 집중하고, UserRepository와 EmailService로 메서드를 분리함으로써 각 클래스들이 단일 책임 원칙을 따르게 되었다.

### 적용 이슈

- 클래스는 자신의 이름이 나타내는 일을 해야한다. 올바른 클래스 이름은 해당 클래스의 책임을 나타낼 수 있는 가장 좋은 방법이다.
- 각 클래스는 하나의 개념을 나타내어야 한다.
- 무조건 책임을 분리한다고 SRP가 적용되는 것이 아니다. 각 개체 간의 응집력이 있다면 병합이 순 작용의 수단이 되고 결합력이 있다면 분리가 순 작용의 수단이 된다. 중요한 것은 클래스의 단일 책임이다.

## 2. OCP(Open Close Principle) 개방폐쇄의 원칙

- 소프트웨어 개체(클래스, 모듈, 함수 등)는 확장에는 열려 있어야 하고, 변경에는 닫혀 있어야 한다. 즉, 새로운 기능을 추가할 때 기존 코드를 수정하지 않고도 기능을 확장할 수 있어야 한다는 원칙이다.
- 변경을 위한 비용은 가능한 줄이고 확장을 위한 비용은 가능한 극대화 해야 한다는 의미로 통한다.

### 적용 방법

1. 변경될 것과 변하지 않을 것을 엄격히 구분한다.
2. 이 두 모듈이 만나는 지점에 인터페이스를 정의한다.
3. 구현에 의존하기보다 정의한 인터페이스에 의존하도록 코드를 짠다.

### OCP 적용 전 코드

```kotlin
class Employee(val name: String, val type: String, val salary: Double)

class SalaryCalculator {
    fun calculateSalary(employee: Employee): Double {
        return when (employee.type) {
            "Developer" -> employee.salary * 1.2
            "Manager" -> employee.salary * 1.5
            else -> employee.salary
        }
    }
}

fun main() {
    val employees = listOf(
        Employee("John", "Developer", 1000.0),
        Employee("Jane", "Manager", 1500.0)
    )
    
    val calculator = SalaryCalculator()
    for (employee in employees) {
        println("Salary of ${employee.name} is ${calculator.calculateSalary(employee)}")
    }
}
```

- SalaryCalculator 클래스는 각 직원 유형에 따라 급여를 계산하는 클래스이다. 겉보기에 돌아가는 데는 문제가 크게 없어 보이지만, 여기서 다른 종류의 직원 유형이 추가 된다면 SalaryCalculator 클래스를 일일이 수정해야하는 번거로움이 발생한다.

### OCP 적용 코드

```kotlin
abstract class Employee(val name: String, val salary: Double) {
    abstract fun calculateSalary(): Double
}

class Developer(name: String, salary: Double) : Employee(name, salary) {
    override fun calculateSalary(): Double {
        return salary * 1.2
    }
}

class Manager(name: String, salary: Double) : Employee(name, salary) {
    override fun calculateSalary(): Double {
        return salary * 1.5
    }
}

fun main() {
    val employees = listOf(
        Developer("John", 1000.0),
        Manager("Jane", 1500.0)
    )
    
    for (employee in employees) {
        println("Salary of ${employee.name} is ${employee.calculateSalary()}")
    }
}
```

- 수정된 코드는 Employee 클래스를 데이터 클래스에서 추상 클래스로 정의하고, 각 직원 유형의 정보를 별도의 클래스로 확장했다. 이렇게 되면 다른 종류의 직원 유형이 추가 되더라도 Employee 클래스가 수정될일은 전혀 없다. 클래스를 하나 추가해서 Employee 를 상속받으면 그만이다.

### 적용이슈

- 인터페이스는 가능하면 변경되어서는  안 된다.
- 여러 경우의 수에 대한 고려와 예측이 요구된다.

## 3. LSP(The Liskov Substitution) 리스코프 치환의 원칙

- 쉽게 말해 프로그램의 객체가 부모 클래스의 인스턴스 대신 자식 클래스의 인스턴스로 교체되어도 프로그램의 정확성이 유지되어야 한다는 원칙이다. 아래 세 가지 핵심 조건을 만족해야 한다.
- 서브 클래스는 기반 클래스의 행위를 변경하지 않고 확장해야 한다.
- 기반 클래스의 모든 메서드가 서브 클래스에서 동일하게 동작해야 한다.
- IS-A 관계가 성립되어야 한다.

### 리스코프 치환의 원칙 위반 사례

1. 자식의 잘못된 메소드 오버로딩

```kotlin
open class Rectangle(val width: Int, val height: Int) {
    open fun area(): Int {
        return width * height
    }
}

class Square(size: Int) : Rectangle(size, size) {
    // 잘못된 오버로딩: 부모의 메서드 서명을 지키지 않음
    fun area(side: Int): Int {
        return side * side
    }
}

fun main() {
    val rectangle: Rectangle = Square(5)
    println(rectangle.area()) // 25를 기대하지만, Rectangle의 area()가 호출됨
}
```

2. 부모의 의도와 다르게 메소드 오버라이딩

```kotlin
open class Bird {
    open fun fly() {
        println("Flying")
    }
}

class Penguin : Bird() {
    // 부모의 의도와 다르게 메서드를 오버라이딩
    override fun fly() {
        println("Penguins can't fly")
    }
}

fun makeBirdFly(bird: Bird) {
    bird.fly()
}

fun main() {
    val bird: Bird = Penguin()
    makeBirdFly(bird) // "Penguins can't fly"가 출력됨
}
```

3. 잘못된 상속 관계 구성으로 인한 메서드 정의

```kotlin
open class Bird {
    open fun fly() {
        println("Flying")
    }
}

class Ostrich : Bird() {
    // 잘못된 상속 관계: Ostrich는 fly 메서드를 지원하지 않음
    override fun fly() {
        throw UnsupportedOperationException("Ostriches can't fly")
    }
}

fun makeBirdFly(bird: Bird) {
    bird.fly()
}

fun main() {
    val bird: Bird = Ostrich()
    makeBirdFly(bird) // 예외 발생: UnsupportedOperationException
}
```

### 리스코프 치환의 원칙 적용 주의점

다형성의 특징을 이용하기 위해 상위 클래스 타입으로 객체를 선언하여 하위 클래스의 인스턴스를 받으면, 업캐스팅된 상태에서 부모의 메서드를 사용해도 동작이 의도대로만 플러가도록 구성하면 되는 것이다. 그리고 LSP원칙의 핵심은 상속이다.

그러나 IS-A 관계가 성립이 되는 경우로만 제한되는데, 그 외 경우에는 합성(Composition)을 이용하도록 권고된다.

따라서 다형성을 이용하고 싶다면 추상 클래스 대신 인터페이스를 상속하여 인터페이스 타입으로 사용하기를 권고하며, 상위 클래스의 기능을 이용하거나 재사용을 하고 싶다면 상속 보단 합성으로 구성 하는 것이 낫다.

## 4. ISP(Interface Segregation Principle) 인터페이스 분리의 원칙

- 자신이 사용하지 않는 인터페이스는 구현하지 말아야 한다.
- 어떤 클래스가 다른 클래스에 종속될 때에는 가능한 최소한의 인터페이스를 사용해야한다.
- 인터페이스를 잘게 분리함으로써, 클라이언트의 목적과 용도에 적합한 인터페이스 만을 제공하는 것이다.
- SRP 원칙이 클래스의 단일 책임을 강조한다면, ISP원칙은 인터페이스의 단일 책임을 강조한다.

### ISP 적용 전 코드

```kotlin
interface Worker {
    fun work()
    fun eat()
}

class HumanWorker : Worker {
    override fun work() {
        println("Human working")
    }

    override fun eat() {
        println("Human eating")
    }
}

class RobotWorker : Worker {
    override fun work() {
        println("Robot working")
    }

    // RobotWorker는 eat() 메서드를 필요로 하지 않음
    override fun eat() {
        throw UnsupportedOperationException("Robots don't eat")
    }
}

fun main() {
    val human = HumanWorker()
    human.work()
    human.eat()

    val robot = RobotWorker()
    robot.work()
    // robot.eat() // 예외 발생
}
```

위 코드의 Worker 인터페이스는 work와 eat 메서드가 있다. 또한 HumanWorker, RobotWorker 클래스가 있는데, RobotWorker 은 eat 메서드가 필요하지 않으므로 ISP원칙을 위반한 셈이다.

### ISP 적용 코드

```kotlin
interface Workable {
    fun work()
}

interface Eatable {
    fun eat()
}

class HumanWorker : Workable, Eatable {
    override fun work() {
        println("Human working")
    }

    override fun eat() {
        println("Human eating")
    }
}

class RobotWorker : Workable {
    override fun work() {
        println("Robot working")
    }
}

fun main() {
    val human = HumanWorker()
    human.work()
    human.eat()

    val robot = RobotWorker()
    robot.work()
    // robot.eat() // 이 메서드는 존재하지 않음
}
```

Workable 과 Eatable 로 분리함으로써 각 객체에서 필요한 기능만 구현할 수 있게끔 수정할 수 있다.

### **ISP 원칙 적용 주의점**

ISP원칙의 주의해야 할점은 한 번 인터페이스를 분리하여 구성해놓고 나중에 무언가 수정사항이 생겨서 또 인터페이스를 분리하는 행위를 가하면 안된다.

이미 구현되어 있는 프로젝트에 또 인터페이스들을 분리한다면, 이미 해당 인터페이스를 구현하고 있는 온갖 클래스들과 이를 사용학도 있는 클라이언트에서 문제가 일어날 수 있기 때문이다.

본래 인터페이스라는건 한 번 구성하였으면 웬만해선 변하면 안되는 정책같은 개념이다.

따라서 처음 설계부터 기능의 변화를 생각해두고 인터페이스를 설계해야 하는데, 이는 현실적으로 힘든 부분이며 머리를 최대한 굴려하는 개인적인 역량에 달린 것이다.

## 5. DIP(Dependency Inversion Principle)

- 어떤 클래스를 참조해서 사용해야하는 상황이 생긴다면, 그 클래스를 직접 참조하는 것이 아니라 그 대상의 상위 요소(추상 클래스 or 인터페이스)로 참조하라는 원칙이다.
- 클라이언트가 상속 관계로 이루어진 모듈을 가져다 사용할 때, 하위 모듈을 직접 인스턴스를 가져다 쓰면 안된다
- 하위 모듈의 구체적인 내용에 클라이언트가 의존하게 되어 하위 모듈에 변화가 있을 때마다 클라이언트나 상위 모듈의 코드를 자주 수정해야 하는 번거로움이 발생하기 때문이다.

### DIP 적용 전 코드

```kotlin
class LightBulb {
    fun turnOn() {
        println("LightBulb: Turned on")
    }

    fun turnOff() {
        println("LightBulb: Turned off")
    }
}

class Switch {
    private val lightBulb = LightBulb()

    fun operate() {
        lightBulb.turnOn()
    }
}

fun main() {
    val switch = Switch()
    switch.operate()
}
```

위 코드의 Switch 클래스는 lightBulb 인스턴스를 생성하면서 직접 의존하고 있다. 이 경우에 lightBulb 클래스의 구현이 변경되면 Switch도 영향을 받게되므로 DIP를 위반하는 셈이다.

### DIP 적용 코드

```kotlin
interface Switchable {
    fun turnOn()
    fun turnOff()
}

class LightBulb : Switchable {
    override fun turnOn() {
        println("LightBulb: Turned on")
    }

    override fun turnOff() {
        println("LightBulb: Turned off")
    }
}

class Switch(private val device: Switchable) {
    fun operate() {
        device.turnOn()
    }
}

fun main() {
    val lightBulb = LightBulb()
    val switch = Switch(lightBulb)
    switch.operate()
}
```

수정된 코드는 Switchable 인터페이스를 추가해서 추상화 계층을 만들었다. Switch클래스에서 생성자로 Switchable 를 받아오며 Switchable 의 메서드를 사용가능하게 한다. 이렇게 되면 Switch는 LightBulb 의 구체적인 구현에 전혀 의존하지 않으므로 DIP원칙을 준수하게 된다.
