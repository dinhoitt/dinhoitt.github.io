---
layout: post
title:  "Flutter&Firebase 오류 해결"
date:   2024-01-23T14:03:52+09:00
author: DINHO
categories: "Flutter"
cover:  "/assets/post/flutter.jpg"
---

PlatformException (PlatformException(sign_in_failed, com.google.android.gms.common.api.ApiException: 10: , null, null))

플루터로 개발을 하다가 이런 오류를 보신 적 있으신가요? 보통 Firebase를 이용해서 구글 로그인을 할 때 생기는 문제인데요. 요악하자면 SHA-1키 문제입니다. 

구글에 검색해보시면 여러 방법인 나오지만 저는 아무리 해도 안 되더라고요 ㅠㅠㅠ 그래서 이것저것 시도해 본 결과! 무조건 해결 가능한 방법을 찾았습니다. 주위 동료들도 같은 오류를 겪고 있었는데 제가 다 해결해주었습니다😁😁 단계별로 차근차근 알려드리겠습니다. 그럼 빠르게 GO!!

1. build/app/outputs/apk/debug 경로로 폴더에 들어간다!

    <img src="/assets/post/flt.jpg">

2. 해당 폴더에서 cmd를 실행시킨다!

    <img src="/assets/post/fffff.png">

3. keytool -printcert -jarfile app-debug.apk를 입력한다!!

    ```cmd
    keytool -printcert -jarfile app-debug.apk
    ```

    app-debug.apk에 해당하는 파일 이름이 다르다면 본인이 갖고 있는 파일의 이름을 입력하세요!!

4. 사진과 같이 SHA-1 지문이 나오면 복사해주세요!!!

    <img src="/assets/post/debug.png">

5. Firebase 콘솔로 이동하여 지문 추가를 눌러주고 붙여 넣기만 하면 끝!!!

<img src="/assets/post/sha.png">

    5단계만 거치면 해결 되실겁니다!!! 도움이 되길 바랍니다.

    추가로 여쭤보실 것 있으면 댓글 남겨주세요!!