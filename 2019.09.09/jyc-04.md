---
layout: post
title: "04. Kotlin class, object, interface"
date: 2019-09-01 21:31:28 -0400
categories: [kotlin]
tags: [kotlin, class, object, interface]
---

# 4. 클래스, 객체, 인터페이스

## 4.1 클래스 계층 정의

### 4.1.1 코틀린 인터페이스

코틀린 인터페이스는 자바 8 인터페이스와 비슷하다.

코틀린 인터페이스에는 추상 메소드뿐 아니라 디폴트 구현이 있는 메소드로 정의할 수 있다. 하지만 상태(필드)는 들어갈 수 없다.

코틀린에서 클래스는 클래스로, 인터페이스는 인터페이스로 정의한다.

```kotlin
// 인터페이스 정의
interface Clickable {
	fun click()
  fun showOff() = println("I'm clickable!") // default implements
}
// 인터페이스 구현
class Button : Clickable {
  override fun click() = println("I was clicked")
}
```

자바와 마찬가지로 클래스는 1개만, 인터페이스는 원하는 만큼 구현할 수 있다.

override 변경자는 필수다.  override 변경자를 사용하지 않으면 컴파일이 되지 않는다. 함수의 이름을 변경해야 한다.



 두 개 이상의 인터페이스를 구현할 때, 둘 이상의 디폴트 구현이 있는 경우는 명시적 override가 필요하다.

```kotlin
interface Focusable {
	fun setFocus(b: Boolean) = 
		println("I $(if (b) "got" else "lost"} focus.")
  
  fun showOff() = println("I'm focusable!")
}
// 클래스가 showOff method를 가지고 있는 Clickable, Focusable을 모두 상속한다면
// default implement가 있는 showOff의 중복이 있으므로 override 해야한다.
class Button : Clickable, Focusable {
  override fun click() = println("I was clicked")
  override fun showOff() {
    super<Clickable>.showOff()
    super<Focusable>.showOff()
  }
}
```



#### 자바에서 코틀린의 메소드가 있는 인터페이스 구현하기

자바에서 코틀린 인터페이스를 구현할 수 있지만, 자바6은 디폴트 구현을 지원하지 않는다. (코틀린은 자바6 호환이다.)

따라서 자바에서 디폴트 구현이 있는 코틀린 인터페이스를 상속, 구현할 때는 인터페이스가 자바 인터페이스와 자바 클래스로 나뉘어져 컴파일된다.

?? : 그러므로 코틀린 메소드 본문을 제공하는 메소드를 포함하는 모든 메소드에 대한 본문을 작성해야 한다.(즉 자바에서는 코틀린의 디폴트 메소드 구현에 의존할 수 없다.)

### 4.1.2 open, final, abstract 변경자 : 기본은 final

취약한 기반 클래스(fragile base class) 문제 : 하위 클래스가 기반 클래스에 대해 어떠한 가정을 전제로 구현이 되어있던 상황에서 기반 클래스가 변경되어 생기는 문제.

이 문제를 해결하기 위해 자바에서는 "상속을 위한 설계와 문서를 갖추거나, 그럴 수 없다면 상속을 금지하라"라고 말한다. 코틀린은 기본적으로 클래스와 메소드가 final이다.

```kotlin
open class RichButton : Clickable { // RichButton 클래스는 다른 클래스가 상속할 수 있다.
	fun disable() {} // 이 메소드는 하위 클래스가 override 할 수 없다.
	open fun animate() {} // 이 메소드는 하위 클래스가 override 할 수 있다.
	override fun click() {} // override한 메소드는 기본적으로 open이다.
//   final override fun click() {} // 다음과 같이 override 한 method를 finalize할 수 있다.
}
```

코틀린에도 인스턴스화를 할 수 없는 abstract class가 있다.

```kotlin
abstract class Animated {
	abstract fun animate() // 추상함수, 반드시 override 해야 함
	open fun stopAnimating() { // 비추상함수, open으로 override를 허용해 줄 수 있다.
    // some implementing..
	}
	fun animateTwice() {	
	}
}
```

