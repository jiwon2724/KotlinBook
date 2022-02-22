# inferred 타입(타입 추론)으로 리턴하지 말라
공부자료 : Effective Kotlin 1장 안정성 - nferred 타입으로 리턴하지 말라<br><br>

타입 추론을 사용할 때 몇 가지 위험한 부분들이 있다.<br>
우선 할당 때 inferred 타입은 오른쪽에 있는 피연산자에 맞게 설정된다.<br>
-> 슈퍼클래스 또는 인터페이스로는 설정되지 않는다.
<br>

~~~kotlin
open class Animal
class Zebra : Animal()

fun main(){
    var animal = Zebra()
    animal = Animal() // 오류 : Type mismatch
}
~~~

일반적인 경우에는 문제가 되지 않지만, 원하는 타입보다 제한된 타입이 설정되었다면<br>
타입을 명시적으로 지정해서 문제를 해결할 수 있다.

~~~kotlin
fun main(){
    var animal : Animal = Zebra()
    animal = Animal() 
}
~~~
하지만 직접 라이브러리(또는 모듈)을 조작할 수 없는 경우엔 이런 문제를 간단히 해결할 수 없다.<br>
그리고 이런 경우 inferred 타입을 노출하면 위험한 일이 발생할 수 있다.

~~~kotlin
// 다음과 같은 CarFactory 인터페이스가 있다고 가정하고
interface CarFactory {
    fun produce() : Car
}
~~~

~~~kotlin
// 아무것도 지정하지 않았을 경우 생성되는 디폴트 자동차
val DEFAULT_CAR : Car = Fiat126P()
~~~

코드를 작성하다 보니 DEFAULT_CAR는 Car로 명시적으로 지정되어 있으므로 따로 필요 없다고 판단해<br>
함수의 리턴 타입을 제거했다고 가정한 상황이다.

~~~kotlin
interface CarFactory {
    fun produce() = DEFAULT_CAR
}
~~~

그런데 이후에 다른 개발자가 코드를 보다가 DEFAULT_CAR는 타입 추론에 의해 자동으로 타입이 지정될 것이므로<br>
Car를 명시적으로 지정하지 않아도 된다고 생각해서 다음과 같이 코드를 변경했다.

~~~kotlin
val DEFAULT_CAR = Fiat126P()
~~~

이제 문제가 발생했다. CarFactory에서는 이제 Fiat126P 이외의 자동차를 생산하지 못한다.<br>
만약 인터페이스를 직접 만들었다면, 쉽게 문제를 찾아서 수정할 수 있지만, 외부 API라면 쉽게 해결하지 못한다.<br>
리턴 타입은 외부에서 확인할 수 있게 명시적으로 지정해 주는 것이 좋다.
<br><br>

### 정리
타입을 확실하게 지정해야 하는 경우 명시적으로 타입을 지정해야 한다는 원칙만 가지고 있으면 된다.<br>
이는 굉장히 중요한 정보이므로, 숨기지 않는 것이 좋다. 외부 API를 만들 땐 반드시 타입을 지정해야한다.<br>
inferred 타입은 프로젝트가 진전될 때, 제한이 많아지거나 예측하지 못한 결과를 낼 수 있다.
