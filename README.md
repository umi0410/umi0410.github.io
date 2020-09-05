# [umi0410.github.io](https://umi0410.github.io)
![README_preview.png](static/README_preview.png)

## About this blog

* 글 적은 법
  * Project root에서
  ```bash
  $ hugo new --kind chapter {{*.md_FILE_REL_PATH_ABOUT_/content/}}
  ```

* 배포 자동화
  * branch:master에서 게시글 작성
  * branch:**에 push할 경우 Github Action을 통해 빌드
  * 빌드된 파일은 branch:gh-pages에 배포~! (Github Page가 gh-pages 브랜치를 바라보기 때문.)
* 재미삼아 개발 중~!


## Customization
themes/hugo-theme-learn 을 바탕으로 몇몇 부분을 수정해서 운영하고있다. submodule을 통해 theme을 깔끔하게 이용하고 싶었지만, customization을 위해서는 수정사항을 적용하려다보니 단순 clone을 이용하게됨.

* `themes/hugo-theme-learn/layouts/partials/` 에서 몇몇 html 변경
  * `custom-sidebar-profile.html`에 프로필을 추가.
  * `menu.html`에서 해당 커스텀 사이드바 프로필을 import
  * `custom-comments.html`에 disqus 관련 설정 추가
    ```html
    {{ if not .Params.noDisqus }}
    {{ template "_internal/disqus.html" . }}
    {{ end }}
    ```
    각각의 페이지에서 parameter로 disqus를 비활성화하려면 .Page.Params로 `noDisqus: true`를 정의해준다.
  * `favicon.html`에서 favicon을 `https://emojipedia-us.s3.dualstack.us-west-1.amazonaws.com/thumbs/120/google/241/whale_1f40b.png` 로 설정

* `themes/hugo-theme-learn/statics/css/` 에서 몇몇 css 변경
  * `theme-umi0410.css` Custom theme 정의
    * 기본적인 테마 색상 변경
    * \<code\>에 대한 색상 변경
  * `custom-umi0410.css` 에서 부가적이고 잡다한 css 변경. 테마 자체에 대한 것 외의 css 수정사항은 거의 이곳에 위치.
    * 페이지 헤더의 브레드크럼의 토글되는 Table of Contents 부분인 `.progress` 에대한 수정

* `themes/hugo-theme-learn/layouts/shortcodes/` 을 변경
  * `toc.html`을 이용해 Table of Contents 기능을 추가함.

## Refs and Tips

* [hugo-theme-learn docs](https://learn.netlify.app/en/)

* [Useful short codes](https://learn.netlify.app/en/shortcodes/)

* [Markdown image resizing](https://learn.netlify.app/en/cont/markdown/#resizing-image)

* local에서 disqus가 이용되지 않는 경우
  * [localtest.me란](https://superuser.com/questions/1280827/why-does-the-registered-domain-name-localtest-me-resolve-to-127-0-0-1)
  * trusted domains 에 [localtest.me](localtest.me) 추가하기
* [google search console](https://search.google.com/search-console/sitemaps) 설정
  * static/robots.txt 작성