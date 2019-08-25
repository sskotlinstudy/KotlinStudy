# 2. 코틀린 기초

## 2.1 함수와 변수

### 2.1.1  Hello, world!

```kotlin
fun main(args: Array<String>) {
	println("Hello, world!")
}
```

### 2.1.2 함수

```kotlin
// 1.
fun max(a: Int, b: Int): Int {
	return if (a > b) a else b
}
println(max(1,2))
// 2.
fun max(a: Int, b: Int): Int = if (a > b) a else b
```

 다음과 같이 함수의 parameter와 return type을 지정할 수 있다.

뭔가 의식의 흐름에 맞추어 함수를 정의하도록 유도하는 듯한 느낌이다.



statement / expression

expression 은 값을 만들어낸다.

statement는 자신을 둘러싸고 있는 가장 안쪽 블록의 최사위 요소로 존재하며 아무런 값을 만들어내지 않는다.



### 2.1.3 변수

```kotlin
val question = "삶, 우주, 그리고 모든 것에 대한 궁극적인 질문" // String 변수 선언
val answer = 42 // Int 변수 선언
val answer: Int = 42 // 명시적으로 Int type을 알려줌
val yearsToCompute = 7.5e6 // floating point
val answer: Int // 초기화 식을 사용하지 않고 변수를 사용하려면 변수 타입이 반드시 필요.
answer = 42
```

mutable / immutable variable

- val로 선언된 변수는 일단 초기화되고 나면 immutable 하다. 자바로 말하자면 final 변수에 해당한다.
- var로 선언된 변수는 mutable 하다.
- 기본적으로는 모든 변수를  val 키워드를 사용해 불변 변수로 사용하고, 나중에 꼭 필요할 때에만 var로 변경하라.

```kotlin
// val 변수에 대해 단 하나의 초기화 문장만 사용된다면, 다음과 같이 여러 값으로 초기화 할 수 있다.
val message: String
if (canPerformOperation()) {
	message = "Success"
}
else {
	message = "Failed"
}
```

```kotlin
// val 참조가 가리키는 객체의 내부 값은 변경될 수 있다.
val languages = arrayListOf("Java")
languages.add("Kotlin")
```

```kotlin
//  var 변수는 변경될 수 있지만, 변수의 타입은 변경될 수 없다.
var answer = 42
answer = "no answer"
```



### 2.1.4 문자열 템플릿

```kotlin
fun main(args: Array<String>) {
	val name = if (args.size > 0) args[0] else "Kotlin"
  println("Hello, $name!") // $를 사용해서 println가 name val을 사용하도록 할 수 있다.
  println("print dollor sign : \$") // $ 자체를 출력하고 싶으면 backslash를 사용하라.
  if (args.size > 0) {
    println("Hello, ${args[0]}!") // {} 를 사용하면 복잡한 식도 문자열 템플릿에서 사용할 수 있다.
  }
}

fun main(args: Array<String>) {
  println("Hello, ${if (args.size > 0) args[0] else "someone"}!") // {}안에서 "" 를 사용할 수 있다.
}
```

## 2.2 클래스와 프로퍼티

```java
// name이라는 하나의 프로퍼티만 가지고 있다.
// 프로퍼티가 추가되면 Person 생성자도 변경되어야한다.
public class Person {
  private final String name;
  public Person(String name) {
    this.name = name;
  }
  
  public String getName() {
    return name;
  }
}
```

```kotlin
// 위의 자바로 작성한 Person을 kotlin으로 작성하면 다음과 같다.
// 다음과 같이 코드 없이 데이터만 저장하는 클래스를 data object라 부른다.
// kotlin default visibility modifier는 public
class Person(val name: String)
```

### 2.2.1 프로퍼티

```kotlin
class Person {
  val name: String, // 읽기 전용 프로퍼티, getter가 생성된다.
  var isMarried: Boolean // 읽기/쓰기 프로퍼티, getter, setter가 생성된다.
}
```

```java
// kotlin class를 java에서 사용할 때 다음과 같이 getter, setter에 접근할 수 있다.
>> Person person = new Person("Bob", true);
>> System.out.println(person.getName()); // name property에 대해 getName getter가 자동생성
Bob
>> System.out.println(person.isMarried()); // isMarried에 대해 isMarried getter가 자동생성
true
```

```kotlin
// kotlin code
>> val person = Person("Bob", true) // new 키워드 사용하지 않음.
>> println(person.name) // 프로퍼티 이름을 사용하면 getter가 호출됨.
Bob
>> println(person.isMarried)
true
>> person.isMarried = false // 값을 변경하고 싶으면 그냥 넣으면 setter가 호출됨
```

### 2.2.2 커스텀 접근자 - custom getter

직사각형 클래스를 정의하고 정사각형인지 확인하는 기능을 만들어보자.

```kotlin
class Rectangle(val height: Int, val width: Int) {
  val isSquare: Boolean
  	get() {
    	return height == width
  	}
}
```

### 2.2.3 코틀린 소스코드 구조: 디렉터리와 패키지

자바의 경우 모든 클래스를 패키지 단위로 관리한다

코틀린에도 비슷한 개념의 패키지가 있다.

```kotlin
package geometry.shapes // 이 파일의 패키지를 선언한다. 이 파일안의 모든 선언이 이 패키지에 들어간다.
import java.util.Random // 외부 패키지의 사용을 선언한다.
class Rectangle(val height: Int, val width: Int) {
  val isSquare: Boolean
  	get() = height == width
}
fun createRandomRectangle(): Rectangle {
  val random = Random()
  return Rectangle(random.nextInt(), random.nextInt())
}
```



