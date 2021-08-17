---
layout: post
title: ViewModel 안에서 Coroutine 사용하기
subtitle: LiveData와 함께 사용하는 Coroutine
gh-repo: bbang208
gh-badge: [follow]
tags: [android, 안드로이드, 코루틴, coroutine, livedata, viewmodel, suspend]
comments: true
---

## 코루틴을 사용하는 Repository function

MVVM패턴의 안드로이드 앱 개발을 하다보면 참 많은 일들이 일어난다(?)

이번에는.. Repository에서 데이터를 가져오는 함수가 코루틴을 사용하여 `suspend fun`이었는데, UI처리를 위해 해당 함수를 viewModel에서 바로 변수에 넣어 사용하고 싶었다.

그렇다면 보통의 케이스는..

```kotlin
val time = testRepository.getTime()
```

ViewModel에서 위처럼 사용할 수 있다. 그럼 `time`을 `observe` 하고 있던 곳에서 이벤트가 발생하게 되고 UI 처리 등등이 가능하게 된다.



그러나 `suspend`로 되어있다면 어떻게 될까? 아래 예시 코드를 보면...

```kotlin
val sum = testRepository.getSum()
```

`getSum()`에 빨간줄이 그이면서 `Suspend function 'getSum' should be called only from a coroutine or another suspend function` 이라는 오류 문구를 출력한다.

말 그대로 suspend function은 다른 suspend함수나 Coroutine scope에서 호출되어야 하기 때문이다. 그냥 편하게 함수로 뺄 수도 있지만.. 더 이쁘고 간결한 코드가 있다.

<br>

## LiveData와 Coroutine을 함께 쓸 수 있다.

언제나 그래왔듯이 공식 문서에는 필자와 비슷한 상황에 놓인 가여운 개발자들을 위한 해결방안들이 있다. [여기](https://developer.android.com/topic/libraries/architecture/coroutines?hl=ko#livedata)를 참고해도 좋다.

```kotlin
val sum: LiveData<Int> = liveData {
    val result = testRepository.getSum()
    emit(result)

    //or..
    val data = CoroutineScope(Dispatchers.IO).async { testRepository.getSum() }
    emit(data.await())

    //or..
    val data2 = viewModelScope.async { testRepository.getSum() }
    emit(data2.await())
}
```

**대박..!!**

결론부터 말하자면 **`liveData` 빌더 함수를 통해 `suspend` 함수를 호출하여 결과를 `LiveData` 객체로 제공할 수 있다**는 것이다.

위의 소스코드를 보면 알 수 있듯이 `liveData` 빌더 함수를 사용하여 `suspend fun getSum()` 함수를 비동기 처리하고 `emit()`을 사용하여 결과를 반환했다.

여기서 눈치채신 분들도 계시겠지만, `emit`을 사용하여 결과값을 반환하는 행위는 `coroutine flow`에서도 볼 수 있다.

예시 코드에서는 여러가지 방법을 제공했으나, 실제로 저렇게 그대로 구동시켜도 3번의 emit이 모두 호출되고 데이터가 반환된다. 따라서 3번의 이벤트가 그대로 발생한다. (직접 확인해보시길!)

## 문서를 잘 읽자

흠.. `suspend`를 어떻게 해야 좋을지 고민을 했었는데 답은 생각보다 가까운곳에 있었다. 조금만 더 찾아보는 습관을 기르도록 해야겠다..



### 참고하면 좋은 repository

[https://github.com/bbang208/Android-CleanArchitecture](https://github.com/bbang208/Android-CleanArchitecture)

