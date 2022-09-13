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

## 깃허브 페이지 따라하기

1. [Jekyll](https://jekyllrb.com/) 이란?
    1. 루비로 만든 정적 웹사이트 생성기 (Static websites generator)
    2. Template과 Contests 등 다양한 포맷의 텍스트 데이터를 읽어서 웹사이트를 생성해줌
    3. 웹사이트를 운영하기 위한 HTML 지식없이 Markdown 파일을 통해 HTML 파일로 변환해서 서비스
    4. 지킬을 사용하면 GitHub 없이 로컬 컴퓨터에서 블로그 웹호스팅이 가능하나 24시간 컴퓨터를 켜야하고 웹서비스가 에러가 없는지 모니터링해줘야 함
2. [GitHub Pages](https://pages.github.com/)
    1. 정적 웹사이트 호스팅 서비스
    2. GitHub Pages 내부에 Jekyll이 설치되어 있음

## 1. Git Repository 만들기

- [https://pages.github.com/](https://pages.github.com/) 링크를 따라 Repo 생성
- 자신의 깃허브 계정 [이름.github.io](http://이름.github.io) 로 생성

## 2. 로컬서버에서 Jekyll 구동하여 web page 확인

바로 git push 해서 GitHub에 올려도 되지만 너무 많은 commit 남발을 줄이기 위해 로컬서버에서 확인 후 완성작만 올리는 게 좋음.

1. [Ruby](https://rubyinstaller.org/downloads/) 설치
2. Jekyll, Bundler 설치

```yaml
$ gem install jekyll bundler .
$ jekyll -v # version 확인
```

1. [githubId.github.io](http://githubId.github.io) 폴더로 이동
2. jekyll 실행

```yaml
$ bundle exec jekyll serve
```

### 2.1. Jekyll 실행 에러 해결 방법

- bundle install 커맨드 관련 에러 (dependency 문제)

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

[Solved] Gemfile 에 아래 문구 추가 후 bundle install 실행

```ruby
gem "rake", "12.3.3" 
```

- bundle exec 관련 에러 (dependency 문제)

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

[Solved] 역시 Gemfile에 아래 문구 추가

```ruby
gem "jekyll-include-cache"
```

- bundle exec 관련 에러2 (webrick 문제)

```yaml
$ bundle exec jekyll serve
Auto-regeneration: enabled for 'C:/Users/echaseu/Desktop/cha/github/chachaylmo.github.io'
                    ------------------------------------------------
      Jekyll 4.2.2   Please append `--trace` to the `serve` command
                     for any additional information or backtrace.
                    ------------------------------------------------
C:/Ruby31-x64/lib/ruby/gems/3.1.0/gems/jekyll-4.2.2/lib/jekyll/commands/serve/servlet.rb:3:in `require': cannot load such file -- webrick (LoadError)
```

[Solved] 역시 Gemfile에 아래 문구 추가

```ruby
gem "webrick", "~> 1.7"
```

여러가지 구글링을 통해 아래와 같이 미리 추가하는 것이 좋음

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

## 2.2 로컬호스트 접속으로 web page 확인

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

![image](../../assets/images/localhost.png)
***
<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}