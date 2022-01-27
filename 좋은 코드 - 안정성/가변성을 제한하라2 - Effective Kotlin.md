# 코틀린에서 가변성 제한하기

코틀린은 가변성을 제한할 수 있게 설계되어있다. 그래서 불변(immutable)객체를 만들거나<br>
프로퍼티를 변경할 수 없게 막는 것이 쉽다.


<ul>
<li>읽기 전용 프로퍼티(val)</li>
<li>가변 컬렉션(Collection)과 읽기 전용 컬렉션 구분하기</li>
<li>data class의 copy</li>
</ul>


### 읽기 전용 프로퍼티(val)
`val`을 사용해 `읽기 전용 프로퍼티`를 만들 수 있다. 이렇게 선언된 프로퍼티는<br>
마치 값(value)처럼 동작하고 일반적인 방법으론 값이 변하지 않는다.(읽고 쓰는건 var)<br>
`읽기 전용 프로퍼티`가 완전히 변경 불가능한 것은 아니다. `읽기 전용 프로퍼티`가<br>
mutable 객체를 가지고 있다면, 내부적으로 변할 수 있다.
~~~kotlin
val list = mutableListOf(1, 2, 3)
list.add(4)
~~~
`읽기 전용 프로퍼티`는 커스텀 게터로도 정의할 수 있다.<br>
~~~kotlin
class MainActivity : AppCompatActivity() {
    var name : String = "Kotlin"
    var surName : String = "Book"
    
    val fullName
        get() = "$name $surName"

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        Log.d("fullName : ", fullName) // Kotlin Book
        name = "Java"
        Log.d("fullName : ", fullName) // Java Book
    }
}
~~~
값을 추출(호출)할 때마다 커스텀 게터가 호출되므로 이러한 코드를 사용할 수 있다.

<br>
코틀린의 프로퍼티는 기본적으로 캡슐화되어 있고, 커스텀 getter, setter를 가질 수 있다.<br>
이러한 특성으로 API를 변경하거나 정의할 때 유연하다.<br>
var는 getter, setter 모두 제공하지만, val은 변경이 불가능하므로 getter만 제공한다.<br>
커스텀 getter는 스마트 캐스트를할 수 없다.
<br>
<br>

### 가변 컬렉션과 읽기 전용 컬렉션 구분하기

코틀린은 읽고 쓸 수 있는 프로퍼티와, 읽기 전용 프로퍼티로 구분된다.<br>
컬렉션 또한 읽고 쓸 수 있는 컬렉션과, 읽기 전용 컬렉션으로 구분된다.<br><br>
![image](https://user-images.githubusercontent.com/70135188/151337722-9ce50cb6-c830-4deb-973d-35273c40211c.png)
<br><br>
프로젝트를 진행할 때, 리스트를 읽기 전용으로 리턴하면, 이를 읽기 전용으로만 사용해야한다.<br>
이는 단순히 계약의 문제이다. 하지만 컬렉션에서 다운캐스팅은(ex list를 -> mutableList로)<br>
위 계약문제를 위반하고, 추상화를 무시하는 행위이다.<br>
만약 읽기전용 컬렉션을 mutable컬렉션으로 변경해야 한다면, copy를 통해서<br>
새로운 mutable 컬렉션을 만드는 메서드를 사용해야한다.(toMutableList)
<br><br>

### 데이터 클래스의 copy
내부적인 상태를 변경하지 않는 immutable를 사용하는 이유는 다음과 같은 장점이 있다.

<ol>
    <li>한 번 정의된 상태가 유지되므로, 코드를 이해하기 쉽다.</li>
    <li>공유했을 때도 충돌이 일어나지 않아서 병렬 처리를 안전하게 할 수 있다.</li>
    <li>객체에 대한 참조는 변경되지 않으므로, 쉽게 캐시가 가능하다.</li>
    <li>다른객체(mutable 또는 immutable 객체)를 만들 때 활용하기 좋다. 그리고 실행을 쉽게 예측할 수 있다.</li>
</ol>
<br>

~~~kotlin
val names : SortedSet<FullName> = TreeSet()
val person = FullName("AAA", "AAA")
names.add(person)
names.add(FullName("Jordan", "Hansen"))
names.add(FullName("David", "Blanc"))

print(names) // [AAA AAA, David Blanc, Jordan Hansen]
print(person in names) // true

person.name = "ZZZ"
print(names) // [ZZZ AAA, David Blanc, Jordan Hansen]
print(person in names) // false
~~~
<br>

마지막 출력은 해당 객체가 names안에 있음에도 false를 리턴한다.<br>
그 이유는 객체를 변경했기 때문에 찾을 수 없다. mutable 객체는 예측하기 어렵고<br>
위험하다는 단점이 있다.<br><br>

반면, immutable 객체는 변경할 수 없다는 단점이 있다. 따라서 immutable 객체는<br>
자신의 일부를 수정하는 새로운 객체를 만들어 내는 메서드를 가져야한다.<br>

~~~kotlin
val num : Int = 3
num.plus(4)
// 7 
~~~
Int는 내부적으로 plus와 minus 메서드로 자신을 수정한 새로운 Int를 리턴할 수 있다.<br><br>

우리가 직접 만드는 immutable 객체도 비슷한 형태로 작동해야한다.<br>
~~~kotlin
class User(val name : String, val surname : String){
     fun withSurname(surname : String) = User(name, surname)
}
    
var user = User("Maja", "test")
user = user.withSurname("sur")
// User(name="Maja", surname="sur")
~~~
다만 모든 프로퍼티 대상으로 함수를 하나하나 만드는 것은 귀찮은일이다.<br>
이럴 땐 data 키워드를 사용하면 된다. data 키워드는 copy라는 이름의 메서드를 만들어준다.<br>
copy 메서드를 사용하면 모든 기본 생성자 프로퍼티가 같은 새로운 객체를 만들어 낼 수 있다.<br>

~~~kotlin
data class User(val name : String, val surname : String)

var user = User("Maja", "test")
user = user.copy(surname = "sur")
// User(name="Maja", surname="sur")
~~~


