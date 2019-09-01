# 3. 함수 정의와 호출

내용들

컬렉션, 문자열, 정규식을 다루기 위한 함수

이름 붙인 인자, 디폴트 파라미터 값, 중위 호출 문법 사용

확장 함수와 확장 프로퍼티를 사용해 자바 라이브러리 적용

최상위 및 로컬 함수와 프로퍼티를 사용해 코드 구조화



## 3.1 코틀린에서 컬렉션 만들기

```kotlin
val set = hashSetOf(1, 7, 53)
val list = arrayListOf(1, 7, 53)
val map = hashMapOf(1 to "one", "seven" to "seven", 53 to "fifty-three")

// 생성된 객체의 클래스 알아보기

>> println(set.javaClass) // .javaClass는 자바의 getClass()에 해당하는 코틀린 코드다
class java.util.HashSet
>> println(list.javaClass)
class java.util.ArrayList
```



## 3.2 함수를 호출하기 쉽게 만들기

```kotlin
// 모든 원소를 찍어보는 코드.
val list  = listOf(1,2,3)
println(list) // toString() 호출
>> [1,2,3]
println(joinToString(list, sep, start, end)) 와 같이 컬렉션을 포메팅 해서 출력할 수 있는 코드를 구현해보자.
```



```kotlin
fun<T> joinToString ( // 질문 fun<T> generic 하게 하려면 이게 필요하다는 뜻인가?
	collection: Collection<T>,
	separator: String,
	prefix: String,
	postfix: String
): String {
  val result = StringBuilder(prefix)
  for((index, element) in collection.withIndex()) {
    if(index > 0) result.append(separator)
    result.append(element)
  }
  result.append(postfix)
  return result.toString()
}
```



### 3.2.1 이름 붙인 인자

joinToString의 문제점 1 - 가독성이 좋지 않다.

```kotlin
println(joinToString(list, "; ", "(", ")")) // 각 인자가 어떤 역할을 하는지 파악하기 힘들다
// 코틀린에서는 다음과 같이 가독성을 높일 수 있다.
println(joinToString(list, separator = " ", prefix = " ", postfix = " "))
// list는 named parameter가 아니다. 하지만 named parameter를 사용하기 시작하면 그 이후의 모든 파라미터들은 
// named parameter가 되어야 한다.
```

자바로 작성한 코드를 호출할 때는 이름 붙인 인자를 사용할 수 없다.

### 3.2.2 디폴트 파라미터 값

자바에서는 일부 클래스에서 오버로딩 메소드가 너무 많아진다는 문제가 있다. ex) java.lang.Thread

코틀린에서는 함수 선언에서 파라미터의 디폴트 값을 지정할 수 있으므로 이런 오버로드 중 상당수를 피할 수 있다.

joinToString 함수에 default parameter를 적용해보자.

```kotlin
fun<T> joinToString ( // 질문 fun<T> generic 하게 하려면 이게 필요하다는 뜻인가?
	collection: Collection<T>,
	separator: String = ", ",
	prefix: String = "",
	postfix: String = ""
): String ...
```



```kotlin
joinToString(list, ", ", "", "")
joinToString(list)
joinToString(list, "; ")
joinToString(list, postfix = ";", prefix = "# ") // named parameter를 응용하면 순서에 상관없이 인자를 전달 할 수 있다.
```

자바에는 디폴트 파라미터 개념이 없어서 자바에서 디폴트 파라미터를 사용하는 코틀린 코드를 가져다 쓰고 싶다면

@JvmOverloads annotation를 함수에 추가해야 한다.

```kotlin
@JvmOverloads
fun<T> joinToString ( 
	collection: Collection<T>,
	separator: String = ", ",
	prefix: String = "",
	postfix: String = ""
): String ...
```

```java
// JvmOverloads를 추가한 코틀린 함수는 자바에서 다음과 같은 overload들이 자동으로 생성된다.
String joinToString(Collenction<T> collenction, String separator, String prefix, String postfix);
String joinToString(Collenction<T> collenction, String separator, String prefix);
String joinToString(Collenction<T> collenction, String separator, String prefix);
String joinToString(Collenction<T> collenction);
```



### 3.2.3 정적인 유틸리티 클래그 없애기: 최상위 함수와 프로퍼티

**최상위 함수**

자바에서는 모든 코드를 클래스의 메소드로 작성해야 한다. 하지만 코틀린은 이럴 필요가 없다.

함수를 소스 파일의 최상위 수준에 위치시키면 코틀린 컴파일러가 자동으로 클래스를 생성해준다.

```kotlin
// join.kt file
// JvmName annotation을 파일의 맨 앞에 적용하면 자동으로 생성될 클래스의 이름을 명시적으로 넘겨줄 수 있다. 넘겨주지 않으면 파일이름이 사용된다.
@file:JvmName("StringFunctions")
package strings
fun joinToString(...): String {..}

//
package strings
public class StringFunctions {
	public static String joinToString(...) {...}
}
```

