---
layout: post
title: 로그 이쁘게 출력하기
subtitle: Timber가 좋긴 하네
gh-repo: bbang208
gh-badge: [follow]
tags: [android, kotlin, log, logger, timber]
comments: true
---

## 로그도 이쁘게 출력하고 싶은 개발자

어느 플랫폼이건 프로그래밍을 하다 보면 로그를 출력할 때가 많다. 올바른 결과값이 맞게 출력이 되었는지, 값은 제대로 파싱이 되었는지 확인하기 위해 로그 출력을 한다.

안드로이드에서도 `Log` 또는 `println`을 사용하여 로그를 출력할 수 있다. 하지만 앞서 작성한 두 가지 로그 출력 방법은, 썩 이쁘지 않게 로그가 출력되고 상당히 귀찮다.

- 어떤 클래스에서 작성된 로그인지 직접 작성해야 한다.
- 어떤 메소드에서 발생된 로그인지 직접 작성해야 한다.
- 몇 번째 라인에서 발생된 로그인지 직접 작성해야 한다.

결론은 뭐.. **직접 작성해서 귀찮다는 것이다.** 이를 해결해 줄 로깅 라이브러리가 있다.

<br>

## 로깅의 한 줄기 빛.. [Timber 라이브러리](https://github.com/JakeWharton/timber)

이미 아는 사람들은 다 안다는 '그 분' 께서 제작하신 라이브러리다. 안드로이드 개발자로 유명한 **JakeWharton** 선생님이시다.. 이 분 덕에 코틀린이 많이 유명해지기도 했다.

Timber는 기존에 사용할 수 있었던 Log 클래스의 확장 유틸리티 라이브러리라고 할 수 있겠다. 사용법은 간단하다.

```kotlin
Timber.d("log message")
Timber.d("value: %s", "string value")
```

일반 Log처럼 사용할 수 있다. d, i, e, w 등등... 

<br>

### 그래서 뭐가 다른데?

일단, TAG를 귀찮게 쓰지 않아도 된다. 보통은 final로 박아놓고 해당 클래스의 이름을 쓰는 것이 보통 국룰이라고 할 수 있겠다.. 하지만 Timber 는 그냥 메세지만 써도 **알아서** 로그가 작성된 클래스의 이름을 붙여준다.

```kotlin
viewModel.test.observe(this, { res ->
    Timber.e("status: ${res.status}")
    Timber.e("data: ${res.data?.time}")
    Timber.e("errorMessage: ${res.message}")
})
```

```
2021-08-08 19:07:59.274 io.github.bbang208.cleanarchitecture E/MainActivity: status: SUCCESS

2021-08-08 19:07:59.275 io.github.bbang208.cleanarchitecture E/MainActivity: data: 10:08:03 AM

2021-08-08 19:07:59.275 io.github.bbang208.cleanarchitecture E/MainActivity: errorMessage: null
```

정말이지.. 와튼 선생님은 참으로 대단하신 분이다.

그 분의 선행은 여기서 끝나지 않는다. 로그 스타일 커스텀이 가능하다.

<br>

### Method 이름, Line등 추가 정보가 표시되도록 해보자

Timber를 사용하기 위해서는 Application 또는 onCreate시 DebugTree를 plant 해주어야 한다. 닉값 제대로 하는 라이브러리다 ㅋㅋ.

이 때 로그 출력형태를 결정하는 DebugTree를 커스텀해줄 수 있다. 필자는 메소드명과 로그가 출력된 라인 넘버까지 표시해 주도록 하겠다. 우선 CustomDebugTree 클래스를 만든다.

```kotlin
class CustomDebugTree : Timber.DebugTree() {
    override fun createStackElementTag(element: StackTraceElement): String {
        return String.format(
            "[%s:%s][M:%s]",
            super.createStackElementTag(element),
            element.lineNumber,
            element.methodName
        )
    }
}
```

그리고 Application 에서 심어주도록 한다.

```kotlin
if (BuildConfig.DEBUG) Timber.plant(CustomDebugTree())
```

이렇게 해주면 아까 위에서 출력했던 로그들이 아래처럼 출력된다.

```
2021-08-08 19:15:16.134 io.github.bbang208.cleanarchitecture E/[MainActivity:26][M:onCreate$lambda-0]: status: SUCCESS

2021-08-08 19:15:16.135 io.github.bbang208.cleanarchitecture E/[MainActivity:27][M:onCreate$lambda-0]: data: 10:15:20 AM

2021-08-08 19:15:16.136 io.github.bbang208.cleanarchitecture E/[MainActivity:28][M:onCreate$lambda-0]: errorMessage: null
```

이야.. 이건 뭐... 굉장하다고 밖에 할 수 없겠다(?)

다른 방법으로도 커스텀이 가능하니 잘 활용해보도록 하자.

다음번엔 Network Call에 대한 요청과 응답에 대해 Interceptor를 통해 Logging하는 포스팅을 한 번 작성해볼까 한다.



이상!
