---
title: "Kotlin Study - 05 람다로 프로그래밍 -2"
date: 2019-08-07 23:16:00 
categories: kotlin study
---

# 람다로 프로그래밍

## 지연 계산(lazy) 컬렉션 연산
* map이나 filter 같은 컬렉션 함수는 결과 컬렉션을 즉시 생성한다
* 즉 이러한 컬렉션 함수를 중첩 한다면 매 단계마다 계산 중간 결과를 새로운 컬렉션에 임시로 담기 때문에 효율이 떨어진다
* 이럴 경우 각 연산이 컬렉션을 직접 사용 하는 대신 시퀀스를 사용하게 만들어야 한다
* 코틀린의 지연 계산 시퀀스는 Sequence 인터페이스에서 시작 하며, 그 안에는 iterator 메소드 단 하나만이 존재 한다
* 시퀀스의 원소는 필요할때 비로소 계산 되기 때문에, 효율적인 계산이 가능하다
* asSequence 확장 함수를 호출 하면 어떤 컬렉션이든 시퀀스로 바꿀 수 있다. 

### 시퀀스 연산 실행: 중간 연산과 최종 연산
~~~
listOf(1,2,3,4).asSequence()
  .map { print("map($it) "); it * it }
  .filter { print("filter($it) "); it % 2 == 0 }
  .toList()

=> map(1) filter(1) map(2) filter(4) map(3) filter(9) map(4) filter(16)
~~~

* toList 라는 최종 연산이 없을 경우에 해당 코드를 실행하면 아무것도 출력 되지 않는다 <= 계산을 하지 않기 때문이다
* 원소의 계산 순서를 눈여겨 보라. 하나의 원소에 대해 순차적으로 연산이 실행됨을 알 수 있다
* 따라서 원소에 연산을 차례대로 적용하다가 결과가 얻어지면 그 이후의 원소에는 계산이 안 이뤄질 수 있다
* 이를 이용 하여 filter 등을 다른 중간 연산보다 먼저 실행 한다면 더욱 효율 적으로 할 수 있다

### 시퀀스 만들기
~~~
val naturalNumbers = generateSequence(0) { it + 1 }
val numbersTo100 = naturalNumbers.takeWhile { it <= 100 }
println(numbersTo100.sum())

=> 5050
~~~

* asSequence 이외에 generateSeuqnce를 사용 하여 시퀀스를 만들 수 있다
* naturalNumbers, numbersTo100 모두 시퀀스 이며, 최종 연산인 sum을 수행 하기 전까지는 계산이 되지 않는다
* 시퀀스를 사용하는 일반적인 용례중 하나는 객체의 조상으로 이뤄진 시퀀스를 만들어 내는 것이다

## java 함수형 인터페이스 활용
* 코틀린 람다는 java API에 사용해도 아무런 문제가 없다
* 추상 메소드가 단 하나만 있는 함수형 인터페이스 혹은 SAM(Single Abstract Method) 인터페이스를 인자로 취하는 java 메소드를 호출할때 람다를 쓸 수있다

### java 메소드에 람다를 인자로 전달
~~~
void postponeComputation( int delay, Runnable computation);

postponeComputation(1000) {println(42)}

postponeComputation(1000, object: Runnable {
  override fun run() {
    println(42)
  }
})

fun handleComputation(id: String) {
  postponeComputation(1000) {println(id)}
}
~~~

* Runnable이 대표적인 java의 SAM 인터페이스다. 따라서 이러한 변환은 자동으로 이뤄진다
* 따라서 위의 두 코드는 같은 코드이다
* 객체를 명시적으로 선언하는 경우 메소드 호출 시마다 새 객체가 생성 되지만, 람다의 경우 무명 객체를 메소드 호출때마다 반복하여 사용한다
* 변수 포획이 가능한 코틀린 람다의 특성을 이용해 변수를 포획한다면, 매 호출마다 새로운 인스턴스가 생성 될 것이다

### SAM 생성자: 람다를 함수형 인터페이스로 명시적으로 변경
~~~ 
val listener = OnClickListener { view ->
  val text = when (view.id) {
    R.id.button1 -> "First button"
    R.id.button2 -> "Second button"
    else -> "Other button"
  }
  toast(text)
}

