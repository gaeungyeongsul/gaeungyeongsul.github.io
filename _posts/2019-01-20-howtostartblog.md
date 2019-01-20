---
layout: post
title:  "깃허브 블로그 만들기"
date:   2019-01-20 17:15:11 +0000
categories: portfolio
---
# 깃허브 블로그 만들기

포트폴리오를 만들기 위해 깃허브 블로그를 개설했습니다. (짝짝짝짝)

>1. 디자인 선택

우선 블로그 디자인을 선택해야 합니다.
디자인 목록은 [여기를 클릭해주세요](https://github.com/jekyll/jekyll/wiki/themes)
`demo`를 클릭하여 샘플을 보면서 어떤 디자인을 할지 결정해주세요.
<br>
저는 깔끔한 strata를 선택했습니다. 선택한 후 `source`를 클릭해주세요.<br>
<img src="/images/blogmake/make1.JPG" width="400" height="400">
<br>
이렇게 깃허브와 연결됩니다. 여기서 `fork`를 눌러주세요. 그러면 나의 github 계정으로 다른 사람이 만들어 놓은 source 를 가져올 수 있습니다.
<br>
<img src="/images/blogmake/make3.jpg" width="800" height="400">
<br>
>2. 코드 수정하기

`README.md` 파일을 클릭해주세요. 설명히 잘 나와있어요.
하라는대로 하면 됩니다.
<br>
<img src="/images/blogmake/make4.jpg" width="400" height="400">
<br>
README.md 파일을 보면 이렇게 나와있습니다.
<br>
1. `Repository name`을 yourgithubusername.github.io로 변경하고 <br>
2. `_config.yml`파일을 수정해서 테마 옵션을 변경해야합니다.
<br>
<img src="/images/blogmake/make5.jpg" width="400" height="400">

<br>
우선 settings에 들어가서 repositoryname을 yourgithubusername.github.io 변경 후 Rename버튼을 클릭해주세요.
<br>
<img src="/images/blogmake/make6.jpg" width="400" height="400">
<br>
다시 코드로 돌아와서 `_config.yml` 파일에 들어가서 연필 모양의 수정하기 버튼을 클릭하세요.
<br>
<img src="/images/blogmake/make7.jpg" width="400" height="400">
<br>
그런 다음 title(블로그명), description(소개글)을 원하는대로 변경하시고 url: `https://yourgithubusername.github.io`로, github_username, email을 변경해주세요. 그외에도 수정할 부분을 마음대로 수정하면 됩니다.
<br>
<img src="/images/blogmake/make8.jpg" width="400" height="400">
<br>이제 `https://yourgithubusername.github.io`를 들어가면 블로그가 잘 만들어졌습니다! 짝짝짝짝
<br>
프로필사진을 변경하고 싶다면 우선 `images` 폴더에 이미지 파일을 올립니다.
<br>
<img src="/images/blogmake/make9.jpg" width="400" height="400">
<br>
<img src="/images/blogmake/make10.jpg" width="400" height="400">
<br>
`header.html`로 가서 "{{ site.github.url }}/images/사진이름.jpg(png)"로 경로를 변경해줍니다.
<img src="/images/blogmake/make11.jpg" width="400" height="400">
이러면 정말 블로그가 완성됩니다.
