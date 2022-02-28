# 예외를 활용해 코드에 제한을 걸어라

공부자료 : Effective Kotlin 1장 안정성 - 예외를 활용해 코드에 제한을 걸어라
<br><br>
확실하게 어떤 형태로 동작해야 하는 코드가 있다면, 예외를 활용해 제한을 걸어주는 것이 좋다.<br>
코틀린에선 코드의 동작에 제한을 걸 때 다음과 같은 방법을 사용할 수 있다.<br><br>

<ul>
  <li>require 블록 : 아규먼트(argument)를 제한할 수 있다.</li>
  <li>check 블록 : 상태와 관련된 동작을 제한할 수 있다.</li>
  <li>assert 블록 : 어떤 것이 true인지 확인할 수 있다. assert 블록은 테스트 모드에서만 작동한다.</li>
  <li>return 또는 throw와 함께 활용하는 Elvis 연산자</li>
</ul>

코드의 동작에 제한을 걸은 예
~~~kotlin
// Stack<T>의 일부
fun pop(num: Int = 1) : List<T> {
    require(num <= size) { "Cannot remove more elements than current size" }
    // require 블록이 실행되면 예외를 던짐!
    check(isOpen) { "Cannot pop from closed stack" }
    val ret = collection.take(num)
    collection = collection.drop(num)
    assert(ret.size == num)
    return ret
}
~~~
<br>

#### 제한을 걸어줬을 때 장점
<ul>
  <li>제한을 걸면 문서를 읽지 않은 개발자도 문제를 확인할 수 있다.</li>
  <li>문제가 있을 경우 함수가 예상하지 못한 동작을 하지 않고 예외를 throw한다.<br>
  예상하지 못한 동작을 하는 것은 예외를 throw하는 것보다 굉장히 위험하며 상태관리가 힘들다.<br>
  이러한 제한으로 문제를 놓치지 않을 수 있고, 코드가 더 안정적으로 작동한다.</li>
  <li>코드가 어느 정도 자채적으로 검사된다. 따라서 이와 관련된 단위 테스트를 줄일 수 있다.</li>
  <li>스마트 캐스트 기능을 활용할 수 있게 되므로, 캐스팅을 적게 할 수 있다.</li>
</ul>
<br>

### 아규먼트
함수를 정의할 때 타입 시스템을 활용해서 아규먼트(argument)에 제한을 거는 코드를 많이 사용한다.<br><br>
예시
<ul>
  <li>숫자를 아규먼트로 받아서 팩토리얼을 계산하면 수자는 양의 정수여야 한다.</li>
  <li>좌표들을 아규먼트로 받아서 클러스터를 찾을 땐 비어 있지 않은 좌표 목록이 필요하다.</li>
  <li>사용자로부터 이메일 주소를 입력받을 땐 값이 입력되어 있는지, 이메일 형식이 올바른지 확인해야 한다.</li>
</ul>
일반적으로 
이러한 제한을 걸 땐 require 함수를 사용한다. require 함수는 제한을 확인하고<br>
제한을 만족하지 못할 경우 예외를 throw한다.
<br>

~~~kotlin
fun factorial(n: Int): Long {
    require(n >= 0)
    return if (n <= 1) 1 else factorial(n-1) * n
}

fun findClusters(points: List<Point>) : List<Cluster> {
    require(points.isNotEmpty())
    // ...
}

fun sendEmail(user: User, message: String){
    requireNotNull(user.email)
    require(isValidEmail(user.email))
    // ...
}
~~~
이와 같은 형태의 유효성 검사 코드는 함수의 가장 앞부분에 배치되므로, 읽는 사람도 쉽게 확인이 가능하다.<br>
`require`함수는 조건을 만족하지 못할 때 무조건 `IllegalArgumentException`을 발생시키므로 제한을 무시 못한다.
<br><br>

### 상태
어떤 구체적인 조건을 만족할 때만 함수를 사용할 수 있게 해야 할 때가 있다. 예를들면
<ul>
    <li>어떤 객체가 미리 초기화되어 있어야만 처리를 하게 하고 싶은 함수</li>
    <li>사용자가 로그인했을 때만 처리를 하게 하고 싶은 함수</li>
    <li>객체를 사용할 수 있는 시점에 사용하고 싶은 함수</li>
</ul>

상태와 관련된 제한을 걸 때는 일반적으로 `check` 함수를 사용한다.<br>

~~~kotlin
fun speak(text: String){
    check(isInitialized)
    // ...
}

fun getUserInfo(): UserInfo {
    checkNotNull(token)
    // ...
}

fun next(): T {
    check(isOpen)
    // ...
}
~~~

`check`함수는 `require`와 비슷하지만, 지정된 예측을 만족하지 못할 때 예외(IllegalArgumentException)를 던진다.<br>
상태가 올바른지 확인할 때 사용한다. `check`와 `require`의 함수 내부를 살펴보니 코드는 똑같았다.<br>
함수 전체에 대한 어떤 예측이 있을 땐 일반적으로 `require`블록 뒤에 배치한다. `check`를 나중에 해아한다.<br>
`check`를 사용하는 이유는 사용자가 규약을 어기고, 사용하면 안 되는 곳에서 함수를 호출하고 있다고 의심될 때 한다.<br>
사용자가 코드를 제대로 사용할 거라고 믿고 있는 것보다 항상 문제를 예측하고, 문제 상황에 예외를 던지는것이 좋다.<br><br>