| 변경자   | 변경자 속성               | 설명                                                         |
| -------- | ------------------------- | ------------------------------------------------------------ |
| final    | 오버라이드 불가능         | 클래스 멤버의 기본 변경자                                    |
| open     | 오버라이드 가능           | 오버라이드가 필요하다면 명시해야함                           |
| abstract | 반드시 오버라이드 해야 함 | 추상 클래스에서만 사용 가능.<br />이 변경자가 있으면 구현이 있으면 안됨 |
| override | 오버라이드 할게~          | override 멤버는 기본으로 open  되어있음.<br />오버라이드 금지를 위해서는 final을 명시해야 함. |

### 4.1.3 가시성 변경자(access modifier : public, private, protected, internal) : public이 default

가시성 변경자는 클래스 외부에서의 접근을 제어한다.

| 변경자          | 클래스 멤버                      | 최상위 선언                    |
| --------------- | -------------------------------- | ------------------------------ |
| public(default) | 어디서나 볼 수 있다.             | 어디서나 볼 수 있다.           |
| internal        | 같은 모듈 안에서만 볼 수 있다.   | 같은 모듈 안에서만 볼 수 있다. |
| protected       | 하위 클래스 안에서만 볼 수 있다. | 최상위에 사용불가              |
| private         | 같은 클래스 안에서만 볼 수 있다. | 같은 파일 안에서만 볼 수 있다. |

```kotlin
internal open class TalkativeButton : Focusable {
	private fun yell() = println("Hey!")
	protected fun whisper() = println("Let's talk!")
}
fun TalkativeButton.giveSpeech() { // error?
	yell() // error!
	whisper() // error!
}
```

자바에서는 같은 패키지 안에서 protected멤버에 접근할 수 있지만 코틀린은 아니다.

#### 코틀린의 가시성 변경자와 자바

internal은 자바에서 public이 된다.

### 4.1.4 내부 클래스와 중첩된 클래스 : 기본적으로 중첩 클래스

코틀린에서도 클래스 안에 다른 클래스를 선언할 수 있다.

이 중첩 클래스(nested class)는 기본적으로는 바깥쪽 클래스 인스턴스에 대한 접근 권한이 

자바는 있지만 코틀린은 없다.

코드로 이해해보자

```kotlin
interface State: Serializable
interface View {
	fun getCurrentState() : State
	fun restoreState(state: State) {}
}
class Button : View {
  override fun getCurrentState() : State = ButtonState()
  override fun restoreState(state: State) {/**/}
  class ButtonState: State {/**/}
}

```

