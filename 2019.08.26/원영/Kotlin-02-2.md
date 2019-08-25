---
title: "Kotlin Study - 02 코틀린 기초 -2 "
date: 2019-07-28 15:26:00 
categories: kotlin study
---

# 코틀린 기초

## enum과 when
~~~
enum class Color {
  RED, ORANGE, YELLOW, GREEN, BLUE
}
~~~
* enum은 코틀린에서의 soft keyword로 class 키워드 앞에 붙어야만이 enum class로 만들수 있고, 그 외에는 변수명으로 사용가능
* enum안에서도 프로퍼티와 메소드를 정의 할 수 있음
  ~~~
  enum class Color (
    val: r: Int, val g:Int, val b:Int
  ) {
    RED(255,0,0), GREEN(0,255,0), BLUE(0,0,255);  <- ; 이 반드시 들어가야함에 주의
    
    fun rgb() = (r *256 + g) * 256 + b
  }
  ~~~
  * enum 상수 목록과 메소드 정의 사이에 반드시 ; 이 들어간다
  
* java의 switch 문에 해당하는것이 코틀린의 when임
~~~
fun getColor(c : Color) = 
  when(c) {
    Color.RED -> "RED"
    Color.GREEN -> "GREEN"
    Color.Blue -> "BLUE"
  }
~~~
* java와 달리 break 문을 사용하지 않음
* 한 분기 안에서 여러값을 매치 패턴으로 사용 할 경우 , 로 분리하여 사용
  ~~~
  Color.RED, Color.GREEN, Color.BLUE -> "RGB"
  ~~~
* 상수값을 import 하면 enum 상수 이름만으로 가능하다
  ~~~
  import Color.*
  fun getColor(c : Color) = 
  when(c) {
    RED -> "RED"
    GREEN -> "GREEN"
    Blue -> "BLUE"
  }
  ~~~
* 코틀린의 when은 java와 달리 조건문에서 상수이외의 임의의 객체를 허용한다
  ~~~
  fun mix(c1: Color, c2: Color) = 
    when (setOf(c1,c2)) {
      setOf(RED,GREEN) -> "BLACK"
      else -> "Dirty"
    }
  ~~~
  - Set객체로 만드는 setOf 함수
  - 모든 조건을 만족하지 못할경우 else 문이 실행된다
* 인자가 없는 when문을 만들수도 있다
~~~
fun mix(c1: Color, c2: Color) =
  when {
    (c1 == RED && c2 == GREEN ) -> "BLACK"
  }
~~~
  - 인자가 없는 when식을 만들 경우, 각 분기의 조건문이 Boolean 결과를 계산하는 식이여야 한다

## when 문에서의 스마트 캐스트
간단한 산술식을 계산하는 함수를 만들어보자. 식을 트리 구조로 저장하며 구현한다
~~~
interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr
~~~
* 식을 위해서 Expr 인터페이스를 선언 하였지만, Expr 인터페이스는 아무런 메서드를 선언하지 않고 여러타입의 식 객체를 묶는 역할만 한다
* 클래스가 구현하는 인터페이스를 지정하기 위해 : 을 사용한다
* Expr 클래스를 구현하는 두가지 클래스가 있다
  - 어떤 Expr이 수라면 그 값을 반환해야 한다 (Num)
  - 어떤 Expr이 합계라면 좌항과 우항을 더한후 그 값을 반환해야 한다 (Sum)

java스타일 구현
~~~
fun eval(e: Expr): Int {
  if( e is Num ) {
    val n = e as Num
    return n.value
  }
  if( e is Sum ) {
    return eval(e.right) + eval(e.left)
  }
  throw IllegalArgumentException("Unkown")
}
~~~
* 자바의 instanceof 와 비슷한 역할을 하는 is, 하지만 is로 검사를 하며 자동으로 캐스팅 되기때문에, 추후 캐스팅을 다시 할필요가 없다
  - 이것이 스마트 캐스트
  - e가 Num 타입인지 검사를 마쳤으므로 자동으로 Num 타입이 되어, Num의 프로퍼티인 value에 캐스트 없이 접근이 가능하다
  - 이와 동일하게 Sum 타입이면 left, right에 캐스트 없이 접근이 가능하다