### Assert 계열 함수 사용
스스로 구현한 내용을 확인할 땐 일반적으로 `assert`함수를 사용한다.<br>
함수가 올바르게 구현되었다면, 확실하게 참을 낼 수 았는 코드들이 있다. 예를들면<br>
어떤 함수가 10개의 요소를 리턴한다면, "함수가 10개의 요소를 리턴하는가?"라는 코드는 참이다.<br>
그런데 함수가 올바르게 구현되어 있지 않을 수도 있다. 구현을 잘못했거나, 다른 개발자가 함수를 수정해서<br>
제대로 작동하지 않을수도 있다. 이러한 구현 문제로 발생할 수 있는 문제를 예방하려 단위 테스트를 사용하는 것이 좋다.

~~~kotlin
class StackTest {
    @Test
    fun `Stack pops correct number of elements`() {
        val stack = Stack(20){ it }
        val ret = stack.pop(10)
        assertEquals(10, ret.size)
    }
    
    fun pop(num: Int = 1) : List<T> {
        assert(ret.size == num) // 모든 pop 호출 위치에서 테스트
        return ret
    }
}
~~~
단위 테스트는 구현의 정확성을 확인하는 가장 기본적인 방법이다.
현재 코드에선 스택이 10개의 요소를 pop하면 10개의 요소가 나온다는 사실을 테스트하고 있다.<br>
다만 프로덕션 환경에선 오류가 발생하지 않는다.(테스트때만 활성화) 오류가 발생해도 사용자가 알아차릴 수 없다.<br>
만약 이 코드가 심각한 오류고, 나쁜 결과를 초래할 수 있는 경우엔 `check`를 사용하는 것이 좋다.<br>
단위 테스트 대신 함수에서 `assert`를 사용하면, 다음과 같은 장점이 있다.<br>

<ul>
    <li>Assert 계열의 함수는 코드를 자체 점검하고, 더 효율적으로 테스트할 수 있다.</li>
    <li>특정 상황이 아닌 모든 상황에 대한 테스트를 할 수 있다.</li>
    <li>실행 시점에 정확하게 어떻게 되는지 확인할 수 있다.</li>
    <li>실제 코드가 더 빠른 시점에 실패하게 만든다. 따라서 예상하지 못한 동작이 언제 어디서 실행되었는지 쉽게 찾을 수 있다.</li>
</ul>  
참고로 이를 활용해도 여전히 단위테스트는 따로 작성해야 한다. 애플리케이션 실행시 assert는 예외를 던지지 않는다.
<br><br>

### nullabilty와 스마트 캐스팅
`require`와 `check`로 어떤 조건을 확인해서 true가 나왔다면, 핻강 조건은 이후로도 true라고 가정한다.<br>
~~~kotlin
public inline fun require(value: Boolean) : Unit {
    contract {
        returns() implies value
    }
    require(value) {"Failed requirement."}
}
~~~
따라서 이를 활용해 타입을 비교했다면, 스마트 캐스트가 작동한다.<br>
다음 예는 어떤 사람(person)의 복장(person.outfit)의 드레스(Dress)여야 코드가 정상적으로 진행된다.<br>
따라서 만약 이러한 outfit 프로퍼티가 final이라면, outfit 프로퍼티가 Dress로 스마트 캐스트된다.<br>

~~~kotlin
fun changeDress(person: Person) {
    require(person.outfit is Dress)
    val dress: Dress = person.outfit
}
~~~

이러한 특징은 어떤 대상이 null인지 확인할 때 굉장히 유용하다.

~~~kotlin
class Person(val email: String?)

fun sendMail(person: Person, message: String) {
    require(person.email != null)
    val email: String = person.email
}
~~~
이런 경우 requireNotNull, checkNotNull이라는 특수한 함수를 사용해도 괜찮고 둘 다 스마트 캐스트를 지원한다.<br><br>
nullability를 목적으로, 오른쪽에 throw 또는 return을 두고 Elvis 연산자를 활용하는 경우가 많다.<br>
이런 코드는 굉장히 읽기 쉽고, 유연하게 사용할 수 있다. 첫 번째로 오른쪽에 return을 넣으면, 오류를 발생시키지 않고 단순하게 함수를<br>
중지할 수도 있다.
~~~kotlin
fun sendEmail(person: Person, text: String){
    val email: String = person.email ?: return
}
~~~
프로퍼티에 문제가 있어서 null일 때 여러 처리를 해야 할 때도, return/throw와 run 함수를 조합해서 활용하면 됩니다.<br>
이는 함수가 중지된 이유를 로그에 출력해야 할 때 사용할 수 있다.
~~~kotlin
fun sendEmail(person: Person, text: String){
    val email: String = person.email ?: run {
        log("Email not sent, no email address")
        return
    }
}
~~~
이처럼 return과 throw를 활용한 Elvis 연산자는 nullable을 확일할 때 굉장히 많이 사용되는 관용적인 방법이다.<br>
또한 이런 코드는 함수 앞부분에 넣어서 잘 보이게 만드는 것이 좋다.<br><br>

### 정리

<ul>
  <li>제한을 훨씬 더 쉽게 확인할 수 있다.</li>
  <li>애플리케이션을 더 안정적으로 지킬 수 있다.</li>
  <li>코드를 잘못 쓰는 상황을 막을 수 있다.</li>
  <li>스마트 캐스팅을 활용할 수 있다.</li>
  <li>require : 아규먼트와 관련된 예측을 정의할 때 사용하는 방법</li>
  <li>check : 상태와 관련된 예측을 정의할 때 사용하는 방법</li>
  <li>assert : 테스트 모드에서 테스트를 할 때 사용하는 방법</li>
  <li>return과 throw와 함께 Elvis 연산자 사용하기</li>
</ul>