![java dir tree](http://tutorials.jenkov.com/images/java/packages-1.png)

자바에서는 패키지 파일들이 해당 패키지 이름과 같은 폴더 아래 들어있어야 한다.

코틀린은 이런 제약이 없지만 자바와 같이 사용하는 환경을 생각하면 이 룰을 따르는 게 좋다.

### 2.3 enum, when

2.3.1 enum class 정의

```kotlin
// kotlin 에서는 다음과 같이 enum class를 만든다.
enum class Color { // enum은 class keyword 앞에서만 특별한 의미를 갖는다.
	RED, ORANGE, YELLOW, GREEN
}
```

```kotlin
// enum 클래스 안에서도 다음과 같이 method를 정의할 수 있다.
enum class Color (
  val r: Int, val g: Int, val b: Int
) {
  RED(255, 0, 0), ORANGE(255, 165, 0),
  YELLOW(255, 255, 0), GREEN(0, 255, 0); // 세미콜론 필수
  
  fun rgb() = (r * 256 + g) * 256 + b
}
```



### 2.3.2 when

```kotlin
fun getMnemonic(color: Color) = // 함수가 when의 반환값을 바로 사용한다.
	when (color) { // when에서는 break를 사용하지 않아도 된다.
    Color.RED -> "Richard"
    Color.ORANGE -> "Of"
    Color.YELLOW, Color.GREEN -> "York" // 여러 값이 하나의 값으로 매칭 될 때는 , 를 사용한다.
  }
```



### 2.3.3 when과 임의의 객체를 함께 사용

```kotlin
fun mix(c1: Color, c2: Color) =
	when(setOf(c1, c2)) { // 임의의 객체를 사용하지만 하나의 객체로 만들어서 줘야하는가 싶다. 
    setOf(RED, YELLOW) -> ORANGE
    setOf(YELLOW, BLUE) -> GREEN
    setOf(BLUE, VIOLET) -> INDIGO
    else -> throw Excetpion("Dirty color") // 
  }
```

### 2.3.4 when에 인자 없이 사용

```kotlin
fun mixOptimized(c1: Color, c2: Color) =
	when {
    (c1 == RED && c2 == YELLOW) || // 각 분기 조건문이 Boolean을 계산하면 when을 인자없이 사용할 수 있다.    (c1 == YELLOW && c2 == RED) ->
    	ORANGE
    ...
  }
```



2.3.5 스마트 캐스트: 타입 검사 + 타입 캐스트

 ```kotlin
interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr

fun eval(e: Expr): Int =
    when (e) {
        is Num -> // type 검사
            e.value // Num type으로 smart cast 되었다.
        is Sum ->
            eval(e.right) + eval(e.left)
        else ->
            throw IllegalArgumentException("Unknown expression")
    }
 ```



## 2.4 iteration : while, for

### 2.4.1 while

```kotlin
while (condition) {
}

do {
} while (condition)
```

### 2.4.2 수에 대한 이터레이션 : 범위와 수열

코틀린에는 c와 같은 for 는 없다.  for_each와 같은 녀석이 있을 뿐.

c와 같은 for의 기능은 range를 사용한다.

range는 기본적으로 두 값으로 이뤄진 구간이다. 두 값은 정수 등의 숫자 타입의 값이다. .. 연산자로 시작, 끝을 정의한다.

```kotlin
val oneToTen = 1..10 // 1,2,3,4,5,6,7,8,9,10 - 코틀린 range는 폐구간이다.
```

```kotlin
fun fizzBuzz(i: Int) = when {
    i % 15 == 0 -> "FizzBuzz "
    i % 3 == 0 -> "Fizz "
    i % 5 == 0 -> "Buzz "
    else -> "$i "
}

fun main(args: Array<String>) {
    for (i in 1..100) { // 1 부터 100 까지 반복
        print(fizzBuzz(i))
    }
    for (i in 100 downTo 1 step 2) { // 100 부터 1까지 2씩 감소
        print(fizzBuzz(i))
    }
}
```

2.4.3 맵에 대한 이터레이션

```kotlin
import java.util.TreeMap

fun main(args: Array<String>) {
    val binaryReps = TreeMap<Char, String>()

    for (c in 'A'..'F' step 2) { // char range도 있구나. step 가능함
        val binary = Integer.toBinaryString(c.toInt())
        binaryReps[c] = binary
    }

    for ((letter, binary) in binaryReps) { // binaryReps.withIndex() 는 없는듯
        println("$letter = $binary")
    }
}
```

array iteration with index

```kotlin
val list = arrayListOf("10", "11", "1001")
for ((index, element) in list.withIndex()) {
    println("$index: $element")
}
```

2.4.4 in 으로 요소 포함 검사

```kotlin
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
fun isNotDigit(c: Char) = c !in '0'..'9'

fun main(args: Array<String>) {
    println(isLetter('q'))
    println(isNotDigit('x'))
}

////////////

fun recognize(c: Char) = when (c) {
    in '0'..'9' -> "It's a digit!"
    in 'a'..'z', in 'A'..'Z' -> "It's a letter!"
    else -> "I don't know…"
}

fun main(args: Array<String>) {
    println(recognize('8'))
}

////////////

println("Kotlin" in "Java".."Scala") // 사전 순서상 위치 비교
true
```



### 2.5.1 try, cat, finally

```kotlin
fun readNumber(reader: BufferedReader): Int? { // ? 는 nullable 표시이다. int 또는 null값을 리턴할 수 있다. IOException이 처리되지 않았지만 명시할 필요는 없다.
    try {
        val line = reader.readLine()
        return Integer.parseInt(line)
    }
    catch (e: NumberFormatException) {
        return null
    }
    finally {
        reader.close()
    }
}

fun main(args: Array<String>) {
    val reader = BufferedReader(StringReader("239"))
    println(readNumber(reader))
}
```