코틀린 스타일의 구현
~~~
fun eval(e: Expr) Int =
  when(e) {
    is Num -> e.value
    is Sum -> eval(e.right) + eval(e.left)
    else -> throw IllegalArgumentException("Unkown")
  }
~~~
* 블록의 마지막 식이 블록의 결과를 나타내므로, 항상 블록의 마지막에 그 분기의 결과값을위치 시켜야 함

## 이터레이션
* while 문은 java와 동일하다(while/ do while)
* for문은 java의 foreach에 해당하는 형태만 존재한다
  - 초기값, 증가값, 최종값을 사용한 루프를 대신하기 위해 범위를 사용한다
  - 코틀린에서의 범위는 양쪽 끝값을 포함하는 폐구간 이다 
  
  ~~~
  for( i in 1..100) {
    print(i)
  }
  
  1 2 3 ... 100
  ~~~
  
  - 증가값을 지정할때에는 step을 사용할 수 있다
  
    ~~~
    for( i in 1..100 step 2) <- 2씩 증가
    ~~~
    
  - 역방향으로 증가 할때에는 downTo를 사용한다
  
    ~~~
    for (i in 100 downTo 1) 
    ~~~
    
  - 끝값을 포함하지 않는 범위를 만들 때에는 until을 사용한다
  
    ~~~
    for( i in 0 until 100)
    ~~~
    
* map에 대해서도 for in을 사용하여 이터레이션을 사용 할 수 있다

~~~
val example = TreeMap<Char, String>() 
for((letter,string) in example) {
  println("$letter = $string")
}
~~~

  - 원소를 letter(key) 와 string(value)로 분해해서 표현 하였다.
  - 이를 이용하여 인덱스와 함께 컬렉션을 사용 할 수 있다
  
    ~~~
    val list = arrayListOf("1","2","3")
    for((index,element) in list.withIndex()) {
      println("$index : $element")
    }
    ~~~
    
* in 연산자를 사용해 어떤 값이 범위에 속하는지 알 수 있고, !in을 사용해 속하지 않는지를 알 수 있다

~~~
func recognize(c: Char) = when(c) {
  in '0'..'9' -> "Digit"
  in 'a'..'z' -> "Letter"
  else -> "Wrong"
}
~~~

  - 문자에만 국한되지 않고 Comparable을 구현한 클래스라면 모두 사용할 수 있다

## 코틀린의 예외처리
* throw 가 가능하며, Exception 인스턴스를 만들 때에도 new를 사용하지 않는다

~~~
if( percentage !in 1..100) {
  throw IllegalArgumentException("unknown")
}
~~~

* try - catch - finally 는 동일하다

~~~
fun readNumber(reader: BufferedReader): Int? {
  try {
    val line = reader.readLine()
  }
  catch (e: NumberFormatExcetpion) {
    return null
  }
  finally {
    reader.close()
  }
}
~~~

* throws절이 코드에 없다. java에서는 체크 예외를 명시적으로 처리 해야 하지만, 코틀린은 체크예외를 구분하지 않는다
* try도 식으로 사용할 수 있다.
* catch이후에 계속 실행 하고 싶다면 catch 블록도 값을 만들면 된다

~~~
fun readNumber(reader: BufferedReader): {
  val number = try {
    reader.readLine()
  }
  catch (e: NumberFormatExcetpion) {
    null
  }
}
~~~

## 요약
* 함수를 정의할때에는 fun 키워드, val은 읽기전용 변수, var는 변경가능한 변수
* 문자열 템플릿을 사용하면 코드가 간결해짐. 변수이름 앞에 $를 사용하거나 ${}
* 값 객체 클래스를 간결하게 표현 할 수 있음
* if문도 식이며 값을 만들어낼수있다
* when은 switch와 비슷하지만 더욱 강력하다
* 어떤 변수의 타입을 검사하고다면 궂이 캐스팅을 하지 않아도 자동으로 캐스팅된다(스마트캐스트)
* for, while, do-while 은 java와 유사하지만 맵을 이터레이션 할때 코틀린의 for가 더욱 편하다
* 1..5 같은 식으로 범위를 만들며 in, !in 등으로 검사를 할 수 있다
* 코틀린의 예외처리는 java와 비슷 하지만, 함수가 던질 수 있는 예외를 선언하지 않아도 
