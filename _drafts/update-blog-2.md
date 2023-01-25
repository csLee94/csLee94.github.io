---
title: Github Pages로 블로그 만들기(feat.Chirpy) [2]
author: csLee94
date: 2023-01-25 00:00:00 +0900
categories: [others]
tags: [github pages, blog, jekyll]
---

> 저는 개발자도 아니고, 더더욱 Front 관련해서는 관련 지식이 매우 부족합니다. 아래 기술하는 내용들은 최대한 구글링해보며, 개인적인 미봉책에 불과합니다. 이 보다 더 좋은 방법을 댓글 통해 알려주시면 정말 좋을 것 같습니다.

[이전 포스트](/posts/update-blog-2.md)에서 기본 설정 및 빈 페이지를 띄우는 것까지 완료했습니다. 그 이후에 설정할만 한 부분들에 대해서 정리해봅니다. 그간 commit history를 요약해보면, 아래와 같이 업데이트를 진행해왔습니다. 

- Update Favicon
- Update Font
- Enable comment using utterance
- Enable GA4 and Google-search-console 
- Enable Hits 
- Make new layout

그럼 하나 하나, 진행해보겠습니다.

<br>

## Update Favicon 
> [Chirpy 공식 문서](https://chirpy.cotes.page/posts/customize-the-favicon/)를 확인해보시면, 굉장히 쉽게 해결하실 수 있습니다.
{: .prompt-tip}

### Generate the favicon
저희 repository 아래 **assets/img/favicons** path를 확인해보시면 하위 폴더에 파일이 굉장히 많습니다. 다른 파비콘 생성 사이트를 이용하기 보단 공식 문서에 안내되어 있는 대로, [Real Favicon Generator](https://realfavicongenerator.net/)에서 <kbd>Select your Favicon image</kbd> 버튼을 눌러 원하는 이미지를 업로드합니다. 그 다음 페이지에서 최하단으로 내리면 <kbd>Generate your Favicons and HTML code</kbd> 버튼을 통해 favicon을 생성합니다.

### Download & Replace
파일을 다운받고 압축을 풀면 여러 파일이 있을텐데, 그 중 `browserconfig.xml`, `site.webmanifest` 파일을 제외하고 *assets/img/favicons** path에 넣어줍니다. 그리고 사이트를 다시 빌드하면 변경된 favicon을 확인할 수 있습니다.

<br>

## Update Font

### Install Font
먼저, **https://fonts.google.com/**에서 변경하고자 하는 font를 검색합니다. 저는 일단은 가장 무난한 Noto Sans Korean을 선택했습니다. [Noto Sans Korean](https://fonts.google.com/noto/specimen/Noto+Sans+KR?query=Noto+Sans+KR&subset=korean&noto.script=Kore) 상세 페이지에서 스크롤을 하단으로 내리다보면, weight별로 선택할 수 있는 창이 나옵니다.

![img](/assets/img/blog-update/img_3.png)
_참고_

위 사진과 같이, 원하는 굵기별로 선택하신 뒤 **Selected family** 탭을 보시면 아래 **@import**라고 되어 있는 부분을 copy해줍니다. 이후 **assets/css/style.scss** 최하단에 붙여넣습니다.

``` scss
...
/* append your custom style below */ 
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@300;500;700&display=swap');
```
{: file='assets/css/style.scss'}

### apply Font style to specific page
다시 페이지를 빌드하면 Noto Sans Korean이 적용된 모습을 확인할 수 있습니다. 다만, 특정 page에서 font가 굵다거나 색상 조정이 필요할 수 있습니다. 그때는 해당 `scss` 파일을 수정해주시면 됩니다.

<kbd>ctrl+shift+c</kbd>를 누르고, 변경을 원하는 영역에 클릭하면 좌측에서 어떤 스타일이 적용되어 있는지 확인할 수 있습니다. 작은 네모친 부분에서 해당 파일과, line number를 확인하고 해당 파일을 찾습니다. 저는 홈화면에 `post-preview` 부분을 조금 더 얇은 Font를 적용시키려고 합니다.

![img](/assets/img/blog-update/img_4.png)
_ctrl+shift+c 누른 후, 원하는 영역 클릭_

확인한대로, home.scss 파일의 107번째 줄에서 font-weight: lighter를 추가해줍니다. 'bolder', 'lighter'는 상대적으로 얇게, 두껍게를 말합니다. 그 밖에도 이와 같은 방법으로 style들을 업데이트 할 수 있습니다.

```scss
...
    .post-content {
      margin-top: 0.6rem;
      margin-bottom: 0.6rem;
      color: var(--post-list-text-color);

      > p {
        /* Make preview shorter on the homepage */
        margin: 0;
        overflow: hidden;
        text-overflow: ellipsis;
        display: -webkit-box;
        -webkit-line-clamp: 2;
        -webkit-box-orient: vertical;
        font-weight: lighter; // 추가
      }
    }
...
```
{: file='_sass/layout/home.scss'}

> css를 전혀 모르는 제가 자잘한 구글링을 통해 적용한 방법입니다. css을 알고 계시거나, 가볍게 알아보시면 보다 쉽고 올바른 방법이 있을 것 같습니다.
{: prompt-warning} 

<br>

## Enable comment using utterance
> 댓글 기능 활성화를 위해서 기존에는 Disqus를 사용했으나, 별도로 가입이 필요한 점 때문에 이번 기회에 `utterance`로 변경하게 됐습니다.

[utterance](https://utteranc.es/)는 Github issues에 설치되는 위젯입니다. 블로그 댓글 기능, wiki page 등에 사용됩니다. 블로그에서 댓글을 남기게 되면, Utteracnce-bot이 [issue search API](https://docs.github.com/ko/rest/search?apiVersion=2022-11-28#search-issues)를 사용해 자동으로 지정해둔 repository의 issue를 생성합니다.

### configuration
[https://github.com/apps/utterances](https://github.com/apps/utterances)에 접속하셔서 <kbd>Install</kbd> 버튼을 눌러줍니다.

![img](/assets/img/blog-update/img_5.png){: .left}
좌측 사진처럼 <kbd>Only select repository</kbd> > <kbd>Select repositories</kbd>에서 저희의 github page 저장소를 선택합니다.

그리고 <kbd>Install</kbd> 버튼을 누르면 Utterance 홈페이지로 다시 돌아오게 됩니다. 돌아온 [홈페이지](https://utteranc.es/)에서 조금 스크롤을 내려보면 **configuration** 영역을 찾을 수 있습니다. **repo:**되어 있는 부분에 다시 저희 github page 저장소를 입력해 줍니다.

그리고 스크롤을 밑으로 내리시다보면, **Enable Utterances** 영역에 script code을 확인하실 수 있습니다. 이 내용을 그대로 복사해서 저희 블로그 소스에 추가해주면 됩니다.

### apply utterance to post page

> 저 같은 경우, `_config.yml`에서 설정했을 때 정상적으로 작동하지 않아 직접 html 소스에 추가했습니다. 
{: .prompt-warning }

저희는 각각 post 마다 댓글 기능을 활성화 시키려고 합니다. 따라서 **_layout/post.html** 파일을 열어 복사한 코드를 붙여넣습니다. html code를 참고해 원하시는 구역에 붙여넣어주세요. 저는 제 post code 상 `post-tail-wrapper` 영역 하단에 붙여 넣었습니다.

```html
...
  </div><!-- .post-tail-bottom -->

</div><!-- div.post-tail-wrapper -->

<!-- Comments -->
<!-- from utterance -->
<div class="comments">
  <script src="https://utteranc.es/client.js" repo="csLee94/csLee94.github.io" issue-term="pathname"
    theme="github-light" crossorigin="anonymous" async>
    </script>
</div>
```
{. file='_layout/post.html'}

그런대, 이렇게만 붙여넣으면 한 가지 문제가 생깁니다. Chirpy 테마는 light/dark theme간 변경을 지원하는데, 만약 이렇게만 붙여넣으면 저희가 블로그를 dark theme로 변경해도 여전히 utterance 영역은 light theme로 남게 됩니다. 

따라서 theme가 변경될 때마다 utterance를 함께 맞춰주어야 하는 코드를 추가해야합니다. 이 부분은 이미 Chirpy 개발자 분들이 **_includes/comments/utterances.html**에 구현해두었습니다. 해당 파일을 복사해 저희 **post.html**에서 위에서 생성한 `div` 태그 안에 붙여넣어줍니다.

```html
...
<div class="comments">
    <script src="https://utteranc.es/client.js" repo="csLee94/csLee94.github.io" issue-term="pathname"
    theme="github-light" crossorigin="anonymous" async>
    </script>
    <!-- select theme depend on light/dark option -->
    <!-- from _includes/comments/utterances.html -->
    <script type="text/javascript">
    $(function () {
      const origin = "https://utteranc.es";
      const iframe = "iframe.utterances-frame";
      const lightTheme = "github-light";
      const darkTheme = "github-dark";
      let initTheme = lightTheme;

      if ($("html[data-mode=dark]").length > 0
        || ($("html[data-mode]").length == 0
          && window.matchMedia("(prefers-color-scheme: dark)").matches)) {
        initTheme = darkTheme;
      }

      addEventListener("message", (event) => {
        let theme;

        /* credit to <https://github.com/utterance/utterances/issues/170#issuecomment-594036347> */
        if (event.origin === origin) {
          /* page initial */
          theme = initTheme;

        } else if (event.source === window && event.data &&
          event.data.direction === ModeToggle.ID) {
          /* global theme mode changed */
          const mode = event.data.message;
          theme = (mode === ModeToggle.DARK_MODE ? darkTheme : lightTheme);

        } else {
          return;
        }

        const message = {
          type: "set-theme",
          theme: theme
        };

        const utterances = document.querySelector(iframe).contentWindow;
        utterances.postMessage(message, origin);
      });

    });
  </script>
</div>
```
{. file='_layout/post.html'}

이제 theme 변경에 따라 utterance 테마가 함께 변경되는지 확인해보시고, 댓글을 달고 저희 git repository의 issue 탭에 정상적으로 생성되면 완료된 것입니다.

<br>

## Enable GA4 and Google-search-console 


<br>

## Enable Hits 

<br>

## Make new layout

<br>

## Wrap up

<br>

## APPENDIX. github actions

