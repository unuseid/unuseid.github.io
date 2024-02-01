---
title: "Jekyll 를 활용한 블로그 생성(with M1 mac, Minimal Mistake)"
categories:
  - Dev
tags:
  - Ruby
  - Jekyll
  - Blog
---

Github Pages 에 나만의 Jekyll 블로그를 생성해보자

## Jekyll 테마 선정,

- Todo

## Github pages 생성.

- Todo

## 여러버전의 Ruby 를 다루기위해 rbenv 설치.

jekyll 는 Ruby 를 활용해 작성됐다. mac 에는 기본 ruby 가 설치 돼 있을 수 있다. 또한 환경에 따라 다른 버전의 Ruby 사용이 용이하도록 rbenv 를 활용하는것이 좋음.

zshrc 에 아래 내용 추가.

- rbenv Path 잡기.
- rbenv 가 openssl3(기본) 을 바라볼때 문제가 생길 수 있으므로 openssl@1.1을 보도록 수정.

```shell
# rbenv path
export PATH={$Home}/.rbenv/bin:$PATH && \
export RUBY_CONFIGURE_OPTS="--with-openssl-dir=$(brew --prefix openssl@1.0)"
export CONFIGURE_OPTS="--with-openssl-dir=$(brew --prefix openssl@1.0)"
eval "$(rbenv init -)"

#맥에서 기본적으로 제공하고 있는 LibreSSL이 기본 ssl로 설정되어 있는데 내가 설치한 openssl로 설정
export PATH="/usr/local/opt/openssl@1.1/bin:$PATH"

#LDFLAGS는 LD 링커의 플래그, CPPFLAGS는 C 전처리기의 플래그로 주로 Makefile에서 사용된다.
export LDFLAGS="-L/usr/local/opt/openssl@1.1/lib"
export CPPFLAGS="-I/usr/local/opt/openssl@1.1/include"
```

## 아래 명령어로 설치 확인

```shell
curl -fsSL https://github.com/rbenv/rbenv-installer/raw/main/bin/rbenv-doctor | bash

Checking for `rbenv' in PATH: /opt/homebrew/bin/rbenv
Checking for rbenv shims in PATH: OK
Checking `rbenv install' support: /opt/homebrew/bin/rbenv-install (ruby-build 20211203)
Counting installed Ruby versions: none
  There aren't any Ruby versions installed under `/Users/antran/.rbenv/versions'.
  You can install Ruby versions like so: rbenv install 3.0.3
Checking RubyGems settings: OK
Auditing installed plugins: OK
```

## 설치가능한 버전 루비 검색, ruby 3 설치.

```shell
$ rbenv install -l
3.0.6
3.1.4
3.2.3
3.3.0
jruby-9.4.5.0
mruby-3.2.0
picoruby-3.0.0
truffleruby-23.1.2
truffleruby+graalvm-23.1.2

$ rbenv install 3.0.6
```

###### \*\* m1 맥의 경우 Ruby 호환성이 맞지 않는 경우가 발생하기도 함. 가능한 한. 'rbenv install -l' 명령어 결과에 있는 stable version 을 선정하며. 될수 있으면 최신버전은 지향.

## jekyll 가 요구하는 bundler 버전 확인 및 설치

Gemfile.lock 파일의 마지막 부분에 써있다.

```shell
$ tail Gemfile.lock
  x86_64-darwin
  x86_64-linux

DEPENDENCIES
  github-pages
  jekyll-include-cache
  webrick (~> 1.8)

BUNDLED WITH
   2.5.4
```

Bundler 설치.

```shell
$ gem install bundle -v '2.5.4'
```

Bundler 종속성 설치.

```shell
$ bundle init
```

## 서비스 실행(webrick trouble shoot)

```shell
$ bundle exec jekyll serve
Configuration file: /Users/naseongdae/myProject/unuseid.github.io/_config.yml
To use retry middleware with Faraday v2.0+, install `faraday-retry` gem
            Source: /Users/naseongdae/myProject/unuseid.github.io
       Destination: /Users/naseongdae/myProject/unuseid.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
      Remote Theme: Using theme mmistakes/minimal-mistakes
       Jekyll Feed: Generating feed for posts
   GitHub Metadata: No GitHub API authentication could be found. Some fields may be missing or have incorrect data.
                    done in 3.482 seconds.
 Auto-regeneration: enabled for '/Users/naseongdae/myProject/unuseid.github.io'
bundler: failed to load command: jekyll (/Users/naseongdae/.rbenv/versions/3.0.6/bin/jekyll)
/Users/naseongdae/.rbenv/versions/3.0.6/lib/ruby/gems/3.0.0/gems/jekyll-3.9.3/lib/jekyll/commands/serve/servlet.rb:3:in `require': cannot load such file -- webrick (LoadError)
```

에러문구의 'cannot load such file -- webrick (LoadError)' 에서 webrick 을 로드하는데 문제 발생함을 볼 수 있음. 종속성을 Gemfile.lock에 추가해준다

```shell
$ bundle add webrick
$ bundle update
```

다시한번 실행, 성공

```shell
$ bundle exec jekyll serve
Configuration file: /Users/naseongdae/myProject/unuseid.github.io/_config.yml
To use retry middleware with Faraday v2.0+, install `faraday-retry` gem
            Source: /Users/naseongdae/myProject/unuseid.github.io
       Destination: /Users/naseongdae/myProject/unuseid.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
      Remote Theme: Using theme mmistakes/minimal-mistakes
       Jekyll Feed: Generating feed for posts
   GitHub Metadata: No GitHub API authentication could be found. Some fields may be missing or have incorrect data.
                    done in 3.895 seconds.
 Auto-regeneration: enabled for '/Users/naseongdae/myProject/unuseid.github.io'
    Server address: http://127.0.0.1:4000
  Server running... press ctrl-c to stop.
```

## 자동 새로고침으로 실행하기

코드 수정시

```shell
$ bundle exec jekyll serve --livereload

http://127.0.0.1:4000 에서 jekyll 서버 확인 가능.
```

## 테마적용

\_config.yaml 파일 최상단에 아래 내용 추가.(theme 혹은 remote_theme선택가능, theme 설정시 github pages가 빌드오류를 뿜을 수 있음. remote_theme를 선택하자)

```shell
remote_theme: "mmistakes/minimal-mistakes@4.24.0"
```

참고
https://frhyme.github.io/blog/install_jekyll_again/
https://antran.app/2021/m1_mac_part2/
