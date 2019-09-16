
# 5. 람다로 프로그래밍

lambda expression 또는 **람다**는 기본적으로 다른 함수에 넘길 수 있는 작은 코드 조각을 뜻한다. // small anonymous function으로 생각하면 될듯

람다를 사용하면 쉽게 공통 코드 구조를 라이브러리 함수로 뽑아낼 수 있다.

코틀린 표준 라이브러리에서는 람다를 아주 많이 사용한다.

## 5.1 람다 식과 멤버 참조

### 5.1.1 람다 소개: 코드 불록을 함수 인자로 넘기기

```java
// java - onClick event가 발생했을 때 수행할 로직을 anonymous class로 넘겨주고 있다.
button.setOnClickListener(new onClickListener() { 
  @Override
  public void onClick(View view) {...}
});
```

```kotlin
// kotlin - 로직을 lambda를 이용해서 로직을 넘겨주고 있다.
button.setOnClickListener { ... }
```

#### 5.1.2 람다와 컬렉션

코드에 중복을 제거하는 것은 좋다. 컬렉션을 다룰 때 수행하는 대부분의 작업은 몇 가지 일반적인 패턴(sort, maxBy, etc..)에 속한다.

하지만 람다가 없다면 컬렉션을 편리하게 처리할 수 있는 좋은 라이브러리를 제공하기 힘들다. - 코드와 함께 이해해보자.

```kotlin
data class Person(val name: String, val age: Int) // name, age를 저장하는 Person class
```

```kotlin
fun findTheOldest(people: List<Person>) { // 컬렉션 직접 뒤지기
  var maxAge = 0
  var theOldest: Person? = null
  for (person in people) {
    if (person.age > maxAge) {
      maxAge = person.age
      theOldest = person
    }
  }
  println(theOldest)
}
```

```kotlin
val people = listOf(Person("Alice", 29), Person("Bob", 31)) // 람다로 구현하기
println(people.maxBy{it.age})
// BoBob!
```

{ it.age } 는 바로 비교해 사용할 값을 돌려주는 함수다. iterator 가 가르키고 있는 녀석(it)의 age를 반환한다. 

### 5.1.3 람다 식의 문법

