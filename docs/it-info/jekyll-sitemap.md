---
layout: default
title: jekyll 블로그 sitemap 작성하는 방법
parent: IT 및 개발관련정보
---

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "BlogPosting",
  "headline": "{{ page.title }}"
}
</script>

# 개요

jekyll로 만든 정적 블로그의 경우 sitemap을 간단하게 만들 수 있습니다.

그냥 구글에 "블로그 sitemap 만들기를" 검색할 경우

대부분의 블로그가 sitemap을 만들어주는 사이트를 추천해주거나

티스토리의 경우 sitemap을 자동으로 만들어주는 기능이 생겨서 그걸 사용하라고 나옵니다.

하지만 jekyll로 만든 블로그도 그와 비슷하게 자동으로 sitemap을 만들 수 있습니다!


# sitemap.xml 파일 만들기

１．루트 디렉토리에서 커맨드 라인에 아래와 같이 입력한다.

`$touch sitemap.xml`
(그냥 최상위 디렉토리에 직접 sitemap.xml 파일을 생성해도 괜찮다)

２．해당 파일 안에 아래와 같이 입력한다(이상하게 html 코드가 먹통이라 사진으로 올립니다 ㅠㅠ)

![image](https://user-images.githubusercontent.com/69494230/205417835-5897845f-606b-47a1-b3b1-271e6051d12a.png)

주의할 점은 <loc> 부분에 자신의 블로그 baseURL을 입력해야한다
(예: https://app.netlify.com)

# 만들어진 sitemap 확인하기

이렇게 등록했으면 직접 확인할 수 있다
방금 입력한 `"자신의 블로그baseURL"/sitemap.xml`로 들어가면 
다음과 같이 xml 파일이 만들어진 것을 확인할 수 있다

![image](https://user-images.githubusercontent.com/69494230/205417267-1b338004-4d81-411c-a39d-2e791a54ff3c.png)

# 만들어진 sitemap을 구글 웹마스터에 등록하기

생성한 sitemap은 구글 웹마스터 등에 등록하여 사용할 수 있습니다.
(현재 위와 같은 방법을 사용해서 등록했는데 문제없이 잘 색인되고 있습니다.)

![image](https://user-images.githubusercontent.com/69494230/205417967-d73340ab-0c66-4e28-a4ae-4ab526c6dd1c.png)