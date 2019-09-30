---
layout: post
title: "06. Kotlin type system"
date: 2019-09-30 16:31:28 -0400
categories: [kotlin]
tags: [kotlin, type, system]

---

# 6. 코틀린 타입 시스템

코틀린은 자바에서 지원해 주지 않는 nullable type과 읽기 전용 컬렉션을 지원한다. 또한 코틀린은 타입 시스템에서 불필요하거나 문제가 되던 부분을 제거했다. 배열 지원이 그런 예에 속한다.

## 6.1 널 가능성

nullability는  NullPointerException 오류(NPE)를 피할 수 있게 돕기 위한 코틀린 타입 시스템의 특성이다. NPE는 자세한 추가 정보가 없는 경우가 많은데, 코틀린을 비롯한 최신 언어에서의 트랜드는 가능한 한 이 문제를 실행 시점에서 컴파일 시섬으로 옮기는 것이다. 널이 될 수 있는지 여부를 타입 시스템에 추가함으로써 컴파일러가 여러 가지 오류를 컴파일 시 미리 감지해서 실행 시점에 발생할 수 있는 예외의 가능성을 줄일 수 있다.

이번 절에서 배울 것들 : 

- 코틀린에서 널이 될 수 있는 값을 어떻게 표기하는가?
- 코틀린에서 제공하는 도구가  nullable 값을 어떻게 처리하는가?
- nullable type을 코틀린과 자바에서 어떻게 함께 사용할 수 있는가?

### 6.1.1 nullable type

코틀린 타입 시스템은 널이 될 수 있는 타입을 명시적으로 지원한다.

NullPointerException은 nullable type에서 발생할 수 있는 exception이므로 안전하지 않다. 코틀린은 그런 메소드 호출을 금지함으로써 많은 오류를 방지한다.

```java
// java code. 이 함수에 null을 넘기면 NPE가 발생한다.
int strLen(String s) {
	return s.length();
}
```

strLen과 같은 함수를 코틀린으로 작성할 때 가장 먼저 "이 함수가 널을 인자로 받을 수 있는가?" 를 알아야 한다.

널이 인자로 들어올 수 없다면 코틀린에서는 다음과 같이 함수를 정의할 수 있다.

```kotlin
fun strLen(s: String) = s.length
```

strLen에  null or nullable variable을 넘기는 것은 금지되며, 혹시 그런 값을 넘기면 컴파일 시 오류가 발생한다.

이 함수가 널과 문자열을 인자로 받을 수 있게 하려면 타입 이름 뒤에 ?를 명시해야 한다.

```kotlin
fun steLen(s: String?) = s.length
```

String?, Int?, MyCustomType? 등 어떤 타입이든 타입 이름 뒤에 물음표를 붙이면 그 타입의 변수나 프로퍼니에  null 참조를 저장할 수 있다는 뜻이다.

모든 타입은 기본적으로 널이 될 수 없는 타입이다.

 nullable type 변수들을 몇몇 연산이 제한된다.

```kotlin
// 1. var.method() 처럼 메소드를 직접 호출할 수 없다.
>>> fun strLenSafe(s: String?) = s.length()
ERROR: only safe(?.) or nun-null asserted(!!.) calls are allowed on a nullable receiver of type kotlin.String?
// 2. nullable 타입의 변수를 non-null 타입 변수에 대입할 수 없다.
>>> val x: String? = null
>>> val y: String = x
ERROR: Type mismatch: inferred type is String? but String was expected
// 3. nullable type을 non-null value를 parameter로 사용하는 함수에 전달할 수 없다.
>>> strLen(x)
ERROR: Type mismatch: inferred type is String? but String was expected
```



nullable type 변수를 null과 비교하고, null이 아님이 확실한 영역에서는 해당 값을 널이 될 수 없는 타입의 값처럼 사용할 수 있다.

```kotlin
fun strLenSafe(s: String?): Int =
    if (s != null) s.length else 0

fun main(args: Array<String>) {
    val x: String? = null
    println(strLenSafe(x))
    println(strLenSafe("abc"))
}
```

코틀린은 nullable 변수가 null인지를 검사하기 위한 여러 도구를 제공한다.

### 6.1.2 타입의 의미

type : 타입은 classification으로 ... 타입은 어떤 값을이 가능한지와 그 타입에 대해 수행할 수 있는 연산의 종류를 결정한다.

String과 null은 type이 다르기 때문에 잘못 쓰면 에러가 발생하는 것이다.

코틀린의 nullable type은 이런 문제에 대해 종합적인 해법을 제공한다. 널이 될 수 있는 타입과 널이 될 수 없는 타입을 구분하면 각 타입의 값에 대해 어떤 연산이 가능할지 명확히 이해할 수 있고, 실행 시점에 예외를 발생시킬 수 있는 연산을 판단할 수 있다. 따라서 그런 연산을 아예 금지시킬 수 있다.

