---
layout: post
title: 앱 보안성 향상 - 무결성 검증
subtitle: 완성도 높은 탄탄한 앱 만들기
gh-repo: bbang208
gh-badge: [follow]
tags: [android, 안드로이드, 무결성, 보안]
comments: true
---

안드로이드 앱의 보안성을 향상시키는 방법 중에는 여러가지가 있겠지만, 보통은 아래와 같다.

* 무결성 검증
* 루팅 감지
* 난독화

이번 글은 위 세 가지 사항 중 무결성 검증에 대해 작성해보려 한다.



### 무결성 검증

앱의 무결성 검증은 정보보호의 3요소 중 한가지 이기도 하다. 누군가에 의해 위/변조된 앱이 동작하는 것을 방지하는 것을 목표로 한다.

안드로이드 앱은 디컴파일 툴을 이용하여 어느정도 디컴파일이 되기 때문에, 누군가에 의해 임의로 디컴파일 후 일부 소스가 조작되어 다시 컴파일될 수 있다. (이를 막을 수 있는 것이 난독화 처리이다.)

임의로 조작된 앱은 나비효과가 되어 시스템 상에서 큰 문제가 될 수 있다. 또한 이러한 앱이 공유되어 널리 사용되게 된다면... (이하 생략...)

무결성은 소스코드 모듈 난독화를 통해 어느정도 방지할 수는 있지만, 완벽하진 않다. 이를 어느정도 방지하기 위해서는 허가된 사용자만이 수정하고, 배포된 앱만을 사용할 수 있도록 하여야 한다.

<br>

### 어떻게 해야 좋을까?

우리는 앱을 배포하기 위해 빌드 시 Signing을 하게 된다. Signing시에 사용되는 키스토어 파일이 갖고 있는 키와, 이 키스토어로 서명된 앱이 가지고 있는 키를 비교하는 나름(...) 간단한 방법으로 무결성을 검증할 수 있다.

같은 서명 키로 Sign한 앱은 서명키에 대한 Hash값이 동일하기 때문에 이를 비교하여 무결성을 검증할 수 있다. 누군가 임의로 수정 후 다시 컴파일 하였다면 키가 달라지기 때문이다.

아래는 앱에서 서명 키에 대한 hash를 가져올 수 있는 소스다.

```kotlin
fun getHash(): String? {
        try {
            lateinit var result: String

            val info = packageManager.getPackageInfo(
                packageName,
                PackageManager.GET_SIGNATURES
            )
            for (signature in info.signatures) {
                Timber.d(signature.toCharsString())
                val md = MessageDigest.getInstance("SHA-256")
                md.update(signature.toByteArray())
                val hash = Base64.encodeToString(md.digest(), Base64.DEFAULT)
                result = hash
            }
            return result
        } catch (e: NoSuchAlgorithmException) {
            e.printStackTrace()
            return null
        } catch (e: PackageManager.NameNotFoundException) {
            e.printStackTrace()
            return null
        }
    }
```

SHA-256 키를 가져와 Base64로 인코딩한다.

그리고 아래는 Signing에 사용되는 키스토어 파일(jks)의 Hash를 가져오는 커맨드이다.

```
$ keytool -exportcert -alias <key alias> -keystore <keystore 파일 경로> | openssl sha256 -binary | openssl base64
```

이제 위 커맨드에서 나온 값을 서버에 저장하여 앱에서 나온 키와 비교하면 된다.

누군가에 의해 위/변조된 앱이라면 hash값이 다르게 나올 것이고 그에 따라 추가 대응하면 된다.



### 참고

* https://right-hot.tistory.com/entry/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-%ED%82%A4%ED%95%B4%EC%8B%9C-%EC%96%BB%EB%8A%94-%EB%B0%A9%EB%B2%95-debug-keyhash-release-keyhash-googlePlay-keyhash