---
layout: post
title: 동일한 자료형의 다중 Module 제공
subtitle: Hilt를 사용한 Di의 Multiple Module
gh-repo: bbang208
gh-badge: [follow]
tags: [android, 안드로이드, hilt, 힐트, di, multiple, module]
comments: true
---

## 문제의 시작

음.. Hilt를 사용하여 Dependency Injection 을 하고 있는 프로젝트가 있다. Module을 통해 객체를 생성하고 필요한 곳에 주입을 하는 대충 이런 느낌의 것이다.

필자는 `EncryptedSharedPreference` 라는 Android의 보안 라이브러리 중 하나를 쓰고 있다. 해당 라이브러리는 앱에서 설정값을 저장할 때 Key, Value 모두 암호화 하여 값을 읽고 쓸 수 있도록 도와준다.

아무튼, 본인은 저장되는 값들의 도메인이 각각 다르기에 따로 파일을 나눠 저장하고 싶었다. 그런데, Di를 사용하고 있어 같은 자료형에 대해서 구분해줄 무언가가 필요했다. 그래서 이 방법에 대해 작성하려 한다.

<br>

## Qualifier를 사용하자

### Qualifier 지정

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {

    @Qualifier
    @Retention(AnnotationRetention.BINARY)
    annotation class AppSharedPreference

    @Qualifier
    @Retention(AnnotationRetention.BINARY)
    annotation class ConfigSharedPreference
  ...
}
```

예시로 앱 설정에 관한 것들을 저장하는 `SharedPreference` 객체와, 다른 설정값들을 저장하기 위한 `SharedPreference` 객체를 생성하기 위해 두 가지로 분리하여 지정해보도록 하겠다.

### SharedPreference 객체 생성

```kotlin
private val keyGenParameterSpec = MasterKeys.AES256_GCM_SPEC
private val mainKeyAlias = MasterKeys.getOrCreate(keyGenParameterSpec)
```

`EncryptedSharedPreference`를 사용하기 위한 key생성을 한다.

그 다음 **각각 다른 파일로 생성할 같은 `SharedPreference` 객체**를 만든다.

```kotlin
@AppSharedPreference
@Provides
@Singleton
fun provideAppPreference(@ApplicationContext context: Context): SharedPreferences {
    return EncryptedSharedPreferences.create(
        "app_pref",
        mainKeyAlias,
        context,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )
}

@ConfigSharedPreference
@Provides
@Singleton
fun provideConfigPreference(@ApplicationContext context: Context): SharedPreferences {
    return EncryptedSharedPreferences.create(
        "config_pref",
        mainKeyAlias,
        context,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )
}
```

위 코드에서 눈여겨보아야 할 부분은 `@AppSharedPreference` 와 `@ConfigSharedPreference` 이다. 위에서 `@Qualifier` 로 만들어놓은 한정자를 사용한다.

<br>

## 사용할 땐 어떻게 구분할까?

그럼 실제 클래스에서 사용할 땐 아래와 같이 주입받을 수 있다.

```kotlin
class AppPreference @Inject constructor(
    @AppModule.AppSharedPreference private val appPreference: SharedPreferences,
    @AppModule.ConfigSharedPreference private val configPreference: SharedPreferences
) {

    fun getBoolValue(): Boolean {
        return appPreference.getBoolean("key_boolean", false)
    }

    fun getConfigValue(): Boolean {
        return configPreference.getBoolean("config_boolean", true)
    }
}
```

이렇게 되면 각각 다른 Preference에서 원하는 값을 가져올 수 있다. 물론 지정할 때도 마찬가지.

![AppDataImage](/assets/img/2021-08-25-multiple-bindings/multiple-binding-img1.png)

실제 앱 저장소를 따라 가보면 원하는 대로 파일이 구분되어 생성된 것을 볼 수 있다.

