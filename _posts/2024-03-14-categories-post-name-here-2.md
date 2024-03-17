---
title: "[GitHub] GitHub 블로그 사용해보기"
excerpt: "github pages에 대한 설명과 github 블로그 커스텀, 포스팅에 대해서"

categories:
  - GITHUB
tags:
  - [tag1, tag2]

permalink: /github/post-name-here-2/

toc: true
toc_sticky: true

date: 2024-03-14
last_modified_at: 2024-03-14
---

## I. 깃허브 블로그를 제대로 사용해보자
깃허브 블로그를 생성했으니 어떤 방식으로 돌아가는지 살펴보고 변경이 필요한 부분에 대해서는 직접 자료를 찾아가면서 변경해보도록 하려고 한다.

## II. Github 블로그 관련 정보

### GitHub Pages
![output](/assets/images/posts_img/etc-cate/gitpages.png)  
GitHub Pages는 GitHub 리포지토리에서 `HTML`, `CSS` 및 `JavaScript` 파일을 직접 가져와서 필요에 따라 빌드 프로세스를 통해 파일을 실행하고 웹 사이트를 게시하는 `정적 사이트 호스팅 서비스`라고 한다.
![output](/assets/images/posts_img/etc-cate/gitpagesdocs.png)
깃허브 블로그는 `GitHub Pages`를 이용해서 만든 것인데 GitHub Pages는 GitHub를 통해 호스트 되고 게시되는 퍼블릭 웹페이지이다.  
GitHub Docs 공식문서를 보면 GitHub Pages를 시작하고 실행하는 가장 빠른 방법이 `Jekyll` 테마 선택기를 사용해서 미리 만들어진 테마를 로드하는 것이라고 소개하고 있다.  
그 이후 콘텐츠와 스타일을 수정하는 방식으로 사용하면 된다고 한다.

추가적인 정보는 아래 GitHub Docs에서 확인하면 된다.
>
* [GitHub Pages Docs](https://docs.github.com/ko/pages/getting-started-with-github-pages/about-github-pages)
* [GitHub Pages 시작하기](https://docs.github.com/ko/pages/quickstart)
  

### SSG(Static Site Generator)  

> [Static Site Generator](https://jamstack.org/generators/)

위 사이트를 참고하면 다양한 `Static Site Generator`를 볼 수가 있는데 많이 언급되는 것들을 정리해보면 `Jekyll` `Hexo` `Hugo` 가 있다.  

>#### Jekyll
- Ruby 기반으로 GitHub Docs에서도 언급하고 있으며 가장 많이 사용한다. 또한 Github 내부 엔진이라서 Github Pages에서도 자연스럽게 동작한다. 필자의 블로그도 Jekyll 기반이다.  
#### Hexo   
- 자바스크립트(Node.js) 기반이고 다양한 테마가 있다.  
#### Hugo  
- Golang 기반으로 빌드 속도가 매우 빠른데 테마가 부족하다.    
  
처음부터 끝까지 본인이 만들고자 하는 경우 자신이 사용하는 언어나 특징을 살펴보고 SSG를 골라서 사용하면 될 것 같다.  

### minimal-mistakes theme  
살펴보니 minimal-mistakes 테마를 사용하여 Github Pages를 만들었길래 커스텀할 때 minimal-mistakes 관련 정보가 필요할 것 같아서 찾아보았다.  
minimal-mistakes 관련 docs와 github 링크이다.
>[minimal-mistakes github](https://github.com/mmistakes/minimal-mistakes)  
> [minimal-mistakes Docs](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)

![output](/assets/images/posts_img/etc-cate/minimal.png)   
마음에 드는 스킨을 골라 사용하면 될 것 같다.  

## III. 커스텀 방법과 사용기  
### Editor 선택  
GitHub 블로그에 포스팅을 편리하게 하려면 편집기가 필요하다.  
`Markdown`을 지원하는 editor를 사용하면 된다.  
찾아보니 보통 `vscode`를 사용하는 것 같고 필자는 평소에 `intellij`를 제일 많이 사용하기 때문에 편집기로 `intellij`를 선택했다.  

### 커스텀  
![output](/assets/images/posts_img/etc-cate/plum.png)
블로그 테마를 만드신 분이 올려준 설명에서 _config.yml 부분을 보니 minimal_mistakes_skin 중 plum으로 설정해놓은 것을 보고 블로그 메인 색상을 변경하려면 plum skin에서 수정하면 될 것 같았다.  
![output](/assets/images/posts_img/etc-cate/plum2.png)  
primary-color를 원하는 색상으로 변경했더니 일괄 적용된 것을 확인할 수 있었다.  
  
이외에도 블로그명이나 프로필 이미지는 _config.yml 에서 변경하고 관리할 수 있다.  

### 포스트 방식  
#### 카테고리 생성
먼저 카테고리를 추가하는 방법은 `navigation.yml` 과 `_pages`의 `categories` 부분에서 추가하고 수정해주면 된다.  

![output](/assets/images/posts_img/etc-cate/cate.png)   

#### 이미지 업로드 
벨로그를 쓸 때 보다 불편한 점이 있다면 이미지를 올릴 때인데 이미지 링크를 생성해서 올리거나 assets-images에 추가하여 업로드 해야 한다.  
1. github issues로 주소 생성  

   ![output](/assets/images/posts_img/etc-cate/issue.png)
2. assets에 이미지 저장  

   ![output](/assets/images/posts_img/etc-cate/image.png)
  
형식은 markdown 언어를 사용하고 있으니까 아래와 같이 작성하면 된다.  

``` ![output](이미지 주소)```  
  
필자는 후자의 방식을 사용중인데 필요한 이미지를 캡쳐해서 assets에 추가하여 관리하고 있다. 전에 벨로그를 사용할 때에는 이미지 복사해서 바로 붙여넣기를 했었다.
#### 포스팅  

![output](/assets/images/posts_img/etc-cate/post.png)  

보여질 글의 제목, 짧은 소개, 카테고리 등을 설정해주고 글을 작성하면 된다.  

![output](/assets/images/posts_img/etc-cate/deploy.png)  

포스팅을 하면 완전히 deploy 되는데 30초 ~ 1분 정도 소요된다. deploy가 완전히 끝날 때까지 건드리지 않는 것이 좋다.


## IV. etc.
### 생각 정리  
github 블로그를 사용해보니까 아직까지는 만족스럽다.  
ide를 사용해서 로컬에서도 작성할 수 있고 평소 자주 쓰는 intellij를 통해 수정하고 commit push를 통해 업로드 하는 부분이 매우 마음에 든다.  
이후에는 github 프로필에 github 블로그 최신글 불러오기, github 블로그에 댓글 기능 추가하기 등을 해볼 예정이다.


### 참조
https://docs.github.com/ko/pages/getting-started-with-github-pages/about-github-pages
https://github.com/mmistakes/minimal-mistakes
https://github.com/choiiis/minimal-mistakes-choiiis-customized