**최상위 프로퍼티**

```kotlin
var opCount = 0; // 변수
fun performOperation() {
	opCount++
}
val UNIX_LINE_SEPARATOR = "\n" 

// 프로퍼티는 getter, setter를 통해 접근하지만 상수를 getter로 접근하는것은 어색하다.
// const를 사용하면 자바에서도 getter말고 그냥 접근 가능.
const val UNIX_LINE_SEPARATOR = "\n" // kotlin
-> public static final String UNIX_LINE_SEPARATOR = "\n"; // java
```



3.3 메소드를 다른 클래스에 추가 : 확장 함수와 확장 프로퍼티

기존 자바 API를 재작성하지 않고도 코틀린이 제공하는 여러 편리한 기능을 사용할 수 있으면 좋겠다.

**확장 함수(extensrion function)** : 어떤 클래스의 멤버 메소드인 것처럼 호출할 수 있지만 그 클래스 밖에 선언된 함수.

![](https://miro.medium.com/max/916/1*pqu04vgv2R-RfzcfIZEJug.png)

```kotlin
println("Kotlin".lastChar()) // String이 receiver type, "Kotlin"이 receiver object
n
```

JVM 언어로 작성되었다면 확장이 가능하다.

확장 함수 내부에서는 private, protected 멤버를 사용할 수 없다.

### 3.3.1 임포트와 확장 함수 & 3.3.2 자바에서 확장 함수 호출

확장함수를 사용하기 위해서는 추가 임포트가 필요하다.

```kotlin
import strings.lastChar
import strings.*
import strings.lastChar as last
"Kotlin".last()
// 자바에서 사용하려면
StringUtilKt.last("Java");
```

### 

### 3.3.3 확장 함수로 유틸리티 함수 정의

```kotlin
fun <T> Collection<T>.joinToString( // Collection<T>에 대한 확장 함수
        separator: String = ", ",
        prefix: String = "",
        postfix: String = ""
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) { // this == receive object
        if (index > 0) result.append(separator)
        result.append(element)
    }

    result.append(postfix)
    return result.toString()
}

fun Collection<String>.join(
        separator: String = ", ",
        prefix: String = "",
        postfix: String = ""
) = joinToString(separator, prefix, postfix)

fun main(args: Array<String>) {
    println(listOf("one", "two", "eight").join(" "))
}
```



### 3.3.4 확장 함수는 오버라이드할 수 없다.

```kotlin
open class View {
	open fun click() = println("View clicked")
}
class Button: View() {
	override fun click() = println("Button clicked")
}

val view: View = Button()
view.click()
Button clicked
```

다음과 같이  Button이 오버라이드 한 메소드가 호출되는 이유는 호출 메소드가 동적으로 정해진다.

하지만 확장 함수는 정적으로 정해지기 때문에 오버라이드가 되지 않는다.

### 3.3.5 확장 프로퍼티

확장 프로퍼티를 사용하면 프로퍼티 형식의 구문으로 사용할 수 있는 API를 추가할 수 있다. 하지만 기존 class에 필드를 추가하는 식으로 동작하지는 않는다.

lastChar이라는 함수를 프로퍼티로 바꿔보자

```kotlin
val String.lastChar: Char
	get() = get(lenght - 1)
```

기본 getter는 반드시 구현을 해야 한다.

```kotlin
var StringBuilder.lastChar: Char
	get() = get(length - 1)
	set(value: Char) {
		this.setCharAt(length - 1, value)
	}

println("Kotlin".lastChar)
val sb = StringBuilder("Kotlin?")
sb.lastChar = '!'
println(sb)
```



## 3.4 컬렉션 처리:  가변 길이 인자, 중위 함수 호출, 라이브러리 지원

- varargs 로 가변 길이 인자 함수 만들기
- 중위(infix) 함수 호출 구문을 사용해서 인자 1개짜리 메소드 간편하게 호출하기
- 구조 분해 선언(destructruing declaration)을 사용하면 복합적인 값을 분해해서 여러 변수에 나눠 담을 수 있다.

### 3.4.1 자바 컬렉션 API 확장

```kotlin
val strings: List<String> = listOf("first", "second", "fourtheenth")
strings.last()
```

코틀린은 자체 컬렉션 라이브러리가 없는데 어떻게 이런 코드가 동작할 수 있었건 이유는 바로 확장함수를 이용해서 확장시켜 놓았기 때문이다.

3.4.2 가변 인자 함수:  인자의 개수가 달라질 수 있는 함수 정의

```kotlin
val list = listOf(2,3,5,7,11)
// listOf라는 함수는 가변 인자를 가진다.
fun listOf<T>(vararg values: T): List<T> {...}
```



스프레드 연산자 (*) :  배열을 가변 인자로 함수에 넘겨주고 싶을 때 배열을 풀어서 하나하나 함수에 전달해주는 연산자

```kotlin
fun main(args: Array<String>) {
	val list = listOf("args: ", *args)
	println(list)
}
```



### 3.4.3 값의 쌍 다루기: 중위 호출과 구조 분해 선언

```kotlin
val map = mapOf(1 to "one", 7 to "seven", 53 to "fifty-three") // to라는 일반 메소드를 중위 호출 한 문장

//
// 다음 두 호출은 동일하다.
1.to("one")
1 to "one" 
```

인자가 하나뿐인 일반 메소드나 인자가 하나뿐인 확장 함수에 중위 호출을 사용할 수 있다.

```kotlin
//  이게 되네
fun main(args: Array<String>) {
	val (number, name, real) = listOf(1,2,3)
    println("$number $name $real")
}
```

![](https://miro.medium.com/max/300/1*MoBo4bnwM06rvLfXfeJIcw.png)

## 3.5 문자열과 정규식 다루기

### 3.5.1 문자열 나누기

```kotlin
println("12.345-6.A".split("\\.|-".toRegex())) // 명시적으로 정규식으로 바꾼다.
println("12.345-6.A".split(".", "-") // 여러 구분 문자열로 쪼갠다.
```



### 3.5.2 정규식과 3중 따옴표로 묶은 문자열

```kotlin
fun parsePath(path: String) {
    val directory = path.substringBeforeLast("/")
    val fullName = path.substringAfterLast("/")

    val fileName = fullName.substringBeforeLast(".")
    val extension = fullName.substringAfterLast(".")

    println("Dir: $directory, name: $fileName, ext: $extension")
}

fun main(args: Array<String>) {
    parsePath("/Users/yole/kotlin-book/chapter.adoc")
}

//

fun parsePath(path: String) {
    val regex = """(.+)/(.+)\.(.+)""".toRegex() // .을 이스케이프 하기 위해 \\가 아닌 \을 사용
    val matchResult = regex.matchEntire(path)
    if (matchResult != null) {
        val (directory, filename, extension) = matchResult.destructured
        println("Dir: $directory, name: $filename, ext: $extension")
    }
}

fun main(args: Array<String>) {
    parsePath("/Users/yole/kotlin-book/chapter.adoc")
}
```

### 3.5.3 여러 줄 3중 따옴표 문자열

```kotlin
val kotlinLogo = """| //
                   .|//
                   .|/ \""" // indent가 엄청 길게 발생했다. 이걸 지워주기 위해 indent 구분자 .을 추가

fun main(args: Array<String>) {
    println(kotlinLogo.trimMargin(".")) // trimMargin으로 indent 날리기
}
```



## 3.6 코드 다름기: 로컬 함수와 확장

메소드를 너무 쪼개서 사용하면 작은 메소드가 너무 많아진다.

많은 작은 메소드를 내부 클래스에 넣으면 준비 코드가 늘어난다.

코틀린에서 코드 중복을 로컬 함수를 통해 해결해보자.

```kotlin
// 코드 중복이 있는 코드
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {
    if (user.name.isEmpty()) {
        throw IllegalArgumentException(
            "Can't save user ${user.id}: empty Name")
    }

    if (user.address.isEmpty()) {
        throw IllegalArgumentException(
            "Can't save user ${user.id}: empty Address")
    }

    // Save user to the database
}

fun main(args: Array<String>) {
    saveUser(User(1, "", ""))
}
```



```kotlin
// 검증 구문을 로컬 함수로 바꿈
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {

    fun validate(user: User,
                 value: String,
                 fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException(
                "Can't save user ${user.id}: empty $fieldName")
        }
    }

    validate(user, user.name, "Name")
    validate(user, user.address, "Address")

    // Save user to the database
}

fun main(args: Array<String>) {
    saveUser(User(1, "", ""))
}

```



```kotlin
// 로컬 함수는 자신이 속한 바깥 함수의 모든 파라미터와 변수를 사용할 수 있다.
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {
    fun validate(value: String, fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException(
                "Can't save user ${user.id}: " + // user.id에 접근하는 모습
                    "empty $fieldName")
        }
    }

    validate(user.name, "Name")
    validate(user.address, "Address")

    // Save user to the database
}

fun main(args: Array<String>) {
    saveUser(User(1, "", ""))
}
```



```kotlin
// 검증 구문을 User class의 확장함수로 만들었다.
// 하지만 어떤 어떤 값을을 검증하고 있는지 가려지기는 한다.
class User(val id: Int, val name: String, val address: String)

fun User.validateBeforeSave() { 
    fun validate(value: String, fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException(
               "Can't save user $id: empty $fieldName")
        }
    }

    validate(name, "Name")
    validate(address, "Address")
}

fun saveUser(user: User) {
    user.validateBeforeSave()

    // Save user to the database
}

fun main(args: Array<String>) {
    saveUser(User(1, "", ""))
}
```

