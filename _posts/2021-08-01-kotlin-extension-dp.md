---
layout: post
title: Int를 DP로 간단하게 변환하기
subtitle: Kotlin Extension을 사용하면 간단하게!
gh-repo: bbang208
gh-badge: [follow]
tags: [android, kotlin, dp, int, extension]
comments: true
---

## 레이아웃을 그릴 때...

우린 보통 레이아웃을 구성할 때 xml에서 구성을 한다. 그리고 Width, height, padding, margin 등을 설정하기 위해 정수 값을 넣게 된다.

하지만 코드에서 레이아웃을 구성할 땐 이야기가 다르다. xml에서 처럼 정수를 넣는다고 다 되는 것이 아니기 때문이다.

Int로 하든, Float으로 하든 앱을 사용하는 디바이스 화면의 크기에 맞춰 값을 변환하여 넣어주어야 한다. 이 때 우리는 DP로 변환 해줄 수 있는 코드를 **Kotlin Extension** 을 사용하여 좀 더 세련되게 코딩을 해보도록 하자.

## 간단하다.

View에서 작업해도 되고 따로 Util class로 빼서 써도 된다.

```kotlin
fun Int.toDp(): Int {
    return (this * Resources.getSystem().displayMetrics.density + 0.5f).toInt()
}
```

이렇게 하게 되면, Int에서 바로 변환이 가능하다.

`val convertedDP = 32.toDp()` 이렇게...

`Int.toDp()` 함수 스코프 안에 `this`가 바로 파라미터라고 생각하면 좀 쉽게 이해할 수 있다. 예시코드의 32가 this에 들어가는... 그런 구조다.

`Float`자료형도 동일한 코드로 사용할 수 있다. `Int.toDp()` 를 `Float.toDp()`로 수정만 하면 된다.



### 참고

- [Kotlin Extension의 정의를 알아보자](https://thdev.tech/kotlin/2020/10/27/kotlin_effective_08/)