- 실행 시점에 nullable type과 non-nullable type 객체는 서로 같다 단지 컴파일 시점에  null 관련 검사가 진행 될 뿐이다. 따라서 실행 시점에 두 타입의 사용으로 인한 성능 저하는 발생하지 않는다.

이제 nullable 값을 잘, 쉽게 다룰 수 있게 도와주는 연산자들을 살펴본다.



### 6.1.3 안전한 호출 연산자 : ?.

?.은  null 검사와 메소드 호출을 한 번의 연산으로 수행한다.

```kotlin
// 각 줄은 서로 같은 의미이다.
s?.toUpperCase()
if(s != null) s.toUpperCase() else null
```

호출하려는 값이 null이면 이 호출은 무시되고 null이 결과 값이 된다.

![](https://dpzbhybb2pdcj.cloudfront.net/jemerov/Figures/06fig02.jpg)

s?.toUpperCase() 식의 결과 타입은 String? 타입이라는 것에 유의하라.

```kotlin
// nullable property인 Employee에 대해서도 ?. 연산자를 사용할 수 있다.
class Employee(val name: String, val manager: Employee?)

fun managerName(employee: Employee): String? = employee.manager?.name

fun main(args: Array<String>) {
    val ceo = Employee("Da Boss", null)
    val developer = Employee("Bob Smith", ceo)
    println(managerName(developer))
    println(managerName(ceo))
}
```



```kotlin
// 다음과 같이 ?. 연산자를 연쇄해서 사용하면 편할 때가 있다.
class Address(val streetAddress: String, val zipCode: Int,
              val city: String, val country: String)

class Company(val name: String, val address: Address?)

class Person(val name: String, val company: Company?)

fun Person.countryName(): String {
   val country = this.company?.address?.country
   return if (country != null) country else "Unknown"
}

fun main(args: Array<String>) {
    val person = Person("Dmitry", null)
    println(person.countryName())
}
```



### 6.1.4 엘비스 연산자: ?:

코틀린은 null 대신 사용할 디폴트 값을 지정할 때 편리하게 사용할 수 있는 연산자를 제공한다. 그 연산자를 elvis 연산자라고 한다.

```kotlin
fun foo(s: String?) {
	val t: String = s ?: ""
}
```



![](https://dpzbhybb2pdcj.cloudfront.net/jemerov/Figures/06fig03.jpg)



```kotlin
fun strLenSafe(s: String?): Int =
    if (s != null) s.length else 0
//
fun strLenSafe(s: String?): Int = s?.length ?: 0
```



```kotlin
class Address(val streetAddress: String, val zipCode: Int,
              val city: String, val country: String)

class Company(val name: String, val address: Address?)

class Person(val name: String, val company: Company?)

fun printShippingLabel(person: Person) {
    val address = person.company?.address
      ?: throw IllegalArgumentException("No address") //
    with (address) {
        println(streetAddress)
        println("$zipCode $city, $country")
    }
}

fun main(args: Array<String>) {
    val address = Address("Elsestr. 47", 80687, "Munich", "Germany")
    val jetbrains = Company("JetBrains", address)
    val person = Person("Dmitry", jetbrains)
    printShippingLabel(person)
    printShippingLabel(Person("Alexey", null))
}

```



### 6.1.5 안전한 캐스트 : as?

as는 자바 타입 캐스트와 마찬가지로 대상 값을 as로 지정한 타입으로 바꿀 수 없으면 ClassCastException이 발생한다. 물론 as를 사용할 때마다 is를 통해 미리 as로 변환 가능한 타입인지 검사해볼 수도 있다. 그.. 그치만!

as? 연산자는 어떤 값을 지정한 타입으로 캐스트한다. as?는 값을 대상 타입으로 변환할 수 없으면  null을 반환한다.

![](https://dpzbhybb2pdcj.cloudfront.net/jemerov/Figures/06fig04.jpg)

```kotlin
class Person(val firstName: String, val lastName: String) {
   override fun equals(o: Any?): Boolean {
      val otherPerson = o as? Person ?: return false // elvis 연간자와 흔히 같이 쓰인다.

      return otherPerson.firstName == firstName &&
             otherPerson.lastName == lastName
   }

   override fun hashCode(): Int =
      firstName.hashCode() * 37 + lastName.hashCode()
}

fun main(args: Array<String>) {
    val p1 = Person("Dmitry", "Jemerov")
    val p2 = Person("Dmitry", "Jemerov")
    println(p1 == p2)
    println(p1.equals(42))
}
```



elvis 연산자와 함께 사용하는 이 패턴을 사용하면 파라미터로 받은 값이 원하는 타입인지 쉽게 검사하고 캐스트할 수 있고, 타입이 맞지 않으면 쉽게 false를 반환할 수 있다.



### 6.1.6 널 아님 단언: !! (널 값은 너굴맨이 처리했으니 걱정하지 말라구!)

어떤 값이던 !!와 함께 사용하면 non-nullable 타입으로 강제로 변환할 수 있다. 실제 null 값에 대해 !!를 적용하면 NPE가 발생한다.

![](https://dpzbhybb2pdcj.cloudfront.net/jemerov/Figures/06fig05.jpg)



다음과 같이 nullable type을 non-nullable type으로 변환할 수 있다.

```kotlin
fun ignoreNulls(s: String?) {
    val sNotNull: String = s!! // 컴파일러는 이 연산을 검사하지 않는다.
    println(sNotNull.length) // 검사하지 않아서 진짜 null이였을 때 NPE가 발생하는 곳은 이곳이다.
}

fun main(args: Array<String>) {
    ignoreNulls(null)
}
```



### 6.1.7 let 함수

let 함수를 사용하면 널이 될 수 있는 식을 더 쉽게 다룰 수 있다.

![](https://dpzbhybb2pdcj.cloudfront.net/jemerov/Figures/06fig06.jpg)

- foo가 null아 아닐 때만 람다 함수가 실행된다.

```kotlin
fun sendEmailTo(email: String) { /**/ } // non-nullable parameter를 받는 함수

>>> val email: String? = "abe@never.org"
>>> sendEmailTo(email) // 안됨
ERROR: Type mismatch: inferred type is String? but String was expected

>>> if(email != null) sendEmailTo(email) // 됨
```

- let을 사용해서 더 쉽게 사용해보자

```kotlin
email?.let { email -> sendEmailTo(email) }
```



```kotlin
val person: Person? = getTheBestPersonInTheWorld()
if(person != null) sendEmailTo(person.email)
// 위 아래 서로 같은 코드임
getTheBestPersonInTheWorld()?.let { sendEmailTo(it.email) }
```



### 6.1.8 나중에 초기화할 프로퍼티

객체 인스턴스를 인단 생성한 다음에 나중에 초기화하는 프레임워크가 많다. 예를 들어 안드로이드에서는 onCreate에서 액티비티를 초기화한다.

하지만 코틀린에서는 일반적으로 생성자에서 모든 프로퍼티를 초기화해야 한다. non-nullable type일 경우 null이 아닌 값으로 초기화를 해 줘야 하는데, 적절한 값을 제공해 줄 수 없다면 nullable type으로 만든 뒤 null값으로 초기화를 해야 한다.

한 번 nullable type으로 선언한 다음에는 해당 타입을 사용하기 위해 모든 프로퍼티 접근에 널 검사를 넣거나 !! 연산자를 써야한다.

```kotlin
import org.junit.Before
import org.junit.Test
import org.junit.Assert

class MyService {
    fun performAction(): String = "foo"
}

class MyTest {
    private var myService: MyService? = null // nullable var

    @Before fun setUp() {
        myService = MyService() // setUp Fixture에서 진짜 초깃값을 지정
    }

    @Test fun testAction() {
        Assert.assertEquals("foo",
            myService!!.performAction()) // myService가 nullable이므로 !!가 필요하다.
    }
}
```

위의 코드는 보기 나쁘다.

이를 해결하기 위해  myService 프로퍼티를 late-initialized 할 수 있다. lateinit 변경자를 붙이면 프로퍼티를 나중에 초기화 할 수 있다.

```kotlin
import org.junit.Before
import org.junit.Test
import org.junit.Assert

class MyService {
    fun performAction(): String = "foo"
}

class MyTest {
    private lateinit var myService: MyService

    @Before fun setUp() {
        myService = MyService()
    }

    @Test fun testAction() {
        Assert.assertEquals("foo",
            myService.performAction())
    }
}
```



late-initialized property는 항상 var여야 한다. val 프로퍼티는 final 필드로 컴파일되며, 생성자 안에서 반드시 초기화해야 한다.

late-initialized property는 생성자 밖에서 초기화 되므로, 반드시 var여야 한다.

late-initialized property를 선언해 놓고 초기화 전에 프로퍼티에 접근하면, "lateinit property myService has not been initialized"라는 에러가 발생한다.



### 6.1.9 널이 될 수 있는 타입 확장

널이 될 수 있는 타입에 대한 확장 함수를 정의하면 null 값을 다루는 강력한 도구로 활용할 수 있다. 어떤 메소드를 호출하기 전에 수신 객체 역할을 하는 변수가 널이 될 수 없다고 보장하는 대신, 직접 변수에 대해 메소드를 호출해도 확장 함수인 메소드가 알아서 널을 처리해준다. 이런 처리는 확장 함수에서만 가능하다. 일반 멤버 호출은 객체 인스턴스를 통해 디스패치(dispatch) 되므로 그 인스턴스가 널인지 여부를 검사하지 않는다.



String? 타입의 수신 객체에 대해 호출할 수 있는 isNullOrEmpty나 isNullOrBlank 메소드가 있다.

```kotlin
fun verifyUserInput(input: String?) {
    if (input.isNullOrBlank()) { // ?. 형식의 안전한 호출을 사용하지 않아도 된다.
        println("Please fill in the required fields")
    }
}

fun main(args: Array<String>) {
    verifyUserInput(" ")
    verifyUserInput(null)
}
```



![](https://dpzbhybb2pdcj.cloudfront.net/jemerov/Figures/06fig07.jpg)

```kotlin
fun String?.isNullOrBlank(): Boolean =  
        this == null || this.isBlank()   // this는 null이 될 수 있다. 명시적으로 null check 필요
```



```kotlin
>>> val person: Person? = ...
>>> person.let { sendEmailTo(it) }  // let을 그냥 사용하면 it은 nullable type이다.
ERROR: Type mismatch: inferred type is Person? but Person was expected
//
>>> person?.let { sendEmailTo(it) } // 안전한 호출 연산인 ?.를 사용해야 it이 null이 아님이 보장된다.
```



### 6.1.10 타입 파라미터의 널 가능성

코틀린에서는 함수나 클래스의 모든 타입 파라미터는 기본적으로 nullable이다.

```kotlin
fun <T> printHashCode(t: T) { // fun <T: Any?> 와 같은 모양이다.
    println(t?.hashCode()) // t는 nullable이므로 ?. 안전한 연산을 사용해야 한다.
}

fun main(args: Array<String>) {
    printHashCode(null)
}
```

t는 nullable이다.

타입 파라미터가 널이 아님을 확실히 라혀면 널이 될 수 없는 타입 상한(upper bound)를 지정해야 한다. 이렇게 널이 될 수 없는 타입 상한을 지정하면 널이 될 수 있는 값을 거부하게 된다.

```kotlin
fun <T: Any> printHashCode(t: T) { // 이제 T는 non-nullable 타입이다.
    println(t.hashCode())
}
>>> printHashCode(null)            
Error: Type parameter bound for `T` is not satisfied
>>> printHashCode(42)
42
```



### 6.1.11 널 가능성과 자바

이제 지금까지 배운 null 관련 기능들을 자바와 함께 사용했을 때 어떻게 되는지 확인해보자.

자바 코드에도 애노테이션으로 표시된 널 가능성 정보가 있다. 이런 정보가 코드에 있으면 코틀린도 그 정보를 활용한다.

![](https://dpzbhybb2pdcj.cloudfront.net/jemerov/Figures/06fig08.jpg)

JSR-305 표준, 안드로이드, 젯브레인스 도구들이 지원하는 애노테이션과 같이 코틀린이 인식할 수 있는 null annotation은 많다. 이런 널 가능성 애노테이션이 소스코드에 없는 경우는 자바의 타입은 코틀린의 platform 타입이 된다.



#### 플랫폼 타입

플랫폼 타입은 코틀린이 널 관련 정보를 알 수 없는 타입을 말한다. 그 타입을 널이 될 수 있는 타입으로 처리해도 되고 널이 될 수 없는 타입으로 처리해도 된다. (=책임은 너의 것) 컴파일러는 모든 연산을 허용한다.



![](https://dpzbhybb2pdcj.cloudfront.net/jemerov/Figures/06fig09.jpg)

```java
/* Java */
public class Person {
    private final String name;

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
// 코틀린에서 Person 사용하기 - Person은 플랫폼 타입이 된다.
fun yellAt(person: Person) {
    println(person.name.toUpperCase() + "!!!")
}

fun main(args: Array<String>) {
    yellAtSafe(Person(null)) // error!
}
>>> java.lang IllegalArgumentException: Parameter specified as non-null
  is null: method toUpperCase, parameter $receiver
```



- 코틀린 텀파일러는  null 값을 사용할 때가 아니라 null 값이 함수에 넘겨질 때 error를 발생시킨다.
- 자바 라이브러리를 사용하고 싶다면 문서를 잘 살펴보고, 메소드가 널을 반환할지 알아보고 적절히 사용한다.



**상속**

코틀린에서 자바 메소드를 오버라이드 할 때 그 메소드의 파라미터와 반환 타입을 널이 될 수 있는 타입으로 선언할지 널이 될 수 없는 타입으로 선언할지 결정해야 한다.



``` kotlin
interface StringProcessor {
  void process(String value);
}
//
class StringPrinter : StringProcessor {
  override fun process(value: String) {
    println(value)
  }
}
//
class NullableStringPrinter : StringProcessor {
  override fun process(value: String?) {
    if(value != null) {
      print(value)
    }
  }
}
```







