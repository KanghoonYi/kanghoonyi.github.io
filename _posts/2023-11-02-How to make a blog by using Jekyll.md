---
title: How to make a blog by using Jekyll(Jekyll을 사용해서 blog를 만드는 방법)
author: KanghoonYi
name: KanghoonYi(pour)
date: 2023-11-02 19:00:00 +0900
categories: [Guide, Blog]
tags: [blog, github, site]
pin: false
math: false
---
## Overview
Engineer라면 한번쯤 '개인 Blog를 운영해야하나?'라는 생각이 들때가 있습니다.  
이때, 쉽게 blog를 호스팅(hosting)하는 방법중 하나인 [Github Pages](https://pages.github.com/)를 사용하게 되는데, [Jekyll](https://jekyllrb-ko.github.io/)를 통해 그 내용을 구성하게 됩니다.  
여기선, 'Jekyll'의 개념과 이를 이용해서 Blog를 만드는 방법을 다룹니다.

>여기선, MacOS기준으로 작성합니다. 다른 guide는 link로 첨부합니다.
{: .prompt-info }


## Jekyll을 이해하자(Jekyll의 개념)
Jekyll은 규칙에 따라 text를 작성하면, 이 내용을 기반으로 static website를 만들어줍니다.  
프로그래밍 언어는 Ruby로 되어 있으며, [RubyGems](https://rubygems.org/)를 통해 배포됩니다.
>RubyGems는 Ruby에서 사용하는 Package관리자 입니다. 동명의 package배포를 위한 서버도 제공합니다.
{: .prompt-info }

>여기선, RubyGem을 이용한 운영방법을 제시합니다. 이는 앞으로 있을 package update를 쉽게 반영하기 위함입니다.
{: .prompt-info }

## 설치(Installation)
### Jekyll 설치
Jekyll설치에 앞서, Jekyll이 구동되는 환경인 Ruby환경을 만들어야 합니다.  
Jekyll을 설치하는 과정은, 'Ruby를 설치하는 과정'과 'Jekyll을 설치하는 과정'으로 2개의 단계로 진행됩니다.
>원문은 [Jekyll 설치 가이드](https://jekyllrb-ko.github.io/docs/installation/)에서 확인하실 수 있습니다
{: .prompt-tip }

#### Ruby 설치
1. MacOS의 'Command Line Tools'를 설치해야 합니다. 여러 개발환경에서 요구하는 만큼, 이미 설치되어 있을 수 있습니다.
```shell
$ xcode-select --install
```

2. Homebrew를 사용하여 ruby를 설치합니다.
   - rbenv를 통해 여러개의 ruby버젼을 하나의 컴퓨터에서 사용할 수 있게 합니다.
    ```shell
    # Homebrew 설치
    $ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
    # rbenv 와 ruby-build 설치
    $ brew install rbenv
    # 쉘 환경에 rbenv 가 연동되도록 설정
    $ rbenv init # 설치한 rbenv의 initializing을 실행합니다.
    $ rbenv install 3.3.3 # ruby version 3.3.3을 설치합니다.
    $ rbenv global 3.3.3 # 설치한 3.3.3버젼을 PC환경내의 global default로 설정합니다.
    $ ruby -v # 설치된 버젼을 확인합니다.
    ```
   - 1개의 ruby버젼만을 설치하여 사용합니다
    ```shell
    $ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
    $ brew install ruby
    ```

#### Jekyll 설치
1. Jekyll과 Bundler(Gemfile을 통해 package를 관리할 수 있게 해줍니다.)를 설치합니다.
```shell
$ gem install --user-install bundler jekyll
```
2. 이제 shell(Command line)환경에서 gem package에서 제공하는 명령어(여기서는 `bundler`를 사용하기 위함입니다.)를 찾을 수 있도록 연결해줍니다.
```shell
$ echo 'export PATH="$HOME/.gem/ruby/3.3.0/bin:$PATH"' >> ~/.bash_profile
```

    > 위에서 ruby를 '3.3.3'으로 설치했지만, directory는 '3.3.0'으로 생성됩니다. version을 바꾸시려면, 가장 마지막 숫자만 0으로 바꾸신후, 나머지 숫자만 맞춰주시면 됩니다.
    {: .prompt-info }

이렇게 Jekyll설치가 끝났습니다. 이제 Gem과 Bundler를 통해, 언제든 Jekyll의 version을 업데이트할 수 있게 되었습니다.

### Blog Theme 설치하기
Blog의 design을 바꾸고 싶다면, Theme를 설치해야 합니다.
이 Theme또한 RubyGem을 통해 설치할 수 있으며, 업데이트가 빈번히 이루어지는 만큼 이 방법(package manager를 활용한)을 통해 설치하는게 좋습니다.

1. bundle 명령어를 통해, theme를 다운받습니다. 여기선, 'jekyll-theme-chirpy'를 예시로 사용합니다.
```shell
$ bundle add jekyll-theme-chirpy # 'jekyll-theme-chirpy'를 설치하고 Gemfile에 기록합니다.
```

theme설치가 완료되었습니다.

### Theme 업데이트 방법
1. `Gemfile`에 정의되어 있는 `jekyll-theme-chirpy`의 version을 원하는 정책으로 설정합니다. 예를 들면, 아래와 같습니다.
    ```text
   # Gemfile에 입력하세요.
    gem "jekyll-theme-chirpy", "~> 4.0"
    ```
2. bundle명령어를 통해 위에서 수정한 내용을 반영합니다.(패키지를 다시 다운 받습니다)
    ```shell
    $ bundle update jekyll-theme-chirpy
    ```

## Test를 위한 Local 구동
Local에서 website를 구동함으로서 생성되는 페이지를 빠르게 확인할 수 있습니다.
```shell
$ bundle exec jekyll s
```



## Reference

Jekyll을 사용하여 Github Pages 사이트 설정
: [https://docs.github.com/ko/pages/setting-up-a-github-pages-site-with-jekyll](https://docs.github.com/ko/pages/setting-up-a-github-pages-site-with-jekyll)  

Jekyll 설치 방법  
: [https://jekyllrb-ko.github.io/docs/](https://jekyllrb-ko.github.io/docs/)

Jekyll에서 Post 작성방법
: [https://jekyllrb-ko.github.io/docs/posts/](https://jekyllrb-ko.github.io/docs/posts/)

jekyll-theme-chirpy 테마의 Post작성에 필요한 추가 spec
: [https://chirpy.cotes.page/posts/write-a-new-post/](https://chirpy.cotes.page/posts/write-a-new-post/)
