---
title: "[GitHub] GitHub 블로그 timezone 관련 문제"
excerpt: "github 블로그 글을 오늘 날짜로 작성 후 글이 보이지 않을 때"

categories:
  - GITHUB
tags:
  - [github, timezone]

permalink: /github/github-post2/

toc: true
toc_sticky: true

date: 2024-03-18
last_modified_at: 2024-03-18
---

## I. 오늘 날짜의 글이 보이지 않는 문제 발생
전에 포스팅했던 글은 모두 보이는데 오늘 작성한 글은 블로그에 보이지 않는 문제가 발생하여서 의심되는 부분을 하나씩 수정해보다가 날짜를 이전 날짜로 작성하면 글이 보이는 것을 발견했다.


## II. Jekyll timezone 설정
> [Jekyll Docs](https://jekyllrb-ko.github.io/docs/installation/windows/)
  
Jekyll에 대한 설명을 찾아보니까 default가 UTC/GMT 00:00 였다.  
![output](/assets/images/posts_img/etc-cate/timezone.png)
  
미래 시간으로 인식되면 글이 보이지 않았던 것 같다... 그래서 한국 시간대로 바꿔주기로 하였다.  

![output](/assets/images/posts_img/etc-cate/timezonegpt.png)  
  
gpt가 timezone 부분을 Asia/Seoul로 변경하라고 알려줬다.  

![output](/assets/images/posts_img/etc-cate/timezone1.png)  
  
_config.yml에서 timezone 부분을 찾아서 변경해주었다.  

![output](/assets/images/posts_img/etc-cate/18.png)  
  
이제 오늘 작성한 글도 아주 잘 보인다.


## III. etc.

### 참조
https://jekyllrb-ko.github.io/docs/installation/windows/

