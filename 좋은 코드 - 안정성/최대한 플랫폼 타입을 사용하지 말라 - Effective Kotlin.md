# 최대한 플랫폼 타입을 사용하지 말라
공부자료 : Effective Kotlin 1장 안정성 - 최대한 플랫폼 타입을 사용하지 말라
<br><br>
`플랫폼 타입(platform type)`이란?<br>
-> 다른 프로그래밍 언어에서 잔달되어 nullable인지 아닌지 알 수 없는 타입
<br><br>
널 안정성(null-safety)은 코틀린의 주요 기능중 하나이다. 자바에서 자주 볼 수 있던 NPE는<br>
코틀린에서 찾아보기 힘들다. 하지만 null-safety 메커니즘이 없는 자바, 다른 프로그래밍 언어와<br>
코틀린을 연결해서 사용할 땐 이러한 예외가 발생할 수 있다.
<br><br>
자바에선 모든 것이 nullable일 수 있으므로 최대한 안전하게 접근한다면<br>
nullable로 가정하고 다루어야한다. 하지만 어떤 메서드는 리턴값이 절대로 null이 아닐수가있다.<br>
이러한 경우에는 마지막에 not-null을 나타내는 `!!`를 붙인다.<br><br>
nullable과 관련하여 자주 문제가 되는 부분은 자바의 제네릭 타입이다.<br>
예를들어 List<User>를 리턴하고, 어노테이션(@Nullable 혹은 @NotNull)이 따로 붙어 있지 않은 경우를 생각해보자.<br>

코틀린이 디폴트로 모든 타입을 nullable로 다룬다면 이 객체를 사용할 때 리스트와, 리스트 내부의 User도<br>
널이 아니라는 것을 알아야 한다. 따라서 리스트 자체만 널인지 확인해선 안되고, 내부(User)에 있는 것들도 널인지 확인해야한다.
<br>
~~~kotlin
  // 자바
  public class UserRepo {
      public List<User> getUser() {
          // ...
      }
  }
  
  // 코틀린
  val users : List<User> = UserRepo.users!!.filterNotNull()
~~~
<br>
리스트는 적어도 map과 filterNotNull을 제공하지만 다른 제네릭 타입이라면 널을 확인하는 자체가 복잡한 일이다.<br>  
위 코드처럼 다른 프로그래밍 언어에서 넘어온 타입을 특수하게 다룬다.
<br><br>
    
`플랫폼 타입`을 사용할 땐 항상 주의를 기울여야 한다. 설계자가 명시적으로 어노테이션(@Nullable, @NotNull)을 표시하거나<br>
주석을 달아두지 않으면, 언제든지 동작이 변경될 가능성이 있다.(null이 아니라고 생각되는 것이 null일수도 있음.)
    
~~~kotlin
    import org.jetbrains.annotations.NouNull;
    
    public class UserRepo {
      public @NotNull List<User> getUser() {
          // ...
      }
  }
~~~
    
<br>

### 정리
다른 프로그래밍 언어에서 와서 nullable 여부를 알 수 없는 타입을 `플랫폼 타입`이라고 부른다.<br>
`플랫폼 타입`을 사용하는 코드는 해당 부분만 위험할 뿐만 아니라, 이를 활용하는 곳까지 위험하다.<br>
`플랫폼 타입`을 지양하고 사용해야한다면 어노테이션을 이용하자.    
