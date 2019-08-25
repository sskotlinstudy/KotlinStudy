# 2. 코틀린 기초

## 2.1 함수와 변수

2.1.1  Hello, world!

```kotlin
fun main(args: Array<String>) {
	println("Hello, world!")
}
```

2.1.2 함수

```kotlin
fun max(a: Int, b: Int): Int {
	return if (a > b) a else b
}
println(max(1,2))
```

 다음과 같이 함수의 parameter와 return type을 지정할 수 있다.

뭔가 의식의 흐름에 맞추어 함수를 정의하도록 유도하는 듯한 느낌이다.

  