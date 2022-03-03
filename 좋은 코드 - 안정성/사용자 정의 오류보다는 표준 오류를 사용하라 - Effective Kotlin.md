# 사용자 정의 오류보다는 표준 오류를 사용하라

공부자료 : Effective Kotlin 1장 안정성 - 사용자 정의 오류보다는 표준 오류를 사용하라
<br><br>

`require`, `check`, `assert` 함수를 사용하면, 대부분의 코틀린 오류를 처리할 수 있다.<br>
하지만 이외에도 예측하지 못한 상황을 나타내야 하는 경우가 있다. json 형식을 파싱하는 라이브러리를 구현한다고 가정해보면<br>
기본적으로 입력된 json파일의 형석에 문제가 있다면, `JSONParsingException`등을 발생시키는 것이 좋다.<br>

~~~kotlin
inline fun <reified T> String.readObject(): T {
    // ..
    if(incorrectSign) {
        throw JsonParsingException()
    }
    // ..
    return result
}
~~~
표준 라이버르리에는 이를 나타내는 적절한 오류가 없어서, 사용자 정의 오류를 사용했다.<br>
하지만 가능하다면, 직접 오류를 정의하는 것보단 최대한 표준 라이브러리의 오류를 사용하는 것이 좋다.<br>
표준 라이브러리의 오류는 많은 개발자가 알고 있으므로, 재사용 하는 것이 좋다.<br>
잘 만들어진 규약을 가진 널리 알려진 요소를 재사용하면, 다른 사람들이 API를 더 쉽게 배우고 이해할 수 있다.
