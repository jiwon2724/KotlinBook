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


