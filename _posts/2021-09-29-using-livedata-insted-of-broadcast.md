---
layout: post
title: LocalBroadcastManager 대신 LiveData 사용하기
subtitle: 왜 진작에 쓰지 않았을까?
gh-repo: bbang208
gh-badge: [follow]
tags: [android, 안드로이드, broadcast, livedata]
comments: true
---

그동안 필자는 [LocalBroadcastManager](https://developer.android.com/jetpack/androidx/releases/localbroadcastmanager?hl=ko)라는 것을 앱 내 이벤트 발생 시 전달 및 처리를 위해 사용해 왔었다. 예를 들자면.. 앱이 동작중인 상태에서 FCMService 를 통해 Push가 들어오면, 즉시 이벤트 처리를 하기 위함이었다.

Push 수신이 되면 이벤트를 처리할 곳에 register 하고.. FCM서비스에서는 broadcast를 해주고.. 말은 쉽지만 intent를 통해 데이터를 넣고 관리한다는 것이 썩 맘에 들진 않았었다.

그러던 중.. 해당 서비스를 CleanArchitecture 기반으로 리팩토링 하면서 이참에 LiveData로 바꿔보자..! 라고 생각이 되어 시도해 본 것을 작성해보려 한다.

## LiveData 클래스 만들기

우선.. LiveData를 상속받는 클래스를 먼저 만들어야 한다.

```kotlin
class PushEvent: LiveData<String>() {

    private val listener = { data: String ->
        /**
         * in background...
         */
         postValue(data)

        /**
         * in main thread...
         */
        //value = price
    }

    override fun onActive() {
      	super.onActive()
    }

    override fun onInactive() {
      	super.onInactive()
    }
  
    companion object {
        private lateinit var sInstance: PushEvent

        @MainThread
        fun getInstance(): PushEvent {
            sInstance = if (::sInstance.isInitialized) sInstance else PushEvent()
            return sInstance
        }
    }
```

기본 코드는 위와 같다.

* `onActive()` 는 이 LiveData 객체에 활성 상태의 옵저버가 있다면 호출된다. 따라서 반복적으로 이벤트를 발생시켜야 하는 경우, 이 메서드에서 업데이트를 해야한다.
* `onInactive()` 는 역으로 이 LiveData 객체의 활성 상태의 옵저버가 없다면 호출된다. 이 함수가 호출된다면 옵저버가 없기 때문에 불필요하게 데이터를 가져오지 않아도 된다.

### 여기서 잠깐.. 왜 Singleton을 만들었는지?

사실 굳이 Singleton으로 하지 않아도 옵저빙과 값 입력은 잘 된다. Singleton으로 만들어 사용하고자 하는 이유는 여러 Activity, Fragment, Service간에 객체를 공유하기 위함이다.

우리는 LiveData를 옵저빙 할 때 아래와 같은 방법으로 사용한다.

```kotlin
testLiveData.observe(viewLifecycleOwner, {
...
})
```

이렇게 view와 연결된 lifecycle을 파라미터로 받게 된다. LiveData 객체가 Lifecycle 즉 생명주기를 알고 있다는 뜻은 여러 Activity, Fragment, Service간에 객체를 공유할 수 있다는 의미도 포함하고 있다.

필자가 목표하는 `LocalBroadCastManager` 를 대체하기 아주 적합하다. 이벤트가 발생하면 어디서든지 같은 이벤트를 옵저빙하고 처리할 수 있기 때문이다.



## 값을 넣고 옵저빙하려면?

```kotlin
fun updateValue(data: String) {
    value = data
}
```

위 함수를 추가하고..

값을 쓰고자 하는 곳에서는..

```kotlin
PushEvent.getInstance().updateValue("asdasd")
```

그리고 옵저빙은 아래와 같이 한다.

```kotlin
PushEvent.getInstance().observe(this, {
    Timber.e("event: $it")
})

//in fragment....
PushEvent.getInstance().observe(viewLifecycleOwner, {
    Timber.e("event: $it")
})
```



그럼 끝이난다.



<br>

## High Order Function 으로 사용해보기

맨 처음으로 돌아가 보면 기본 코드에 `listener` 객체가 있다. 이것을 활용한 방법이다. 간단한 예시이기 때문에 그냥 같은 클래스 안에 만들어서(...) 보여드리도록 하겠다. ~~귀찮아..~~

```kotlin
private fun requestUpdate(listener: (String) -> Unit) {
    CoroutineScope(Dispatchers.Default).launch {
        delay(3000)
        listener("test22")
    }
}
```

약 3초뒤 이벤트를 발생시키도록 하게 함수를 만들었다. 그리고..

```kotlin
override fun onActive() {
    super.onActive()
    requestUpdate(listener)
}
```

`onActive()` 시점부터 데이터를 가져오도록 리스너 등록을 한다.

이렇게 하면 똑같이 옵저빙 하고 있는 객체에 가서 3초 뒤 이벤트 발생이 된다.

## 전체 코드

<script src="https://gist.github.com/bbang208/0facdb8820ffbd4ed7dfc7ffa43d3d0f.js"></script>

<br>

## 결론

활용도도 상당할 것 같고 데이터도 내 맘대로 만들어 보낼 수 있다는 것과, 생명주기를 따라간다는 점이 상당히 인상깊다. Push말고도 다방면으로 사용이 가능해서 열심히 적용해야겠다.
