# 가변성을 제한하라

공부자료 : Effective Kotlin 1장 안정성 - 가변성을 제한하라<br><br>



코틀린은 모듈로 프로그램을 설계한다.<br>
`모듈` - 클래스, 객체, 함수, 타입 별칭(type alias), 최상위(top-level)프로퍼티 등 다양한 요소로 구성

이러한 요소는 상태(state)를 가질 수 있다.<br>
~~~kotlin 
var num = 10
var list : MutableLisdt<Int> = mutableListOf()
~~~

위 코드처럼 요소가 상태를 갖는 경우, 해당 요소의 동작은 사용 방법뿐만 아니라<br>
그 이력(history)에도 의존하게 된다.<br><br>

~~~kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val account = BankAccount()
        Log.d("account.balance : ", account.balance.toString()) // 0.0

        account.deposit(100.0)
        Log.d("account.balance : ", account.balance.toString()) // 100.0

        account.withDraw(50.0)
        Log.d("account.balance : ", account.balance.toString()) // 50.0
    }
}

class BankAccount {
    var balance = 0.0
        private set

    fun deposit(depositAmount : Double){
        balance += depositAmount
    }

    fun withDraw(withDrawAmount : Double){
        if(balance < withDrawAmount){
            throw InsufficientFunds()
        }
        balance -= withDrawAmount
    }
}

class InsufficientFunds : Exception()
~~~

BankAccount에는 계좌에 돈이 얼마나 있는지 상태를 나타낸다.<>상태를 적절하게 관리하는 것은<br>
생각보다 꽤 어렵다.<br><br>

### 01- 프로그램을 이해하고 디버그하기 힘들어진다.
상태를 갖는 부분의 관계를 이해해야하고, 상태 변경이 많아지면 추적하기 힘들다.<br>
이후에 코드를 수정하기도 힘들고, 오류를 발생시키는 경우 큰 문제가 된다.

<br>

### 02- 가변성(mutability)이 있으면, 코드의 실행을 추론하기 어렵다.
시점에 따라서 값이 달라질 수 있고, 현재 어떤 값을 가지고 있는지 알아야<br>
코드의 실행을 예측할 수 있다. 또한 확인한 값이 동일하게 유지된다고 확신이 불가능하다.

<br>

### 03- 멀티스레드 프로그램일 때는 적절한 동기화가 필요하다.
변경이 일어나는 모든 부분에서 충돌이 발생할 수 있다.

<br>

### 04- 테스트하기 어렵다.
모든 상태를 테스트해야 하므로, 변경이 많을수록 더 많은 조합을 테스트해야한다.

<br>

### 05- 상태 변경이 일어날 때, 다른 부분에 알려야하는 경우가 있다.
예를들어 정렬되어 있는 리스트에 가변 요소를 추가한다면,<br>
요소에 변경이 일어날 때 마다 리스트 전체를 정렬해야한다.

<br><br>

가변성은 시스템의 상태를 나타내기 위한 중요한 방법이다.<br>
하지만 번경이 일어나야 하는 부분을 신중하고 확실하게 결정하고 사용해야한다.


