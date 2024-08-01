---
categories: [BackEnd, Kotlin, JetBrains]
tags: [Kotlin, 변수]
---

<br>

## 기본 타입

<br>

Java와 동인한 기본 타입
- Byte
- Short
- Int ★
- Long ★
- Float ★
- Double ★
- 부호 없는 정수들

<br>

Java와 다른 내용
- Java : 기본 타입 간의 변환은 `암시적`으로 이루어 짐
- Kotlin : 기본 타입 간의 변환은 `명시적`으로 이루어 짐

<br>

Kotlin의 예시
```kotlin
val num1 = 4 // Int Type
val num2: Long = num1 // Type mismatch

println(num1 + num2)
```
위와 같이 `num1`은 `Int Type`이기 때문에 `Long`에 넣게 되면 컴파일 `Type mismatch`가 발생

<br>

해결 방법 (to 변환 타입)
```kotlin
val num1 = 3
val num2: Long = num1.toLong()

println(num1 + num2)
```
위와 같이 `toLong()`을 사용해 변환하면 해결된다.

<br>

---

<br>

## 타입 캐스팅

<br>

Java 예시
```java
  public static void printAgeIfPerson(Object obj) {
    if (obj instanceof Person) {
      Person person = (Person) obj;
      System.out.println(person.getAge());
    }
  }
```
Java에서 `instanceof`는 변수가 주어진 타입이면 `true`, 아니면 `false`

<br>

Kotlin 예시
```kotlin
fun printAgeIfPerson(obj: Any) {
    if(obj is Person){
        val person = obj as Person
        println(person.age)
    }
}
```
Kotlin에서는 `instanceof` 대신 `is`를 사용  
타입 또한 `(Person)`가 아닌, `as Person`으로 사용

<br>

형변환 생략 가능
```kotlin
fun printAgeIfPerson(obj: Any) {
    if(obj is Person){
        println(obj.age)
    }
}
```
위와 같이 `obj`를 바로 사용 가능  
- 이런한 변환을 `스마트 캐스트`라고 부른다.
  - Kotlin 컴파일러가 컨텍스트를 분석해서 `if`에서 타입 체크를 해주면 같은 타입으로 간주

<br>

형이 같지 않은 경우
```kotlin
fun printAgeIfPerson(obj: Any) {
    if(obj !is Person){
        println(obj.age)
    }
}
```
즉, `obj`가 `Person` 타입이 아닌 경우를 `!is`로 확인할 수 있다.

<br>

null이 가능한 경우
```kotlin
fun printAgeIfPerson4(obj: Any?) {
    var person = obj as? Person
    println(person?.age)
}
```
`obj`에 `null`이 들어올 수 있는 구조라면 형변환 `as`를 사용할 때 `as?`를 사용해서 `null`사용 허용
- 그렇게 되면 `person`은 `person.age`으로 가져 올 수 없고, `person?.age`와 같이 `safe call`을 사용해야 한다.
- tip).`as?`는 `obj`가 `null`이어도 `null`을 반환하고 `Person`이 아니어도 `null`을 반환한다.
  - 중요!!. 타입이 `일치하지 않아도` 예외가 아닌 `null`을 반환하기 때문에 주의 필요

<br>

---

<br>

## Kotlin의 3가지 특이한 타입

<br>

### Any
- Java의 Object 역할 (모든 객체의 최상위 타입)
- 모든 `Primitive Type`의 최상위 타입 (Java는 아님)
- Any 자체로는 null을 포함할 수 없어 `Any?`를 사용해서 null을 포함
- `equals` / `hashCode` / `toString` 존재

### Unit
- Java의 void와 동일한 역할 (타입 추론이 가능해서 생략 가능)
- void와 다르게 Unit은 그 자체로 타입 인자로 사용 가능
- 단 하나의 인스턴스만 갖는 타입을 의미 (실제 존재하는 타입)

### Nothing
- 함수가 정상적으로 끝나지 않았다는 사실을 표현
- 무조건 예외를 반환하는 함수 / 무한 루프 등.
  - 자주 사용되는 타입은 아님.
```kotlin
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}
```

<br>

---

<br>

## String Interpolation, String indexing

<br>

- 문자열에 ${변수}를 사용해서 값을 넣을 수 있다.
- 주의!! `간단한 변수를 사용할 때만 사용하는 것이 좋다`

<br>

Java의 경우
```java
    val person = Person("김용준", 29)
    System.out.println(String.format("이름 : %s", person.name))
```
Java는 `String.format`을 사용해서 출력

<br>

Kotlin의 경우
```kotlin
    val person = Person("김용준", 29)
    println("이름 : ${person.name}")
```
`${person.name}`와 같이 `${변수}`를 사용할 수 있다.

<br>

일반 적인 String의 경우
```kotlin
    val name = "김용준"
    println("이름 : $name")
```
`$`만을 사용해서 작성 가능

<br>

trimIndent 사용 예시
```kotlin
    val name = "김용준"
    val str = """
        ABC
        EFG
        ${name}
    """.trimIndent()
    println(str)
```
위와 같이 문자열에 개행하면서 자유롭게 사용할 수 있다.
- 출력 예시   
  ABC  
  EFG  
  김용준  

<br>

Java 특정 문자 가져오기 
```java
String str = "ABCDE";
char ch = str.charAt(1);
```
Java는 위와 같이 `charAt`을 사용해서 특정 문자의 index를 사용해서 특정 문자를 가져옴

<br>

Kotlin의 특정 문자 가져오기
```kotlin
val str = "ABCDE"
val ch = str[1]
```
위와 같이 배열을 이용해서 index에 해당하는 특정 문자를 가져옴





