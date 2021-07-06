---
layout: post
title: 네트워크 작업을 비동기 처리하기
subtitle: MVVM + Coroutine을 사용한 비동기 처리
gh-repo: bbang208
gh-badge: [follow]
tags: [android, mvvm, kotlin, coroutine, network, async]
comments: true
---

## 사건의 발단

필자는 현재 사내 모 프로젝트를 리팩토링하는 과정에 있다. 사실 말이 리팩토링이지 리메이크가 더 맞는 표현인 듯...

아무튼 기존 프로젝트를 AAC(Android Architecture Component)를 이용하여 MVVM패턴 + 100% Kotlin으로 바꾸는 아주 ~~미친짓~~을 하고 있다.

보안서약서를 작성한 나는 모든 사실에 대하여 말해줄 수 없지만, 한 가지 확실한 것은 네트워크 요청 시에 이루어져야 하는 작업량이 어마어마 하다는 것이다. Application Context를 변수로 갖고 있으면서, 싱글톤인 클래스가 꽤 있다.. 네트워크 요청 시에 Instance를 가져오고 값을 가져오고.. I/O 성능에 영향을 줄 법한 작업 요청들이 무수히 많다.

해당 로직을 실행하면, 우선 Android UI는 많은 thread를 감당하지 못한 채 프레임드랍이 일어난다. 로그에서도 skipped frame ~~~ 하면서 main thread에서 많은 작업을 하지 말라고 경고를 한다. 앱이 튕기거나 하진 않지만 이런 건 못 참지.. 절대..

하여튼 이러한 이유로 네트워크 요청, 응답, 오류 핸들링을 모두 비동기로 처리할 생각이다. 더 정확히는 NetworkCall 시에 필요한 Params들을 만들어야 하는데 이 과정에서 너무 많은 작업이 이루어지고 있다. createCall함수부터 processResponse, onFetchFailed 처리까지 모두 비동기 함수 처리로 변경할 생각이다.

요청에 필요한 데이터를 백그라운드를 통해 작업하고, `.async { ... }.await()`를 통해 작업이 완료될때까지 기다렸다가 수행하도록 할 예정이다.

 

## 작업 시작

우선 해당 프로젝트는 Android Clean Architecture를 기반으로 구성하였다. 그래서 NetworkCall 부분은 [해당 Repository](https://github.com/bbang208/architecture-components-samples/tree/main/GithubBrowserSample/app/src/main/java/com/android/example/github/repository)를 참고하길 바란다. 여기있는 소스를 기준으로 변경할 것이기 때문.

우선 네트워크 요청을 생성하는 부분을 살펴보면, Repository에서 NetworkBoundResource 클래스를 통해 network call을 생성하게 된다. 아래는 그 예시이다.

```kotlin
fun verifyMobileAuthCode(): LiveData<Resource<String>> {
        return object : BaseNetworkBoundResource<String, ResponseBody>() {
            override fun createCall(): LiveData<ApiResponse<ResponseBody>> {
              
                return appApiService.requestVerifyMobileAuth()
            }

            override fun processResponse(response: ApiSuccessResponse<ResponseBody>): String {
                TODO("Not yet implemented")
            }
        }.asLiveData()
    }
```

우리의 목적은 createCall 내부에서 Coroutine을 사용하는 것이다. 필자는 처음에 그냥 `CoroutineScope(Dispatchers.IO).launch { ... }` 를 사용하면 안 되나? 라는 생각을 했었다. 하지만 의미가 없었던게 어차피 Background에서 모든 작업을 마치고 반환된 결과값으로 파라미터에 넣어야 했기 때문이다.

그래서 아예 suspend함수로 만들어서 사용하기로 했다.



## NetworkBoundResource 를 수정하자

우선 Repository에서 재정의하여 사용할 함수들을 suspend로 변경해준다.

```kotlin
protected abstract suspend fun createCall(): LiveData<ApiResponse<RequestType>>

protected abstract suspend fun processResponse(response: ApiSuccessResponse<RequestType>): ResultType
```

요청을 생성하는 `createCall`과 응답 처리를 하는 `processResponse`를 `suspend`로 변경해준다. 그리고 Network 처리를 담당하는 `fetchFromNetwork()` 함수도 suspend로 변경해준다.

```kotlin
private suspend fun fetchFromNetwork() {
```

이렇게...



그리고 `fetchFromNetwork()` 에서 response에 따라 분기처리를 하게 되는데 여기서 CoroutineScope 를 이용하여 비동기 처리를 한다.

```kotlin
when (response) {
    is ApiSuccessResponse -> {
        CoroutineScope(Dispatchers.IO).launch {
            setValue(Resource.success(processResponse(response)))
        }
    }
    ...
}
```

`fetchFromNetwork()` 함수가 suspend가 되었기 때문에 해당 함수도 Coroutine Scope내에서 처리하여야 한다. 다만, LiveData를 Observe해야 하기 때문에 해당 Scope는 Main에서 처리하도록 한다.

```kotlin
init {
    CoroutineScope(Dispatchers.Main).launch {
        fetchFromNetwork()
    }
}
```

그리고 `ApiSuccessResponse`시에 백그라운드 처리를 하기 때문에 `setValue` 함수 또한 비동기 처리를 하여야 한다. 이 경우에는 Livedata에서 제공하는 함수가 있는데, `value.postValue()` 를 하면 된다.

```kotlin
@MainThread
private fun setValue(newValue: Resource<ResultType>) {
    if (result.value != newValue)
        result.postValue(newValue)
}
```



## 결과

이렇게 되면 suspend함수가 되어 비동기 처리를 할 수 있게 된다. 그럼 아래와 같은 처리가 가능해진다.

```kotlin
val myValue = withContext(CoroutineScope(Dispatchers.IO).coroutineContext) {
    getValue()
}
```

`getValue()`에서 데이터를 가져올 때 까지 기다렸다가 그 이후 작업을 처리한다.



아래는 해당 소스 gist를 공유한다.

<script src="https://gist.github.com/bbang208/9a8aeaf16bcb45501b54d7619edbe3f2.js"></script>