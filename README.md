# [umi0410.github.io](https://umi0410.github.io)
![README_preview.png](static/README_preview.png)
* themes/hugo-theme-learn 을 바탕으로 몇몇 부분을 수정해서 운영하고있다.
  * `themes/hugo-theme-learn/layouts/partials/custom-sidebar-profile.html`에 프로필을 추가.
  * `themes/hugo-theme-learn/layouts/partials/menu.html`에서 해당 사이드바 프로필을 import
  * `themes/hugo-theme-learn/statics/css/hugo-theme.css`에서 일부 변경
  * `themes/hugo-theme-learn/statics/css/theme-umi0410.css`에서 custom theme 정의
  * `themes/hugo-theme-learn/statics/css/custom-umi0410.css`에서 custom css 추가
  * submodule을 통해 theme을 깔끔하게 이용하고 싶었지만, customization을 위해서는 수정사항을 적용하려다보니 단순 clone을 이용하게됨.
  * `themes/hugo-theme-learn/layouts/partials/custom-comments.html`에 disqus 관련 설정 추가
    ```html
    {{ if not .Params.noDisqus }}
    {{ template "_internal/disqus.html" . }}
    {{ end }}
    ```
    각각의 페이지에서 parameter로 disqus를 막을 페이지에서는 `noDisqus: true`를 정의해준다.
  * `themes/hugo-theme-learn/layouts/partials/favicon.html`에서 favicon을 `https://emojipedia-us.s3.dualstack.us-west-1.amazonaws.com/thumbs/120/google/241/whale_1f40b.png`로 설정
* 개발은 dev 브랜치에서
* default branch는 master=>dev로 변경했음.
* 배포 자동화
  * branch:dev에서 게시글 작성
  * branch:dev에 push할 경우 Github Action을 통해 빌드
  * 빌드된 파일은 branch:master에 배포~! (Github Page가 master 브랜치를 바라보기 때문.)
* 재미삼아 개발 중~!
* local에서 disqus가 이용되지 않는 경우
  * [localtest.me란](https://superuser.com/questions/1280827/why-does-the-registered-domain-name-localtest-me-resolve-to-127-0-0-1)
  * trusted domains 에 [localtest.me](localtest.me) 추가하기

## Refs

* [hugo-theme-learn docs](https://learn.netlify.app/en/)

* [Useful short codes](https://learn.netlify.app/en/shortcodes/)

* [Markdown image resizing](https://learn.netlify.app/en/cont/markdown/#resizing-image)