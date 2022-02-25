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
사용자가 코드를 제대로 사용할 거라고 믿고 있는 것보다 항상 문제를 예측하고, 문제 상황에 예외를 던지는것이 좋다.