button1.setOnclickListener(listener)
button2.setOnclickListener(listener)
~~~

* 함수형 인터페이스의 인스턴스를 반환하는 메소드가 있다면 람다를 직접 반환 할 수 없고, 해당 람다를 SAM 생성자로 감싸야 한다
* SAM 생성자의 이름은 사용 하려는 함수형 인터페이스의 이름과 같다
* 람다로 생성한 함수형 인터페이스의 인스턴스르 변수에 저장하는 경우에도 SAM 생성자를 이용 할 수 있다
* 위의 코드가 그러한 특성을 사용해 하나의 리스너를 여러 버튼에 적용 한 것이다
* 람다로 만든 리스너는, 람다로 만들어진 무명 클래스의 인스턴스를 참조할 방법이 없으므로, 만약 리스너 등록 해제가 필요 할 경우는 람다를 쓸 수 없다


## 수신 객체 지정 람다: with와 apply

### with 함수
~~~
fun alphabet(): String {
  val stringBuilder = StringBuilder()
  return with(stringBuilder) {
    for( letter in 'A'..'Z' ) {
      this.append(letter)
    }
    toString()
  }
}
~~~

* 어떤 객체의 이름을 반복하지 않고 그 객체에 다양한 연산을 할 수 있게 해주는 것이 with이다
* with뒤에 수신 객체를 지정 한 후, this를 통해 해당 객체를 부르고 있다
* 이러한 this는 생략이 가능 하다
* 하지만 같은 이름의 메서드가 존재 할때에는 this를 적절히 명시하여주는것이 좋다
* with는 사실 파라미터로 stringBuilder와 람다를 받은 것이며, 람다를 괄호 밖으로 빼낸 것이다

### apply함수
~~~
fun alphabet() = StringBuilder().apply {
  for( letter in 'A'..'Z' ) {
      this.append(letter)
  }
}.toString()
~~~

* apply는 거의 with와 비슷 하지만, apply는 항상 자기 자신에게 전달된 객체인 수신 객체를 반환한다
* 이러한 apply함수는 객체의 인스턴스를 만들면서 즉시 프로퍼티중 일부를 초기화 해야 하는 경우 유용하다
* with와 apply같은 수신 객체 지정 람다는 DSL(Domain Specific Language), 영역 특화 언어를 만들때 매우 유용하다

## 요약
* 람다를 사용하면 코드 조각을 다른 함수에 인자로 넘길 수 있다
* 코들린에서는 람다가 함수 인자인 경우 괄호 밖으로 빼고 람다를 빼낼 수 있다
* 람다의 인자가 단 하나 뿐인 경우 인자의 이름을 지정 하지않고 it 라는 디폴트 이름으로 사용 할 수 있다
* 람다 안에 있는 코드는 그 람다가 들어 있는 바깥 함수의 변수를 읽거나 쓸 수 있다
* 메소드, 생성자, 프로퍼티 이름 앞에 :: 을 붙이면 각각에 대한 참조를 만들 수 있다. 그러한 참조를 람다 대신 다른 함수에 넘길 수 있다
* filter, map, all, any 등의 함수를 활용 하면 컬렉션에 대한 대부분의 연산을 직접 원소를 이터레이션 하지 않고 수행 할 수 있다
* 시퀀스를 사용하면 중간 결과를 담는 컬렉션을 생성하지 않고도 컬렉션에 대한 여러 조합을 할 수 있다
* 함수형 인터페이스 (SAM 인터페이스)를 인자로 받는 java 함수를 호출 할 경우 직접 람다를 함수형 인터페이스 인자 대신 넘길 수 있다
* 수신 객체 지정 람다를 사용하면, 람다 안에서 미리 정해준 수신 객체의 메소드를 쓸 수 있다
* 표준 라이브러리의 with함수로 어떤 객체에 대한 참조를 반복하지 않고 그 객체의 메소드를 호출 할 수 있다.
* apply는 어떤 객체라도 빌더 스타일의 api를 이용해 생성하고 초기화 할 수 있다
    
