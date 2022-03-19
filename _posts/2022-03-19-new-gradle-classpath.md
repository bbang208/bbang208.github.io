---
layout: post
title: 새로운 Gradle에서 classpath의 위치
subtitle: 가이드좀 친절하게 써주면 어디 덧나?
gh-repo: bbang208
gh-badge: [follow]
tags: [android, gradle, classpath]
comments: true
---
어느날 프로젝트를 새로 만들고 dependency를 추가하던 중.. classpath를 추가하라는 가이드가 있어서 추가를 하려고 했다. 어느 때와 다르지 않게 자연스럽게 project `build.gradle` 파일을 열어 추가하려고 하는 순간.. 이게 뭐지? 싶었다.

아래는 우리가 잘 알고있는 build.gradle(project) 파일의 일부 내용이다.

```groovy
// Top-level build file where you can add configuration options common to all sub-projects/modules.
buildscript {
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:7.1.2'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:1.6.10"

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files

        classpath "com.google.dagger:hilt-android-gradle-plugin:2.38.1"
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

그런데 이번에 새로 만든 프로젝트의 파일은 좀 달랐다.

```groovy
// Top-level build file where you can add configuration options common to all sub-projects/modules.
plugins {
    id 'com.android.application' version '7.1.2' apply false
    id 'com.android.library' version '7.1.2' apply false
    id 'org.jetbrains.kotlin.android' version '1.6.10' apply false
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

**이게 뭐야???**

그래서 이것저것 정보를 찾아봤다. Gradle에 무언가 업데이트가 되었는지... 새로운 스타일이 생겼는지...

```groovy
id 'com.google.dagger:hilt-android-gradle-plugin' version '2.38.1' apply false
```

일단... 추가는 해야되니까 위에거 처럼 바꿔서 해봤다. 근데 Sync오류가 계속 난다..

일단은 검색을 좀 해봤는데..

내가 정보력이 부족한건가 싶었다. 아무리 찾아봐도 관련 정보는 없고.. StackOverflow에서도 나와 같은 질문은 하는 사람은 있었지만 답은 없었다.

## 어이없는 해결 방법

그냥 이전 Gradle 의 모습처럼 다른 옛날 프로젝트에서 그대로 긁어서 적용해봤다.

```groovy
buildscript {
    dependencies {
        classpath "com.google.dagger:hilt-android-gradle-plugin:2.38.1"
    }
}

plugins {
    id 'com.android.application' version '7.1.2' apply false
    id 'com.android.library' version '7.1.2' apply false
    id 'org.jetbrains.kotlin.android' version '1.6.10' apply false
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

Plugins 태그 위에 그냥 넣었다.

근데 Sync가 잘 된다... 아...



이런거 바뀌면 제대로 설명좀 해주란 말이야 ㅠㅠ

저렇게 추가 했을 때 그냥 호환이 되는 건지.. 아니면 정석적인 방법이 있는건지.. 저대로 쓰는게 맞는건지는 나도 잘 모르겠다. 최신 버전의 정확한 가이드가 있다면 수정토록 하겠다.



하여간 Sync는 잘 되어서 문제는 없다.