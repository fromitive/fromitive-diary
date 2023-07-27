
<figure markdown>
![Image title](/fromitive-diary/assets/2023-07-27/galaxy.png){ width="300"}
<figcaption><a href="https://www.freepik.com/icon/sky_6355753#position=31&page=1&term=galxy&fromView=search">Icon by Dighital</a></figcaption>
</figure>


# 리소스 모음집

## 왜 오랫동안 다녔을까?
  - 끈기가 없어 보이기 싫어서
  - 내가하는 일에 `재미` 그리고 `행복` `보람`도 중요함
  - 돈도 당연 좋아함 여유롭고 싶음
  - 적어도 여유롭지 않으면 재미 행복 보람이라도 있어야 함

## % 의 제수(divisor)가 음수일 때 결과 값이 다르게 나온다고 한다.

python과 다르게 자바는 int끼리 나누면 소수점이 나오지 않고 잘린다.

```
(a/b) * b + (a % b) == a
``` 

## ROP (not return oriented programming but railway oriented programming)

[참고 링크](https://kciter.so/posts/railway-oriented-programming)

### Error에 대해 처리하는 대표적인 방법

#### 뛰기 전에 보라

```
if(list.isEmpty()){
    // 빈 값에 대해 처리
}
else {
    // 코드 실행
}
```

#### 허락보다 용서가 쉽다

```
try {
    // 코드 실행
} catch Exception as e {
    // 잘 안될 경우 대응
}
```

### ROP? 

``` kotlin hl_lines="24 25"
sealed class NumberException: RuntimeException() {
  data class DivideByZero(override val message: String): NumberException()
  data class TooBig(override val message: String): NumberException()
  data class TooSmall(override val message: String): NumberException()
}

fun sum(a: Int, b: Int): Result<Int, NumberException> {
  val result = a + b
  if (result > 100) return Result.Failure(NumberException.TooBig("Too Big"))
  if (result < 0) return Result.Failure(NumberException.TooSmall("Too Small"))

  return Result.Success(result)
}

fun divide(a: Int, b: Int): Result<Int, NumberException> {
  if (b == 0) return Result.Failure(NumberException.DivideByZero("Divide By Zero"))
  return Result.Success(a / b)
}

fun main() {
  val result = sum(5, 10)
    .flatMap { divide(it, 0) }
    .recover {
      when (it) { // 예상한 에러가 발생할 경우 아래와 같이 처리 
                  // But, 예상하지 못한 에러가 발생했을 경우 어떻게 해야 할까?
        is NumberException.DivideByZero -> -1
        is NumberException.TooBig -> 100
        is NumberException.TooSmall -> 0
      }
    }

  println(result.value) // -1
}
```

## 모르는 것이 당연하다.

같은 공부를 했어도 누구는 깊이있게 이야기한다. 하지만, 걱정하지마라 {==모르는 것이 당연하니까==}

모르면 질문을 하자 그것은 부끄러운게 아니다.