![inner class](https://miro.medium.com/max/200/1*LDQVi952GYAwRIbcIW4E4w.png)

```kotlin
class Outer {
	inner class Inner {
		fun getOuterReference(): Outer = this@Outer // Outer class 참조 문법
	}
}
```



### 4.1.5 봉인된 클래스 : 클래스 계층 정의 시 계층 확장 제한

```kotlin
interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr
fun eval(e: Expr): Int =
	when(e) {
    is Num -> e.value
    is Sum -> eval(e.right) + eval(e.left)
    else ->
    	throw IllegalArgumentException("Unknown expression")
  }
```

코틀린 컴파일러는 when을 이용하여 Expr 타입의 값을 검사할 때  else 문을 포함하기를 강제한다.

이런 구현은 새로운 클래스를 추가했지만 when 분기에 잘 추가해주지 않았을 때, else 분기를 타게 되면서 버그를 유발할 수 있다.

코틀린은 이런 문제에 대한 해답으로 sealed 클래스를 제공한다.

sealed 상위 클래스에서, 상속받을 하위 클래스를 제한할 수 있다.

sealed 클래스는 자동으로 open이다.

```kotlin
sealed class Expr { // sealed 기반클래스
	class Num(val value: Int): Expr() // 모든 하위클래스를 중첩 클래스로 나열
	class Sum(val left: Expr, val right: Expr): Expr() // ? Expr()은 무엇이냐
}
fun eval(e: Expr): Int =
	when(e) {
    is Expr.Num -> e.value // sealed 안의 모든 하위클래스가 처리되어 else가 필요없다
    is Expr.Sum -> eval(e.right) + eval(e.left)
  }
```



## 4.2 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언

자바도 생성자를 여러 개 만들 수 있지만, 코틀린은 주 생성자와 부 생성자를 구분한다. 또한 초기화 블록을 통해 초기화 로직을 추가할 수 있다.

### 4.2.1 클래스 초기화: 주 생성자와 초기화 블록

```kotlin
class User(val nickname: String) // 클래스 이름 뒤에 오는 괄호로 둘러싸인 코드를 주 생성자라고 한다.
// 이 User class를 길게 풀어쓰면 다음과 같다
class User constructor(_nickname: String) { // 주 생성자
  val nickname: String
  init { // 초기화 구문
    nickname = _nickname
  }
}
//
class User(val nickname: String) // constructor 생략가능, val 키워드를 사용하면 자동으로 nickname이 class property로 초기화된다.
//
class User(val nickname: String, val isSubscribed: Boolean = true) // default parameter 가능
val hyun = User("진행") // new 키워드 없이 생성자를 직접 호출하면 된다.
```

- constructor : 주 생성자 또는 부 생성자 정의를 시작할 때 사용한다.
- init : 초기화 블록을 시작한다. 주 생성자와 함께 사용된다.

```kotlin
open class User(val nickname: String) {}
class TwitterUser(nickname: String): User(nickname) {} // 기반 클래스의 constructor 호출
//
open class Button
class RadioButton: Button() // interface와는 다르게 무조건 ()이 들어가야 한다.
```

### 4.2.2 부 생성자 : 상위 클래스를 다른 방식으로 초기화

```kotlin
class MyButton: View {
  constructor(ctx: Context): super(ctx) {} // 상위 클래스의 생성자 호출
  constructor(ctx: Contect, attr: AttributeSet): super(ctx, attr) {}
}
//
class MyButton: View {
  constructor(ctx: Context): this(ctx, MY_STYLE) {} // this로 자신의 다른 생성자 호출
  constructor(ctx: Contect, attr: AttributeSet): super(ctx, attr) {}
}
```

클래스에 주 생성자가 없다면 모든 부 생성자는 반드시 상위 클래스를 초기화하거나 다른 생성자에게 생성을 위임해야 한다.

### 4.2.3 인터페이스에 선언된 프로퍼티 구현

```kotlin
interface User { // User 인터페이스를 구현하는 클래스는 nickname의 값을 얻을 수 있는 방법을 제공해야 한다.
	val nickname: String
}
```



``` kotlin
fun getFacebookName(accountId: Int) = "fb:$accountId"

interface User {
    val nickname: String
}
class PrivateUser(override val nickname: String) : User

class SubscribingUser(val email: String) : User {
    override val nickname: String
        get() = email.substringBefore('@')
}

class FacebookUser(val accountId: Int) : User {
    override val nickname = getFacebookName(accountId)
}

fun main(args: Array<String>) {
    println(PrivateUser("test@kotlinlang.org").nickname)
    println(SubscribingUser("test@kotlinlang.org").nickname)
}
```

### 4.2.4 게터와 세터에서 뒷받침하는 필드에 접근

```kotlin
class User(val name: String) {
  var address: String = "unspecified"
  	set(value: String) {
      field = value // field라는 특별한 식별자로 필드에 접근할 수 있다.
    }
}
```

### 4.2.5 접근자의 가시성 변경

```kotlin
class LengthCounter {
	var counter: Int = 0
		private set // 클래스 밖에서의 프로퍼티 값 변경을 막는다.
	fun addWord(word: String) {
		counter += word.length
	}
}
```



## 4.3 컴파일러가 생성한 메소드: 데이터 클래스와 클래스 위임

코틀린은  equals, hashCode, toString과 같은 코드를 보이지 않는 곳에서 자동으로 해준다.

### 4.3.1 모든 클래스가 정의해야 하는 클래스

#### toString()

```kotlin
class Client(val name: String, val postalCode: Int) {
    // override fun toString() = "Client(name=$name, postalCode=$postalCode)"
}
fun main(args: Array<String>) {
    val client1 = Client("Alice", 342562)
    println(client1)
}
// Client@28a418fc
// Client(name=Alice, postalCode=342562)

```

#### equals() & hashCode()

자바에서는 equals를 오버라이드 할 때 반드시 hashCode도 함께 오버라이드 해야 한다.

jvm언어에서는 equals() 가 true를 반환하는 두 객체는 반드시 같은 hashCode를 반환해야 한다.

hashSet은 hashCode()를 먼저 비교하고 그 다음에야 equals()를 비교한다.

```kotlin
package ch04.ex3_1_2_2_ObjectEqualityEquals1

class Client(val name: String, val postalCode: Int) {
    override fun equals(other: Any?): Boolean {
        if (other == null || other !is Client)
            return false
        return name == other.name &&
               postalCode == other.postalCode
    }
    override fun toString() = "Client(name=$name, postalCode=$postalCode)"
    override fun hashCode(): Int = name.hashCode() * 31 + postalCode
}

fun main(args: Array<String>) {
    val client1 = Client("Alice", 342562)
    val client2 = Client("Alice", 342562)
    println(client1 == client2) // true
  
    val processed = hashSetOf(Client("Alice", 342562)
    println(processed.contains(Client("Alice", 342562)))
}
```



### 4.3.2 데이터 클래스: 모든 클래스가 정의해야 하는 메소드 자동 생성

data 키워드와 함께 클래스를 data class로 만들면  toString, equals, hashCode가 자동으로 생성된다.

```kotlin
data class Client(val name: String, val postalCode: Int)

fun main(args: Array<String>) {
    val client1 = Client("Alice", 342562)
    val client2 = Client("Alice", 342562)
    println(client1 == client2) // true
    
    val processed = hashSetOf(Client("Alice", 342562))
    println(processed.contains(Client("Alice", 342562))) // true
}
```

#### 데이터 클래스와 불변성: copy() 메소드

데이터 클래스의 프로퍼티는 val, var 둘 다 가능하지만 val을 사용하여 불변(immutable) 클래스로 만들 것을 권장한다.

### 4.3.3 클래스 위임 : by 키워드

대규모 객체지향 시스템을 설계할 때 문제는  implementation inheritance에 의해 발생한다. 하위 클래스가 상위 클래스의 메소드 중 일부를 오버라이드 하면 하위 클래스는 상위 클래스의 세부 구현 사항에 의존하게 된다. 이 상황에서 상위 클래스에 변경이 생기면 하위 클래스의 가정이 깨져 하위 클래스가 정상적으로 동작하지 않는 경우가 생길 수 있다.

이 문제를 완화해 보고자 코틀린은 기본적으로 클래스가 final이고, open 키워드로 열어 둔 클래스만 상속할 수 있다.

final 클래스에 기능을 추가하고 싶을 때는 decorator 패턴을 사용한다. decorator class가 final class를 has a 관계로 사용하고 기능을 확장한다.

```kotlin
import java.util.HashSet

class DelegatingCollection<T>: Collection<T> {
  private val innerList = arrayListOf<T>()
  
  override val size: Int get() = innerList.size
  override fun inEmpty(): Bollean = innerList.isEmpty()
  override fun contains(element: T): Boolean = innerList.contains(element)
  override fun iterator(): Iterator<T> = innerList.iterator()
  override fun containAll(elements: Collection<T>): Boolean =
  	innerList.containsAl(elements)
}

///

class CountingSet<T>(
        val innerSet: MutableCollection<T> = HashSet<T>()
) : MutableCollection<T> by innerSet {

    var objectsAdded = 0

    override fun add(element: T): Boolean {
        objectsAdded++
        return innerSet.add(element)
    }

    override fun addAll(c: Collection<T>): Boolean {
        objectsAdded += c.size
        return innerSet.addAll(c)
    }
}

fun main(args: Array<String>) {
    val cset = CountingSet<Int>()
    cset.addAll(listOf(1, 1, 2))
    println("${cset.objectsAdded} objects were added, ${cset.size} remain")
}
```

## 4.4 object 키워드: 클래스 선언과 인스턴스 생성

object 키워드를 사용하는 상황들

- 객체 선언(object declaration)은 싱글턴을 정의하는 방법 중 하나다.
- 동반 객체(companion object)는 인스턴스 메소드는 아니지만 어떤 클래스와 관련 있는 메소드와 팩토리 메소드를 담을 때 쓰인다.
  동반 객체 메소드에 접근할 때는 동반 객체가 포함된 클래스의 이름을 사용할 수 있다.
- 객체 식은 자바의 무명 내부 클래스 대신 쓰인다.

### 4.4.1 객체 선언(object declaration): 싱글턴을 쉽게 만들기 - class 대신 object 키워드를 쓰겠다.

코틀린은 객체 선언 기능을 통해 싱글턴을 언어에서 기본 지원한다.

object declaration은 클래스를 정의하고 그 클래스의 인스턴스를 만들어서 변수에 저장하는 모든 작업을 단 한 문장으로 처리한다.

생성자는 사용할 수 없다.

객체 선언문이 있는 그 자리에서 바로 만들어진다.

```kotlin
object Payroll  { // object declaration
  val allEmployees = arrayListOf<Person>()
  fun calculateSalary() {
    /**/
  }
}
// 클래
Payroll.allEmployees.add() // . 을 통해 property, method에 접근
Payroll.calculateSalary()
```

클래스 안에서 object declaration를 사용할 수 있다. 이 object도 하나의 instance를 가진다.

```kotlin
import java.util.Comparator

data class Person(val name: String) {
    object NameComparator : Comparator<Person> { // NameComparator instance는 단 하나!
        override fun compare(p1: Person, p2: Person): Int =
            p1.name.compareTo(p2.name)
    }
}

fun main(args: Array<String>) {
    val persons = listOf(Person("Bob"), Person("Alice"))
    println(persons.sortedWith(Person.NameComparator))
}
```

- singleton이 적절하지 않은 경우 : 시스템을 구현하는 다양한 구성 요소와 상호작용하는 대규모 컴포넌트에는 싱글턴이 적합하지 않다. 객체 생성을 제어할 방법이 없고 생성자 파라미터를 지정할 수 없어서다. -> 클러스터 환경 또는 클라우드 환경과 같은 상황을 말하고 있는 것 같다. load balencing이 일어나 여러개의 micro service가 동작한다면 다른 micro service instance 내부의 객체들을 제어할 방법이 없을 테니까.

### 4.4.2 동반 객체 : 팩토리 메소드와 정적 멤버가 들어갈 장소

코틀린 클래스 안에는 정적인 멤버가 없다. 코틀린은 자바의 static 키워드를 지원하지 않는다.

클래스 내부의 object에 companion 키워드를 사용하면 그 object를 동반 객체(companion object)로 만들 수 있다.

동반 객체는 자신을 둘러싼 클래스의 모든 private 멤버에 접근할 수 있다. 하나의 클래스 안에는 하나의 companion object가 있을 수 있다.

```kotlin
class A {
    companion object {
        fun bar() {
            println("Companion object called")
        }
    }
}

fun main(args: Array<String>) {
    A.bar() // companion object에 접근하기 위해서는 A라는 outer class에서 바로 접근이 가능하다.
}
```

companion obect가 private 멤버에 접근할 수 있으므로, factory pattern을 적용할 때 유용하게 사용될 수 있다.

팩토리 메소드는 목적에 따라 팩토리 메소드의 이름을 정할 수 있어서 좋다.

companion object는 하위 클래스에서 override 할 수 없다.

```kotlin
fun getFacebookName(accountId: Int) = "fb:$accountId"

class User private constructor(val nickname: String) {
    companion object { // object에 이름이 없다. 자동으로 Companion이라는 이름을 가진다.
        fun newSubscribingUser(email: String) =
            User(email.substringBefore('@'))

        fun newFacebookUser(accountId: Int) =
            User(getFacebookName(accountId))
    }
}

fun main(args: Array<String>) {
    val subscribingUser = User.newSubscribingUser("bob@gmail.com")
    val facebookUser = User.newFacebookUser(4)
    println(subscribingUser.nickname)
}
```

### 4.4.3 동반 객체를 일반 객체처럼 사용

동반 객체에 이름을 붙이거나, 동반 객체가 인터페이스를 상속하거나, 동반 객체 안에 확장 함수와 프로퍼티를 정의할 수 있다.

```kotlin
class Person(val name: String) {
	companion object Loader { // object에 이름이 있다.
    fun fromJSON(jsonText: String): Person = ...
  }	
}
person = Person.Loader.fromJSON("{name: 'a'}") // 둘 다 가능
person = Person.fromJSON("{name: 'b'}")
```

#### 동반 객체에서 인터페이스 구현

```kotlin
interface JSONFactory<T> {
  fun fromJSON(jsonText: String): T
}
class Person(val name: String) {
  companion object: JSONFactory<Person> {
    override fun fromJSON(jsonText: String): Person = //
  }
}
//
fun loadFromJSON<T>(factory: JSONFactory<T>): T { // JSONFactory interface를 인자로 받는다.
  ...
}
loadFromJSON(Person) // JSONFactory 인터페이스를 구현한 것은 Person.Companion 이지만 Person으로 넘길 수 있다.
```

#### 동반 객체에 대한 확장 함수 정의 // java static

```kotlin
class Person(val firstName: String, val lastName: String) {
	companion object {} // 빈 object라도 반드시 필요하다.
}
fun Person.Companion.fromJSON(json: String): Person {
  //
}
```



### 4.4.4 객체 식(object expression): 무명 내부 클래스를 다른 방식으로 작성

무명 객체(anonymous object)를 정의할 때도 object 키워드를 쓴다.

```kotlin
window.addMouseListener(
  object : MouseAdapter() { // MouseAdapter class를 확장하는 anonymous object
    override fun mouseClicked(e: MouseEvent) {
      //
    }
    override fun mouseEntered(e: MouseEvent) {
      //
    }
  }
  val listener = object : MouseAdapter2() { // 이렇게 이름을 줄 수 있다.
    override fun mouseClicked(e: MouseEvent) {
    }
    override fun mouseEntered(e: MouseEvent) {
    }
  }
)
```



## 4.5  요약

- 코틀린 인터페이스는 default implement을 가질 수 있다. 프로퍼티도 가질 수 있다.
- 모든 코틀린 선언은 기본적으로 final이며 public이다.
- open으로 상속, 오버라이딩이 가능하게 만들 수 있다.
- internal 선언은 같은 모듈 안에서만 볼 수 있다.
- nested class는 outer class를 참조할 수 없다. inner 키워드를 사용하여 inner class로 만들어야 outer class를 참조할 수 있다.
- sealed 클래스를 상속하고 싶으면 부모 클래스 정의 안에 중첩 클래스로 정의해야 한다.
- init 초기화 블록, 부 생성자를 활용해 클래스 인스턴스를 더 유연하게 초기화할 수 있다.
- field 식별자를 통해 프로퍼티 접근자 안에서 프로퍼티의 데이터를 저장하는데 쓰이는 뒷받침하는 필드를 참조할 수 있다.
- 데이터 클래스를 사용하면 컴파일러가 equals, hashCode, toString, copy 등의 메소드를 자동으로 생성해준다.
- by 키워드를 통한 클래스 위임을 사용하면 위임 패턴(ex. decorator)를 사용할 때 많은 준비코드를 줄일 수 있다.
- object declaration을 사용하면 코틀린답게 싱글턴 클래스를 정의할 수 있다.
- companion object는 자바의 정적 메소드와 필드 정의를 대신한다.
- 동반 객체도 다른 객체와 마찬가지로 인터페이스를 구현할 수 있다. 외부에서 동반 객체에 대한 확장 함수와 프로퍼티를 정의할 수 있다.
- 코틀린의 객체 식은 자바의 무명 내부 클래스를 대신한다.??