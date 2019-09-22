---
layout: post
title: "05. lambda programming2"
date: 2019-09-24 23:31:28 -0400
categories: [kotlin]
tags: [kotlin, lambda]

---

## 5.3 지연 계산(lazy) 컬렉션 연산

map, filter 와 같은 컬렉션 함수는 결과 컬렉션을 즉시(eagerly) 생성한다.

```kotlin
// map 결과 컬렉션을 생성하고, filter 결과 컬렉션을 생성한다.
// 컬렉션을 2번 생성하게 된다.
people.map(Person::name).filter { it.startsWith("A") }  
```

이러한 eagerly 연산은 컬렉션의 원소가 많아지면 많아질수록 효율이 떨어지게 된다.

연산이 컬렉션을 직접 사용하는 대신 시퀸스를 사용하게 만들어야 한다.

```kotlin
// 위의 예제와 결과는 같지만 중간 컬렉션을 생성하지 않아 성능이 좋아진다.
people.asSequence()
	.map(Person::name)
	.filter { it.startsWith("A") }
	.toList()
```

Sequence 안에는 iterator라는 단 하나의 메소드가 있다. 이 메소드를 통해 시퀸스로부터 원소 값을 얻을 수 있다. 시퀸스의 원소는 필요할 때 비로소 계산되기 때문에, 중간 처리 결과를 저장하지 않고도 연산을 연쇄적으로 적용해서 효율적으로 계산을 수행할 수 있다.

toList 로 시퀸스를 다시 리스트로 바꾸는 이유는 index 접근 등의 다른 작업을 사용하기 위해서이다.

### 5.3.1 시퀸스 연산 실행 : 중간 연산과 최종 연산

시퀸스에 대한 연산은 중간(intermediate)연산과 최종(terminal) 연산으로 나뉜다.

중간 연산은 시퀸스를 반환한다.

최종 연산은 결과를 반환한다.

