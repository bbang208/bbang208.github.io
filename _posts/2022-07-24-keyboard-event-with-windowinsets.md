---
layout: post
title: 키보드가 열리고 닫히는 이벤트 받기
subtitle: 드디어 구글이..!
gh-repo: bbang208/WindowInsets-Android
gh-badge: [star, fork, follow]
tags: [android, keyboard, event, windowinsets, 안드로이드, 키보드, 이벤트]
comments: true

---

## 기존에는..

우린 키보드가 열리고 닫히는 이벤트를 받기가 굉장히 어려웠다. 당장 스택오버플로만 찾아봐도 나오는 건...

edittext의 focus를 따른다거나, 키보드가 올라와 view를 가리는 영역의 크기를 계산하여 구한다거나.. 방법은 있었으나 썩 마음에 들진 않았다. 그러나 구글이 드디어 이를 지원해 준다는 포스트를 보게 되었다.



## 기다린 보람이 있구나

[참고 블로그 주소를 먼저 링크한다.](https://sungbin.land/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-windowinsets%EB%A1%9C-%ED%82%A4%EB%B3%B4%EB%93%9C-%EC%95%A0%EB%8B%88%EB%A9%94%EC%9D%B4%EC%85%98-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0-1-b6452ed44bc8)

위 블로그에서도 `WindowInsets`을 사용한다. 다만 저쪽은 애니메이션 구현까지 한 것이므로 우리는 키보드가 올라왔는지 안 올라왔는지 여부에 대해서 까지만 참고하도록 한다.



## 사용해보자.

원래 `WindowInsets` 은 Android SDK 29부터 추가되었다. 하지만 `AndroidX Compat`라이브러리를 통해 21버전 부터 지원이 가능하다.

그리고 추가로 `fitsSystemWindows`를 `false`로 설정해주어야 한다. 이를 변경해주지 않고 기본값인 `true`로 설정하게 되면 시스템 기본 insets을 적용하게 되면서 사용하고자 하는 WindowInsets이 제대로 동작하지 않는다고 한다.

```kotlin
WindowCompat.setDecorFitsSystemWindows(window, false)
```

그리고 실행을 하면...

![AppScreenCapture](/assets/img/2022-07-24-keyboard-event-with-windowinsets/Screenshot_20220724_170503.png){: width="300"}

위쪽 툴바가 가려진 것을 볼 수 있다. 당연하다. 이를 해결해보자.

우선 `WindowInsets Callback`을 받아야 한다. `OnApplyWindowInsetsListener` `implement`를 추가하자.

![AddImpl](/assets/img/2022-07-24-keyboard-event-with-windowinsets/Screenshot_2022-07-24 17.11.50.png)

주의할 점은.. `androidx.core.view` 에 포함된 클래스를 import해야 한다. 안 그럼 하위버전에서 앱이 죽어버린다.

그 다음 몸체를 구현해준다. (override)

```kotlin
override fun onApplyWindowInsets(view: View, insets: WindowInsetsCompat): WindowInsetsCompat {
        //시스템바, InputMethod
        val types = WindowInsetsCompat.Type.systemBars() or WindowInsetsCompat.Type.ime()

        // 패딩으로 설정하여, 결정된 insets을 적용
        val typeInsets = insets.getInsets(types)
        view.setPadding(typeInsets.left, typeInsets.top, typeInsets.right, typeInsets.bottom)

        // 새로운 WindowInsetsCompat.CONSUMED를 반환하여 insets가 뷰 계층 구조로 더 이상 전달되지 않도록 함
        // 이것은 deprecated된 WindowInsetsCompat.consumeSystemWindowInsets() 및 관련 함수를 대체합니다
        return WindowInsetsCompat.CONSUMED
    }
```

이제 ViewCompat을 사용하여 최상위 뷰에 적용해준다.

```kotlin
ViewCompat.setOnApplyWindowInsetsListener(binding.root, this)
```

이제 padding 관련 문제는 해결되었으니, 키보드 이벤트 처리를 해보자.



## 키보드 이벤트를 받을 수 있는 Callback 등록

```kotlin
ViewCompat.setWindowInsetsAnimationCallback(
            binding.root,
            object : WindowInsetsAnimationCompat.Callback(
                DISPATCH_MODE_STOP
            ) {
                override fun onProgress(
                    insets: WindowInsetsCompat,
                    runningAnimations: MutableList<WindowInsetsAnimationCompat>
                ): WindowInsetsCompat {
                    return insets
                }

                override fun onEnd(animation: WindowInsetsAnimationCompat) {
                    super.onEnd(animation)
                    val isKeyboardVisible = WindowInsetsCompat.toWindowInsetsCompat(binding.root.rootWindowInsets).isVisible(WindowInsetsCompat.Type.ime())
                    Log.e("MainActivity", "keyboardVisible: $isKeyboardVisible")

                    //ViewCompat.getRootWindowInsets(binding.root)?.isVisible(WindowInsetsCompat.Type.ime())
                }
            })
```

이렇게 되면, 키보드 Animation이 끝나는 시점에 `doEnd`가 호출되고, 키보드인 `InputMethod` 가 `visible` 인지 검사하여 `true` or `false` 를 반환한다.

로그 아래 주석코드로도 동작이 되지만, 우리는 하위버전 호환을 해야하므로, `WindowInsetsCompat.toWindowInsetsCompat` 함수를 사용하여 `RootWindowInsets`를 가져와 `isVisible` 함수를 사용하도록 한다.



## 전체 소스코드

[https://github.com/bbang208/WindowInsets-Android](https://github.com/bbang208/WindowInsets-Android)