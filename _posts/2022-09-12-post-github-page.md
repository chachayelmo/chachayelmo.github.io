---
published: true
title:  "[Github Page] 깃허브 페이지 사용법"
excerpt: "깃허브 페이지 만들기 A to Z"
categories:
  - Etc
tags:
  - [깃허브, 페이지, GitHub, Etc, Minimal-mistakes]
toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2022-09-12
last_modified_at: 2022-09-14
---

## 깃허브 페이지
1. [Jekyll](https://jekyllrb.com/) 이란?
    - 루비로 만든 정적 웹사이트 제너레이터 (Static websites generator)
    - Template과 Contests 등 다양한 포맷의 텍스트 데이터를 읽어서 웹사이트를 생성
    - 웹사이트를 운영하기 위한 HTML 지식 없이 Markdown 파일을 통해 HTML 파일로 변환해서 서비스
    - Jekyll을 사용하면 GitHub 없이 로컬 컴퓨터에서 블로그 웹호스팅이 가능하나 24시간 컴퓨터를 켜야하고 웹서비스가 에러가 없는지 모니터링 필요
2. [GitHub Pages](https://pages.github.com/)
    - 정적 웹사이트 호스팅 서비스
    - GitHub Pages 내부에 Jekyll이 설치되어 있음

## 1. Git Repository 만들기
- [https://pages.github.com/](https://pages.github.com/) 링크를 따라 Repo 생성
- 자신의 깃허브 계정 [이름.github.io](http://이름.github.io) 로 생성

## 2. 로컬서버에서 Jekyll 구동하여 web page 확인
바로 git push 해서 GitHub에 올려도 되지만 너무 많은 commit 남발을 줄이기 위해 로컬서버에서 확인 후 완성작만 올리는 게 좋음

-  [Ruby](https://rubyinstaller.org/downloads/) 설치
-  Jekyll, Bundler 설치

```yaml
$ gem install jekyll bundler .
$ jekyll -v # version 확인
```

-  [githubId.github.io](http://githubId.github.io) 폴더로 이동
-  jekyll 실행

```yaml
$ bundle exec jekyll serve
```

### 2.1. Jekyll 실행 에러 해결 방법

#### 2.1.1. bundle install 커맨드 관련 에러 (dependency 문제)

```yaml
$ bundle install

[!] There was an error parsing `Gemfile`:
[!] There was an error while loading `minimal-mistakes-jekyll.gemspec`: No such file or directory - git ls-files -z. Bundler cannot continue.

 #  from C:/Users/echaseu/Desktop/cha/github/chachaylmo.github.io/minimal-mistakes-jekyll.gemspec:14
 #  -------------------------------------------
 #    spec.add_development_dependency "rake", ">= 12.3.3"
 >  end
 #  # coding: utf-8
 #  -------------------------------------------
. Bundler cannot continue.

 #  from C:/Users/echaseu/Desktop/cha/github/chachaylmo.github.io/Gemfile:2
 #  -------------------------------------------
 #  gemspec
 >  gem 'rake', '12.3.1' #  source "https://rubygems.org"
 #  -------------------------------------------
```

*[Solved]*
    - Gemfile 에 아래 문구 추가 후 bundle install 실행

```ruby
gem "rake", "12.3.3" 
```

#### 2.1.2. bundle exec 관련 에러 (dependency 문제)

```yaml
$ bundle exec jekyll serve
Configuration file: C:/Users/echaseu/Desktop/cha/github/chachaylmo.github.io/_config.yml
To use retry middleware with Faraday v2.0+, install `faraday-retry` gem
  Dependency Error: Yikes! It looks like you don't have jekyll-include-cache or one of its dependencies installed. In order to use Jekyll as currently configured, you'll need to install this gem. If you've run Jekyll with `bundle exec`, ensure that you have included the jekyll-include-cache gem in your Gemfile as well. The full error message from Ruby is: 'cannot load such file -- jekyll-include-cache' If you run into trouble, you can find helpful resources at https://jekyllrb.com/help/!
                    ------------------------------------------------
      Jekyll 4.2.2   Please append `--trace` to the `serve` command
                     for any additional information or backtrace.
                    ------------------------------------------------
```

*[Solved]*
    - Gemfile에 아래 문구 추가

```ruby
gem "jekyll-include-cache"
```

#### 2.1.3. bundle exec 관련 에러2 (webrick 문제)

```yaml
$ bundle exec jekyll serve
Auto-regeneration: enabled for 'C:/Users/echaseu/Desktop/cha/github/chachaylmo.github.io'
                    ------------------------------------------------
      Jekyll 4.2.2   Please append `--trace` to the `serve` command
                     for any additional information or backtrace.
                    ------------------------------------------------
C:/Ruby31-x64/lib/ruby/gems/3.1.0/gems/jekyll-4.2.2/lib/jekyll/commands/serve/servlet.rb:3:in `require': cannot load such file -- webrick (LoadError)
```

*[Solved]*
    - Gemfile에 아래 문구 추가

```ruby
gem "webrick", "~> 1.7"
```
  
<br/>여러가지 구글링을 통해 아래와 같이 미리 추가하는 것이 좋음

```yaml
source "https://rubygems.org"
gem "jekyll"
gem "jekyll-paginate"
gem "jekyll-sitemap"
gem "jekyll-gist"
gem "jekyll-feed"
gem "jemoji"
gem "jekyll-include-cache"
gem "rake", "12.3.3"
```

### 2.2 로컬호스트 접속으로 web page 확인

```yaml
$ bundle exec jekyll serve
Configuration file: C:/Users/echaseu/Desktop/cha/github/chachaylmo.github.io/_config.yml
To use retry middleware with Faraday v2.0+, install `faraday-retry` gem
            Source: C:/Users/echaseu/Desktop/cha/github/chachaylmo.github.io
       Destination: C:/Users/echaseu/Desktop/cha/github/chachaylmo.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
       Jekyll Feed: Generating feed for posts
                    done in 0.797 seconds.
 Auto-regeneration: enabled for 'C:/Users/echaseu/Desktop/cha/github/chachaylmo.github.io'
    Server address: **http://127.0.0.1:4000**
```

아래와 같이 로컬호스트에 접속하여 웹페이지 확인이 가능

![localhost.png](../../assets/images/localhost.png)

## 3. posts 폴더에 글 등록

포스트 글 형태는 _posts 폴더에 md 파일 확장자를 사용하면 Jekyll 이 포스트 글을 인식하여 md를 html로 변환하여 웹에 보여줌.

```bash
# _posts 폴더 및 md 파일 생성
$ mkdir _posts
$ cd _posts
$ vim 2022-09-12-first-post.md
```

```yaml
# 2022-09-12-first-post.md 파일 수정
---
published: true
title:  "[GitHub Page] 깃허브 페이지 따라하기"
excerpt: "깃허브 페이지 만들기 "

categories: 
  - GitHub Page
tags:
  - [Github, Page]

toc: true
toc_sticky: true

date: 2022-09-12
last_modified_at: 2022-09-12
---

# GitHub page
something
```

## 4. _config.yml 파일 수정

_config.yml 파일은 환경설정을 위한 템플릿

### 4.1. 가장 기본적인 구성: 웹사이트에 대한 주소 설정 및 이름 설정

```yaml
title                    : "나의 공부일지"
name                     : "Chachaylmo"
description              : "끄적끄적"
url                      : "https://chachaylmo.github.io"
baseurl                  : "" # the subpath of your site, e.g. "/blog"
repository               : "chachaylmo/chachaylmo.github.io"
```

### 4.2. teaser와 logo 파일

```yaml
teaser                   : # path of fallback teaser image, e.g. "/assets/images/500x300.png"
logo                     : # path of logo image to display in the masthead, e.g. "/assets/images/88x88.png"
```

### 4.3. 블로그 댓글 기능: 필자는 disqus를 사용

```yaml
comments:
  provider               : "disqus" # false (default), "disqus", "discourse", "facebook", "staticman", "staticman_v2", "utterances", "giscus", "custom"
  disqus:
    shortname            : "chachaylmo" # https://help.disqus.com/customer/portal/articles/466208-what-s-a-shortname-
  discourse:
    server               : # https://meta.discourse.org/t/embedding-discourse-comments-via-javascript/31963 , e.g.: meta.discourse.org
  facebook:
    # https://developers.facebook.com/docs/plugins/comments
    appid                :
    num_posts            : # 5 (default)
    colorscheme          : # "light" (default), "dark"
  utterances:
    theme                : # "github-light" (default), "github-dark"
    issue_term           : # "pathname" (default)
  giscus:
    repo_id              : # Shown during giscus setup at https://giscus.app
    category_name        : # Full text name of the category
    category_id          : # Shown during giscus setup at https://giscus.app
    discussion_term      : # "pathname" (default), "url", "title", "og:title"
    reactions_enabled    : # '1' for enabled (default), '0' for disabled
    theme                : # "light" (default), "dark", "dark_dimmed", "transparent_dark", "preferred_color_scheme"
  staticman:
    branch               : # "master"
    endpoint             : # "https://{your Staticman v3 API}/v3/entry/github/"
```

### 4.4. 오픈그래프 이미지 등록

```yaml
og_image                 : # Open Graph/Twitter default site image
```

### 4.5. 구글 anlytics 등록

```yaml
# Analytics
analytics:
  provider               : false # false (default), "google", "google-universal", "google-gtag", "custom"
  google:
    tracking_id          :
    anonymize_ip         : # true, false (default)
```

### 4.6. 사이트 저자 소개

```yaml
# Site Author
author:
  name             : "공부하는 챠챠"
  avatar           : # path of avatar image, e.g. "/assets/images/bio-photo.jpg"
  bio              : "Today I Learned!"
  #location         : "Seoul"
  links:
    - label: "Email"
      icon: "fas fa-fw fa-envelope-square"
      url: "mailto:chachaylmo@email.com"
    - label: "GitHub"
      icon: "fab fa-fw fa-github"
      url: "https://github.com/chachaylmo"
```

### 4.7. 사이트 저자 소개 - footer

```yaml
# Site Footer
footer:
  links:
    - label: "GitHub"
      icon: "fab fa-fw fa-github"
      url: "https://github.com/chachaylmo"
```

### 4.8. 블로그 표시 방법 설정

```yaml
# Outputting
permalink: /:categories/:title/
paginate: 5 # amount of posts to show
paginate_path: /page:num/
timezone: # https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
```

### 4.9. post, page 기본 설정

```yaml
# Defaults
defaults:
  # _posts
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: true
      read_time: true
      comments: true
      share: true
      related: true
      idebar_main: true
```

<br/>이외에도 많은 사용법이 있으니 필요한 부분을 알아보면 좋을 것 같음

## 5. 페이지 등록

```bash
# _pages
$ mkdir _pages
$ cd _pages
$ vim 404.md
```

```markdown
---
title: "Page Not Found"
excerpt: "Page not found. Your pixels are in another canvas."
permalink: /404.html
author_profile: false
---

Page Not Found.  
Please go back to <a href="https://chachaylmo.github.io">home</a>.

<script>
  var GOOG_FIXURL_LANG = 'en';
  var GOOG_FIXURL_SITE = '{{ site.url }}'
</script>
<script src="https://linkhelp.clients.google.com/tbproxy/lh/wm/fixurl.js">
</script>
```

404.md 파일에 대한 예제이며, GitHub Pages에서 주소를 찾을 수 없을 경우 기본으로 보여주는 페이지임

## 6. 메뉴 등록

### 6.1. navigation.yml 파일 수정

_data/navigation.yml 파일에 등록하면 특정 페이지로 이동하는 버튼을 만들 수 있음

```markdown
# main links
main:
  - title: "Home"
    url: /index.html
  - title: "Categories"
    url: /categories/
```

포스트한 글에 categories, tags를 지정하면 navigation.yml 파일에서 특정 단어를 검색하여 관련 정보를 볼 수 있음

```markdown
# 2022-09-12-first-post.md 파일 수정
---
published: true
title:  "[GitHub Page] 깃허브 페이지 따라하기"
excerpt: "깃허브 페이지 만들기 "

**categories: 
  - GitHub Page
tags:
  - [Github, Page]**

toc: true
toc_sticky: true

date: 2022-09-12
last_modified_at: 2022-09-12
---

# GitHub page
something
```

### 6.2. Yaml에 Tags, categories 링크 연결될 페이지 생성

_pages 밑에 아래와 같이 생성

 

```bash
$ cd _pages
$ vim category-archive.md
$ vim tag-archive.md
```

```yaml
# category-archive.md
---
title: "Posts by Category"
layout: categories
permalink: /categories/
author_profile: true
sidebar_main: true
---

# tag-archive.md
---
title: "Posts by Tag"
permalink: /tags/
layout: tags
author_profile: true
---
```

## 참고

[https://mmistakes.github.io/minimal-mistakes/](https://mmistakes.github.io/minimal-mistakes/)
[https://jekyllrb-ko.github.io/](https://jekyllrb-ko.github.io/)
[https://devinlife.com/](https://devinlife.com/howto/)
[https://ansohxxn.github.io/](https://ansohxxn.github.io/)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}