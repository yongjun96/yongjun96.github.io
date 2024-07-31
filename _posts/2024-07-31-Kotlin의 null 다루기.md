---
categories: [BackEnd, Kotlin, JetBrains]
tags: [Kotlin, 변수]
---

<br>

## Kotlin에서 null 체크

<br>

Java 코드

```java
public boolean startsWithA(String str) {
    return str.startsWith("A");
  }
```

위 코드는 안전한 코드 일까??
- 아니다!!

만약 null이 들어오면 `NullPointException (NPE)`이 발생한다.

<br>

3가지 경우로 개선 가능
```java
  public boolean startsWithA1(String str) {
    if (str == null) {
      throw new IllegalArgumentException("null이 들어왔습니다");
    }
    return str.startsWith("A");
  }


  public Boolean startsWithA2(String str) {
    if (str == null) {
      return null;
    }
    return str.startsWith("A");
  }


  public boolean startsWithA3(String str) {
    if (str == null) {
      return false;
    }
    return str.startsWith("A");
  }
```

<br>

---

<br>

Kotlin 코드
```kotlin
fun startsWithA1(str: String?): Boolean {
    if (str == null) {
        throw IllegalArgumentException("null이 들어왔습니다")
    }
    return str.startsWith("A")
}


fun startsWithA2(str: String?): Boolean? {
    if(str == null){
        return null
    }
    return str.startsWith("A")
}


fun startsWithA3(str: String?): Boolean {
    if(str == null){
        return false
    }
    return str.startsWith("A")
}
```

<br>

null을 허용하지 않도록 작성할 수 있다.
```kotlin
fun startsWithA(str: String): Boolean {
    return str.startsWith("A")
}
```

<br>

---

<br>

## Safe Call과 Elvis 연산자

<br>

- Kotlin에서는 null이 가능한 타입은 완전히 다르게 취급
- null이 가능한 타입만을 위한 기능 : `Safe Call`, `Elvis`

### Safe Call

예제
```kotlin
val str: String? = "ABC"

str.length // 불가능
str?.length // 가능
```

- null이 아니면 실행
- null이면 미실행 (그대로 null)

<br>

### Elvis

예제
```kotlin
val str: String? = "ABC"
str?.length ?: 0
```

- 앞에 연산 결과가 null이면 뒤에 값을 사용
- 여담으로 `?:`을 90도 돌리면 가수 Elvis의 헤어스타일과 닮아서 Elvis연산자라고 부른다고 한다.
  <img alt="img_1.png" height="300" src="../assets/img/postimg/2024-07-31/Elvis연산자.png" width="300"/>

<br>

Safe Call과 Elvis를 사용해 수정한 예제
```kotlin
fun startsWithA1(str: String?): Boolean {
    return str?.startsWith("A") ?: throw IllegalArgumentException("null이 들어왔습니다")
}


fun startsWithA2(str: String?): Boolean? {
    return str?.startsWith("A")
}


fun startsWithA3(str: String?): Boolean {
    return str?.startsWith("A") ?: false
}
```

<br>

---

<br>

## 널 아님 단언

- nullable type이지만, null이 될 수 없는 경우
- 예를 들어 처음 생성되는 경우에는 null일 수 있지만 값이 한번 들어오면 null이 될 수 없는 경우 등...

<br>

```kotlin
fun startsWith(str: String?): Boolean {
    return str!!.startsWith("A")
}
```
절대 null이 아니라는 의미로 `!!`를 붙여서 사용

<br>

---

<br>

## 플랫폼 타입

- Kotlin에서 Java를 가져와 사용 가능
- Java코드에 @Nullable이 없거나 null 관련 정보를 알 수 없는 타입
  - `Runtime 시 Exception이 발생`할 수 있음

<br>

예제 Person
```java
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

<br>

Kotlin에서 사용
```kotlin
fun main(){

    val person = Person("개발자");
    startsWithNotNull(person.name)
}


fun startsWithNotNull(str: String): Boolean{
  return str.startsWith("A")
}
```
