---
layout: post
title: Animation과 Viewpager2 사용하기
subtitle: 이거 완전 RecyclerView
gh-repo: bbang208
gh-badge: [follow]
tags: [android, 안드로이드, viewpager2, animation, fragemnt]
comments: true
---

이번에 Viewpager를 사용하여 카드뷰 리스트를 구현할 일이 생겨 고민하던 중, 이참에 Viewpager2를 적용해 보고자 이것저것 찾아보게 되었다.

## 왜 Viewpager2?

기존의 ViewPager는 item관리가 어려웠던 것으로 기억한다. 새로고침 이슈도 있었던 것 같고.. 지원 중단된 시점에서 언젠가는 Viewpager2로 마이그레이션을 해야했던 것도 있었다. 이 참에 한 번 써보기로 하였다. Viewpager2는 아래와 같은 장점이 있다.

### 지속적인 지원.

일단 기존의 Viewpager의 문제를 개선하기도 했고 확장성도 뛰어나며(?) 지속적인 개발 지원이 이루어 지고 있다.

### 세로 방향 스크롤을 지원한다.

Viewpager2는 RecyclerView를 기반으로 개발되었다. 그래서 그런지 `android:orientaion` attribute를 사용하여 세로 방향 스크롤을 바로 사용할 수 있다. RecyclerView를 사용하면서 LayoutManager가 많은 편리함을 주었는데 여기서도 빛을 발하는 듯.

### RTL지원

RTL이 뭔지 생소한 사람들이 있을 수 있다. 이건 아랍어를 사용하는 나라에서 자주 볼 수 있는 것인데, **Right to Letf**라고 한다. 글을 쓸 때 왼쪽에서 오른쪽으로 진행하는 우리나라와는 달리 아랍어는 오른쪽에서 부터 왼쪽으로 글을 쓴다. 이 때문에 시스템 언어를 아랍어로 변경하게 되면 모든 레이아웃이 오른쪽에서 부터 시작하는 것을 볼 수 있다.

`AndroidMenifest` 파일에서 보면 `supportRtl` 이라는 설정값이 있는데 이는 아랍어 환경 지원 여부에 대한 설정값이다.

자꾸 얘기가 길어지는데.. 하여튼 RTL지원으로 인해서 국가를 코드상에서 구분할 필요 없이 바로 item을 맞게 설정해 준다는 이야기다. **아랍어를 지원하는 앱들에겐 희소식..!**

### UI 업데이트 문제 해결

이는 기존 Viewpager에서 발생했던 `notifyDataSetChanged` 버그를 해결한 것이다. RecyclerView를 기반으로 개발되었기 때문에 데이터 변경 시 UI변경을 즉각 할 수 있다.

<br>

## Viewpager2 적용하기

### 의존성 추가

```groovy
implementation "androidx.viewpager2:viewpager2:1.0.0"
```

app모듈 gradle에 의존성을 추가한다.

### XMl 생성

```xml
<androidx.viewpager2.widget.ViewPager2
    android:id="@+id/pager"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
```

xml에 위와같이 생성해주고..

### Adapter  생성

```kotlin
private inner class ScreenSlidePagerAdapter(fa: FragmentActivity) : FragmentStateAdapter(fa) {
    override fun getItemCount(): Int = 5

    override fun createFragment(position: Int): Fragment = CardFragment()
}
```

### Adapter 적용

```kotlin
val pagerAdapter = ScreenSlidePagerAdapter(this)
pager.adapter = pagerAdapter
```

이러면 끝난다... `CardFragment()`는 본인이 원하는 대로 Fragment를 만들어 테스트 해 보면 쉽게 생성이 된다. 자 그럼 Animation을 추가해 보도록 하자. 정말 간단하다.

### setPageTransformer 사용

슬라이드 할 때마다 크기가 작아졌다 커졌다 하는 아래와 같은 모양의 view를 만들어보자.

![page transformer](/assets/img/2021-09-04-viewpager/anim_page_transformer.gif)

*출처: Android Developeres*

<br>

아래와 같은 Transformer 클래스를 만든다.

```kotlin
  private const val MIN_SCALE = 0.85f
    private const val MIN_ALPHA = 0.5f

    class ZoomOutPageTransformer : ViewPager2.PageTransformer {

        override fun transformPage(view: View, position: Float) {
            view.apply {
                val pageWidth = width
                val pageHeight = height
                when {
                    position < -1 -> { // [-Infinity,-1)
                        // This page is way off-screen to the left.
                        alpha = 0f
                    }
                    position <= 1 -> { // [-1,1]
                        // Modify the default slide transition to shrink the page as well
                        val scaleFactor = Math.max(MIN_SCALE, 1 - Math.abs(position))
                        val vertMargin = pageHeight * (1 - scaleFactor) / 2
                        val horzMargin = pageWidth * (1 - scaleFactor) / 2
                        translationX = if (position < 0) {
                            horzMargin - vertMargin / 2
                        } else {
                            horzMargin + vertMargin / 2
                        }

                        // Scale the page down (between MIN_SCALE and 1)
                        scaleX = scaleFactor
                        scaleY = scaleFactor

                        // Fade the page relative to its size.
                        alpha = (MIN_ALPHA +
                                (((scaleFactor - MIN_SCALE) / (1 - MIN_SCALE)) * (1 - MIN_ALPHA)))
                    }
                    else -> { // (1,+Infinity]
                        // This page is way off-screen to the right.
                        alpha = 0f
                    }
                }
            }
        }
    }
```

이 클래스를 만들었다면.. 적용을 해보자.

```kotlin
pager.setPageTransformer(ZoomOutPageTransformer())
```

끝이다..



## 결론

정말 RecyclerView를 많이 닮았고, 사용하기 편리해졌다. PageTransformer를 더 커스텀하여 멋진 앱을 만들 수 있을거 같다. 이런 거 미리 만들어놓고 모아놓은 사이트 어디 없나..

위 프로젝트는 아래 Repository 에서 확인할 수 있다.

[https://github.com/bbang208/ViewPager2-Animation-Sample](https://github.com/bbang208/ViewPager2-Animation-Sample)



### 참고한 블로그

* https://www.charlezz.com/?p=1037
* https://developer.android.com/training/animation/screen-slide-2?hl=ko#zoom-out
* https://blog.gangnamunni.com/post/viewpager2/
