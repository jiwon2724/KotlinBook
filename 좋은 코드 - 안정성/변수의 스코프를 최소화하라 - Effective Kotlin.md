# 변수의 스코프를 최소화하라

공부자료 : Effective Kotlin 1장 안정성 - 변수의 스코프를 최소화하라<br>


상태를 정의할 때는 변수와 프로퍼티의 스코프(범위)를 최소화하는 것이 좋다.<br>
-> 변수가 참조 가능한 범위를 좁혀라!<br>

<ul>
<li>프로퍼티보단 지역 변수를 사용하는 것이 좋다.</li>
<li>최대한 좁은 스코프를 갖게 변수를 사용해야한다.<br> 반복문 내부에서만
변수가 사용된다면, 변수를 반복문 내부에 작성하는 것이 좋다.</li>
</ul>
<br>

코틀린의 `스코프`는 기본적으로 중괄호로 만들어지며, 내부 스코프에서 외부 스코프에 있는 요소에만 접근이 가능하다.
<br>

### 변수의 스코프를 제한하는 예<br>
~~~kotlin
// 나쁜 예
var user : User
for(i in users.indices) {
    user = users[i]
    print("User at $i is $user")
}
~~~
<br>

~~~kotlin
// 조금더 좋은 예
for(i in users.indices) {
    val user = users[i]
    print("User at $i is $user")
}
~~~
<br>

~~~kotlin
// 제일 좋은 예
for((i, user) in users.withIndex()){
    print("User at $i is $user")
}
~~~
<br>

첫 번째 예에서 변수 user는 반복문 스코프 내부뿐만 아니라 외부에서도 사용 할 수 있다.<br>
반면 두 번째 예와 세 번째 예에서는 user의 스코프를 반복문 내부로 제한한다.
<br><br>
스코프를 좁게 만드는 것이 좋은 이유는 굉장히 많지만, 가장 중요한 이유는<br>
프로그램을 추적하고 관리하기 쉽기 때믄이다. 코드를 분석할 때는 어떤 시점에 어떤 요소가 있는지를 알아야 한다.
<br>
이때 요소가 많아져서 프로그램에 변경될 수 있는 부분이 많아지면, 프로그램을 이해하기가 어려워진다.<br>
-> var보다 val을 선호하는 이유
<br><br>

변수는 var, val 상관없이 변수를 정의할 때 초기화 해주는게 좋다.
~~~kotlin
// 나쁜 예
val user : Uesr
if(hasValue){
    user = getValue()
} else {
    user = User()
}
~~~

~~~kotlin
// 조금 더 좋은 예
val user = if(hasValue){
    getValue()
} else {
    User()
}
~~~

<br>
여러 프로퍼티를 한꺼번에 설정해야 하는 경우에는 구조분해 선언(destructuring declaration)을 활용하는게 좋다.

~~~kotlin
// 나쁜 예
fun updateWeather(degrees : Int){
    val description : String
    val color : Int
    if(degrees < 5){
        description = "cold"
        color = Color.BLUE
    } else if(degrees < 23){
        description = "mild"
        color = Color.YELLOW
    }
    // ...
}
~~~
~~~kotlin
// 조금 더 좋은 예
fun updateWeather(degrees : Int){
    val(description, color) = when {
        degrees < 5 -> "cold" to Color.BLUE
        degrees < 23 -> "mild" to Color.YELLOW
    }
    // ...
}
~~~

<br>

### 캡처링(capturing 포획)
람다 본문 블럭 내에서 외부 함수의 로컬 변수나 전역 변수등을 사용할 수 있다.<br>
~~~kotlin
// 소수를 구하는 알고리즘 시퀀스를 활용한 예제
// Sequence의 장점은 리스트를 반환하는 filter와 map을 각 연산의 컬렉션을 직접 사용한다.
// 중간결과를 저장하는 컬렉션이 생기지 않는다.
// 지연 계산(lazy) 컬렉션 연산이다.

val primes : Sequence<Int> = sequence {
            var numbers = generateSequence(2) { it + 1 }
            var prime : Int
            while(true){
                prime = numbers.first()
                yield(prime)
                numbers = numbers.drop(1).filter { it % prime != 0 }
            }
        }
~~~
이렇게 코드를 작성하면, 실행 결과가 이상하게 나온다.
<br><br>
그 이유는 `prime`이라는 변수를 캡처했기 때문이다. 반복문 내부에서 filter를 활용해서 prime으로<br>
나눌 수 있는 숫자를 필터링한다. 그런데 시퀀스를 활용하므로 필터링이 지연된다.

<br>

### 결론
잠재적인 캡처링을 주의하고, 가변성을 피하면서 스코프 범위를 좁게 만들자!<br>
여러가지 이유로 변수의 스코프는 좁게 만들어서 활용하고 var보단 val을 사용하기!<br>
람다에서 변수를 캡처하는거 기억하기!