![](https://dpzbhybb2pdcj.cloudfront.net/jemerov/Figures/05fig07.jpg)

```kotlin
>>> listOf(1, 2, 3, 4).asSequence()
...         .map { print("map($it) "); it * it }
...         .filter { print("filter($it) "); it % 2 == 0 }
...         .toList()
map(1) filter(1) map(2) filter(4) map(3) filter(9) map(4) filter(16) // !! 연산순서를 잘 봐라
```



연산에 따라, 결과가 얻어지면 그 이후의 연소에 대해서는 변환이 아뤄지지 않을 수도 있다.

```kotlin
>>> println(listOf(1, 2, 3, 4).asSequence()
                              .map { it * it }.find { it > 3 })
4
```

![Figure 5.8. Eager evaluation runs each operation on the entire collection; lazy evaluation processes elements one by one.](https://dpzbhybb2pdcj.cloudfront.net/jemerov/Figures/05fig08.jpg)



연산에 따라, 각 연산의 순서가 성능에 영향을 미칠수도 있다.

```kotlin
>>> val people = listOf(Person("Alice", 29), Person("Bob", 31),
...        Person("Charles", 31), Person("Dan", 21))
>>> println(people.asSequence().map(Person::name)   
...         .filter { it.length < 4 }.toList())
[Bob, Dan]
>>> println(people.asSequence().filter { it.name.length < 4 }
...         .map(Person::name).toList())           
[Bob, Dan]
```



![Figure 5.9. Applying filter first helps to reduce the total number of transformations.](https://dpzbhybb2pdcj.cloudfront.net/jemerov/Figures/05fig09_alt.jpg)



- 자바 스트림과 코틀린 시퀸스 : 자바 8을 사용하면 자바 스트림 기능을 사용할 수 있다.



### 5.3.2 시퀸스 만들기

.asSequence() 말고도 generateSequence 함수를 사용하여 시퀸스를 만들 수 있다.

```
val natrualNumbers = generateSequence(0) {it + 1}
val numbersTo100 = naturalNumbers.takeWhile { it <= 100 }
println(numbersTo100.sum())
5050
```



```kotlin
// generateSequence
// https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/generate-sequence.html
fun <T : Any> generateSequence(
    nextFunction: () -> T?
): Sequence<T>
//
fun <T : Any> generateSequence(
    seed: T?,
    nextFunction: (T) -> T?
): Sequence<T>
```



## 5.4 자바 함수형 인터페이스 활용

자바 8 이전의 자바에서는 (람다 없을 때)  메소드에게 특정 기능을 인자로 넘기기 위해 무명 클래스의 인스턴스를 만들어야만 했다.

``` kotlin
button.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View v) {
        ...
    }
}
```

코틀린에서는 무명 클래스 인스턴스 대신 람다를 넘길 수 있다.

```kotlin
button.setOnClickListener { view -> ... }
```



onClick 이라는 명시적인 이름의 메소드를 넘겨주는 것이 아니라 람다를 넘길 수 있는 이유는  OnClickListener에 추상 메소드가 단 하나만 있기 때문이다. 그런 인터페이스를 함수형 인터페이스(Functional Interface) 또는 SAM(single abstract method)라고 한다.

ex.  Runnable이나, Callable과 같은 함수형 인터페이스와 그런 함수형 인터페이스를 확용하는 메소드가 많다.



### 5.4.1 자바 메소드에 람다를 인자로 전달

함수형 인터페이스를 인자로 원하는 자바 메소드에 코틀린 람다를 전달할 수 있다.

예를 들어 다음과 같이 Runnable 타입의 파라미터를 받는 메소가 있을 떄

```kotlin
void postponeComputation(int delay, Runnable computation);
```

코틀린에서 람다를 이 함수에 넘길 수 있다.

컴파일러는 자동으로 람다를 Runnable을 구현한 무명 클래스로 변환시켜준다.

```kotlin
postponeComputation(1000) { println(43) }
// Runnable을 구현하는 무명 객체를 명시적으로 만들어서 사용할 수도 있다.
postponeComputation(1000, object : Runnable {
  override fun run() {
    println(42)
  }
}
// 이렇게 명시적으로 객체를 생성하면 메소드를 호출할 때 마다 새로운 객체가 생성된다.
// 람다는 무명 객체를 반복 사용한다.
```



```kotlin
postponeComputation(1000) { println(43) } // 프로그램에서 단 하나의 Runnable 객체 생성


val runnable = Runnable { println(42) } // Runnable SAM 생성자
fun handleComputation() {
    postponeComputation(1000, runnable) // 모든 handleComputation 호출에 같은 객체 사용
}


fun handleComputation(id: String) { // 람다 안에서 id 변수 포획            
    postponeComputation(1000) { println(id) } // 매 호출 마다 새 Runnable 인스턴스를 만든다.
}
```



람다를 무명 클래스로 만들고 그 클래스의 인스턴스를 만들어서 메소드에 넘긴다는 설명은 함수형 인터페이스를 받는 자바 메소드를 코틀린에서 호출할 때 쓰는 방식을 설명해주지만, 컬렉션을 확장한 메소드에 람다를 넘기는 경우 코틀린은 그런 방식을 사용하지 않는다. 코틀린 inline 으로 표시된 코틀린 함수에게 람다를 넘기면 아무런 무명 클래스도 만들어지지 않는다.

람다와 자바 함수형 인터페이스 사이의 변환은 자동으로 이뤄진다.

하지만 이 둘 사이의 변환을 수동으로 해야하는 경우 어떻게 해야 하는지 알아보자



### 5.4.2 SAM 생성자 : 람다를 함수형 인터페이스로 명시적으로 변경

SAM 생성자는 람다를 함수형 인터페이스의 인스턴스로 변환할 수 있게 컴파일러가 자동으로 생성한 함수다.

```kotlin
fun createAllDoneRunnable(): Runnable {
    return Runnable { println("All done!") } // Runnable이 SAM 생성자
}

>>> createAllDoneRunnable().run()
All done!
```



```kotlin
val listener = OnClickListener { view -> // SAM 생성자로 생성한뒤, 변수에 저장 가능
    val text = when (view.id) {             
        R.id.button1 -> "First button"
        R.id.button2 -> "Second button"
        else -> "Unknown button"
    }
    toast(text)                      
}
button1.setOnClickListener(listener) // listener 인스턴스 재사용하기
button2.setOnClickListener(listener)
```



- this 키워드는 람다에서는 사용할 수 없고 무명 객체에서는 사용할 수 있다.



## 5.5 수신 객체 지정 람다 : with & apply

코틀린은 수신 객체를 명시하지 않고 람다의 본문 안에서 다른 객체의 메소드를 호출할 수 있게 한다.

그런 람다를 **수신 객체 지정 람다(lambda with receiver)** 라고 부른다.

### 5.5.1 with 함수

- with 없이.

```kotlin
fun alphabet(): String {
    val result = StringBuilder()
    for (letter in 'A'..'Z') {
         result.append(letter)
    }
    result.append("\nNow I know the alphabet!")
    return result.toString()
}
>>> println(alphabet())
ABCDEFGHIJKLMNOPQRSTUVWXYZ
Now I know the alphabet!
```

- with with

```kotlin
fun alphabet(): String {
    val stringBuilder = StringBuilder()
    return with(stringBuilder) { // 메소드를 호출하려는 수신 객체 지정
        for (letter in 'A'..'Z') {
            this.append(letter) // 수신 객체 메소드 호출
        }
        append("\nNow I know the alphabet!")  // this 생략
        this.toString() // 람다에서 값을 반환
    }
}
// 리팩토링으로 stringBuilder 변수 없애기
fun alphabet() = with(StringBuilder()) {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
    toString()
}
```

with은 파라미터가 2개 있는 함수다. 1 - StringBuilder, 2 - lambda

with 함수는 첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 수신 객체로 만든다. 인자로 받은 람다 본문에서는 this를 사용해 그 수신 객체에 접근할 수 있다. 일반적인 this와 마찬가지로 this와 .을 사용하지 않고 프로퍼티나 메소드 이름만 사용해도 수신 객체의 멤버에 접근할 수 있다.

### 5.5.2 apply 함수

apply 함수는 거의 with과 같다. 유일한 차이는 apply는 항상 자신에게 전달된 객체를 반환한다는 점 뿐이다.

alphabet을 apply를 이용하여 리팩토링 해보자.

```kotlin
fun alphabet() = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
}.toString() // apply가 StringBuilder 객체를 반환하므로 toString을 사용할 수 있다.
```

apply는 확장 함수로 정의돼 있다.

이런 apply 함수는 객체의 인스턴스를 만들면서 즉시 프로퍼티 중 일부를 초기화해야 하는 경우 유용하다. 자바에서는 보통 별도의 Builder 객체가 이런 역활을 담당한다. 코틀린에서는 어떤 클래스가 정의돼 있는 라이브러리의 특별한 지원 없이도 그 클래스 인스턴스에 대해 apply를 활용할 수 있다.



```kotlin
fun createViewWithCustomAttributes(context: Context) =
    TextView(context).apply {
        text = "Sample Text"
        textSize = 20.0
        setPadding(10, 0, 0, 0)
    }
```