![람다 식 문법](https://miro.medium.com/max/1008/1*8vUOmuLdabVl7uPHKlwbBA.jpeg)

```kotlin
val sum = { x: Int, y: Int -> x + y} // 람다를 변수에 저장할 수 있다.
println(sum(1,2)) // 변수가 함수인 것 처럼 사용할 수 있다.
> 3
println({ x: Int, y: Int -> x + y}(1, 2)) // 람다를 직접 호출할 수 있다. 사용성 떨어짐 
> 3
run { println(42) } // 람다를 직접 호출할 일이 있으면 run을 사용하자.
> 42
```



```kotlin
// lambda as a parameter
// 1.
people.maxBy({ p: Person -> p.age})
//2. 
people.maxBy(){ p: Person -> p.age} // 함수의 제일 마지막 인자가 람다라면 람다를 밖으로 뺄 수 있다.
//3. 
people.maxBy{ p: Person -> p.age } // 람다가 함수의 유일한 인자라면 2.에서 괄호를 없앨 수 있다.
```



```kotlin
// 람다 파라미터 타입 제거하기
people.maxBy{ p: Person -> p.age }
people.maxBy{ p -> p.age } 
people.maxBy{ it.age } // it 키워드는 편하지만 남발하지는 말자.
// 컴파일러가 람타 파라미터의 타입을 추론할 수 있다.
// 추론할 수 없는 경우도 있지만 나중에 배우도록 하자.
```



```kotlin
val sum = { x: Int, y: Int ->
	println("Computing the sum of $x and $y...")
	x + y // 긴 람다에서 본문의 맨 마지막에 있는 식이 람다의 결과 값이 된다.
}
println(sum(1,2))
```



### 5.1.4 현재 영역에 있는 변수에 접근

```kotlin
fun printProblemCounts(responses: Collenction<String>) {
  var clientErrors = 0
  var serverErrors = 0
  responses.forEach { // forEach 문에 lambda를 넘겨준다.
    if(it.startsWith("4")) {
      clientErrors++ // lambda 안에서 printProblemCounts 함수의 property에 접근이 가능하다.
    } else if(it.startsWith("5")) {
      serverErrors++
    }
  }
  println("$clientErrors client errors, $serverErrors server errors")
}
val responses = listOf("200 OK", "418 i'm a teapot", "500 Internal Server Error")
printProblemCounts(responses)
> 1 client errors, 1 server errors
```



```
람다를 실행 시점에 표현하는 데이터 구조는 람다에서 시작하는 모든 참조가 포함된 닫힘(closed) 객체 그래프를 함다 코드와 함께 저장해야 한다. 그런 데이터 구조를 클로저(closure)라고 부른다. 함수를 쓸모 있는 1급 시민으로 만들려면 포획한 변수를 제대로 처리해야 하고, 포획한 변수를 제대로 처리하려면 클로저가 꼭 필요하다. 그래서 람다를 클로저라고 부르기도 한다. 람다, 무명 함수, 함수 리터럴, 클로저를 서로 혼용하는 일이 많다. ?
```



- 함수 안에 정의된 로컬 변수의 생명주기는 함수가 반환되면 끝난다. 하지만 어떤 함수가 자신의 로컬 변수를 포획한 람다를 반환하거나 다른 변수에 저장한다면 로컬 변수의 생명주기와 함수의 생명주기가 달라질 수 있다.
- 파이널 변수를 포획한 경우에는 람다코드를 변수 값과 함께 저장한다. 파이널이 아닌 변수를 포획한 경우에는 변수를 특별한 래퍼로 감싸서 나중에 변경하거나 읽을 수 있게 한 다음, 래퍼에 대한 참조를 람다 코드와 함께 저장한다.

///

### 5.1.5 멤버 참조

코틀린에서는 자바8과 마찬가지로 함수를 값으로 바꿀 수 있다. 이때 이중 콜론(::)을 사용한다.

::를 사용하는 식을 멤버 참조(member reference)라고 부른다. member reference는 프로퍼티나 메소드를 단 하나만 호출하는 함수 값을 만들어준다.

```kotlin
val getAge = Person::age
```



다음의 예시 코드들은 모두 같은 코드이다.

```kotlin
people.maxBy(Person::age)
people.maxBy{ p -> p.age }
people.maxBy{ it.age }
```



최상위에 선언된, 다른 클래스의 멤버가 아닌 함수나 프로퍼티를 member reference 할 수 있다.

```kotlin
fun salute() = println("Salute!")
>>> run(::salute) // 최상위 함수를 참조한다.
>>> Salute!
```



++ 여러가지  member reference 사용법

```kotlin
// 1.
val action = { person: Person, message: String -> // 인자를 받아서 sendEmail함수에게 작업을 위임하는 람다.
		sendEmail(person, message)
}
val nextAction = ::sendEmail // 람다 말고 member reference를 사용하면 편하다.
// 2. 생성자 참조
data class Person(val name: String, val age: Int)

val createPerson = ::Person // 생성자를 참조하는 createPerson을 만든다.
val p = createPerson("Alics", 29)
println(p)
// 3. 확장함수 참조
fun Person.isAdult() = age >= 21
val predicate = Person::isAdult
```



## 5.2 컬렉션 함수형 API

### 5.2.1 filter, map

filter, map은 컬렉션을 활용할 때 기반이 되는 함수다.

- filter : filter 함수는 컬렉션을 이터레이션 하면서 주어진 람다에 각 원소를 넘겨서 람다가 true를 반환하는 원소만 모은다.

```kotlin
val = listOf(1,2,3,4)
println(list.filter{it % 2 == 0})
[2, 4]
println(list)
[1, 2, 3, 4]
//
val list = listOf(1, 2, 3, 4)
val filteredList = list.filter { it % 2 == 0}
println(list.filter { it % 2 == 0 })
[2, 4]
println(filteredList)
[2, 4]
```



- map : map 함수는 주어진 람다를 컬렉션의 각 원소에 적용한 결과를 모아서 새 컬렉션을 만든다.

```kotlin
val list = listOf(1,2,3,4)
println(list.map{it * it})
[1, 4, 9, 16]
```

이런 호출을 쉽게 연쇄시킬 수 있다.

```kotlin
people.filter{it.age > 30}.map(Person::name)
```



filter, map과 같은 함수를 map collection에 적용할 수도 있다.

```
val numbers = mapOf(0 to "zero", 1 to "one")
println(numbers.mapValues{it.value.toUpperCase()})
{0=ZERO, 1=ONE}
```

맵의 경우 filterKeys, mapKeys / filterValues, mapValues로 값을 걸러 내거나 변환한다.

// key, value 둘 다 변환하고 싶은 경우는 없나?

### 5.2.2 all, any, count, find : 컬렉션에 술어 적용

컬렉션에 대해 자주 수행하는 연산으로 컬렉션의 모든 원소가 어떤 조건을 만족하는지 판단하는 연산이 있다.

코틀린에서는 all과 any가 이런 연산이다. count는 조건을 만족하는 원소의 갯수, find는 조건을 만족하는 첫 번째 원소를 반환한다.

```kotlin
val canBeInClub27 = {p: Person -> p.age <= 27}

val people = listOf(Person("Alice", 27), Person("Bob", 31))
println(people.all(canBeInClub27))
>> false
println(people.any(canBeInClub27))
>> true
println(people.count(canBeInClub27))
>> 1
println(people.filter(canBeInClub27).size) // BAD!! 중간 컬렉션이 생긴다.
>> 1
println(people.find(canBeInClub27)) // == firstOrNull, 단 하나!
>> Person(name=Alice, age=27)
```



### 5.2.3 groupBy: 리스트를 여러 그룹으로 이뤄진 맵으로 변경

```kotlin
val people = listOf(Person("Alice", 27), Person("Bob", 31), Person("Carol", 31))
println(people.groupBy{ it.age })
>> {31=[Person(name=Alice, age=31), Person(name=Carol, age=31)], 29=[Person(name=Bob, age=29)]}
//
val people = listOf(Person("Alice", 31), Person("Bob", 29), Person("Carol", 32))
println(people.groupBy { it.age / 10 })
>> {3=[Person(name=Alice, age=31), Person(name=Carol, age=32)], 2=[Person(name=Bob, age=29)]}
// ?! 
```



### 5.2.4  flatMap과 flatten: 중첩된 컬렉션 안의 원소 처리

flatMap  함수는 먼저 인자로 주어진 람다를 컬렉션의 모든 객체에  map하고, 람다를 적용한 결과 얻어지는 여러 리스트를 한 리스토로 한데 모은다(flatten).



```kotlin
class Book(val title: String, val authors: List<String>)

fun main(args: Array<String>) {
    val books = listOf(Book("Thursday Next", listOf("Jasper Fforde")),
                       Book("Mort", listOf("Terry Pratchett")),
                       Book("Good Omens", listOf("Terry Pratchett",
                                                 "Neil Gaiman")))
    println(books.flatMap { it.authors }.toSet())
}
```



```kotlin
val deepArray = arrayOf(
    	arrayOf(1),
    	arrayOf(2, 3),
    	arrayOf(4, 5, 6)
	)
println(deepArray)
>> [[Ljava.lang.Integer;@28a418fc
deepArray.forEach{println(it)}
>> [Ljava.lang.Integer;@5305068a
   [Ljava.lang.Integer;@1f32e575
   [Ljava.lang.Integer;@279f2327
println(deepArray.flatten()) 
>> [1, 2, 3, 4, 5, 6]
```





