---
categories: [BackEnd, Kotlin, JetBrains]
tags: [Kotlin]
---

<br>

## Static 함수와 변수
- static : 클래스가 인스턴스화 될 때 새로운 값이 복제되는게 아니라 정적으로 인스턴스 끼리의 값을 공유

### Kotlin에서의 static -> companion object
- 클래스와 동행하는 유일한 `object`
  - Kotlin에서는 `static`을 사용할 수 없는 대신, `companion object`를 사용
- 유틸성 함수들을 넣어도 되지만 최상단 파일을 활용하는 것을 추천

<br>

Java Person 예제
```java
public class JavaPerson {

  private static final int MIN_AGE = 1;

  // 정적 펙토리 메서드
  public static JavaPerson newBaby(String name) {
    return new JavaPerson(name, MIN_AGE);
  }

  private String name;

  private int age;

  private JavaPerson(String name, int age) {
    this.name = name;
    this.age = age;
  }

}
```

<br>

Kotlin Person 예제
```kotlin
class Person private constructor(
    var name: String,
    var age:Int
) {

    // Kotlin 은 static 이 없다.
    // 대신 동행 객체(companion object)를 사용
    // val MIN_AGE = 1 에 0이라는 값이 런타임 시에 할당
    // val MIN_AGE = 1 에 const 를 붙이게 되면 컴파일 시에 변수 할당
    companion object {
        // const -> 진짜 상수 (기본타입 및 String 에만 사용)
        private const val MIN_AGE = 1
        fun newBaby(name: String): Person{
            return Person(name, MIN_AGE)
        }
    }
}
```
- `companion object`를 사용해서 Java와 같이 `static`으로 사용 가능
- `MIN_AGE`이라는 상수에 `const`를 붙여 주면 컴파일 시에 변수 할당 (진짜 상수)

<br>

### companion object가 Java의 static과 다른 점
- `companion object`는 하나의 객체로 간주 된다.
- 이름이 붙을 수도 있고, `interface`를 구현할 수도 있다.

<br>

interface 구현 예시
```kotlin
    companion object  Factory : Log{
        private const val MIN_AGE = 1
        fun newBaby(name: String): Person{
            return Person(name, MIN_AGE)
        }

        // 추상 메서드 구현
        override fun log() {
            println("log")
        }
    }
```
- `Log`라는 `interface`를 구현할 수 있다.
- 해당 `interface`의 추상 메서드도 구현해야 한다.

<br>

### companion object의 이름이 없는 경우 Java에서 사용
- Java에서 Kotlin의 객체를 사용할 때 `Person.Conpanion.newBaby("아기");`와 같이 `Conpanion`을 이름대신 사용
```java
  public static void main(String[] args) {
    Person.Conpanion.newBaby("아기");
  }
```
- 또는 Kotlin `companion object`안에 있는 메서드에 `@JvmStatic`을 붙여 줘야 한다.
```kotlin
        @JvmStatic
        fun newBaby(name: String): Person{
            return Person(name, MIN_AGE)
        }
```
```java
  public static void main(String[] args) {
    Person.newBaby("아기");
  }
```
- 이름이 있다면 이름을 사용하면 된다.
```java
  public static void main(String[] args) {
     Person.Factory.newBaby("ABC");
  }
```

<br>

---

<br>

## 싱글톤
- 단 하나의 인스턴스를 갖는 클래스
```java
public class JavaSingleton {

  private static final JavaSingleton INSTANCE = new JavaSingleton();

  private JavaSingleton() { }

  public static JavaSingleton getInstance() {
    return INSTANCE;
  }
}
```
- 동시성 처리와 enum 클래스를 활용하는 방법도 있다.

<br>

Kotlin의 싱글톤
```kotlin
object Singleton
```
- `object`만 붙여주면 된다.

간단한 예제
```kotlin
fun main() {

  println(Singleton.a)
  Singleton.a += 10
  println(Singleton.a)

}

// 싱클톤 클래스
// 싱글톤은 어차피 인스턴스가 하나이기 때문에 인스턴스화를 하지 않는다.
object Singleton {
    var a: Int = 0
}
```
출력 결과
- 0
- 10


<br>

---

<br>

## 익명 클래스
- 특정 `interface`나 `class`를 상속받는 구현체를 `일회성`으로 사용할 때 쓰는 클래스

<br>

Java의 익명 클래스
```java
public static void main(String[] args) {
    moveSomething(new Movable() {
        @Override
        public void move() { System.out.println("움직임"); }
  
        @Override
        public void fly() { System.out.println("난다"); }
    })
  }

private static void moveSomething(Movable movable) {
    movable.move();
    movable.fly();
  }
```

<br>

Kotlin 익명 클래스
```kotlin
fun main(){

    moveSomething(object : Movable {
        override fun move() {
            println("움직임")
        }

        override fun fly() {
            println("난다")
        }
    })
}

private fun moveSomething(movable: Movable) {
    movable.move()
    movable.fly()
}
```

### Java와의 차이점
- new Movable() / object : Movable
- @Override / override fun move()
