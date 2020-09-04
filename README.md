# [umi0410.github.io](https://umi0410.github.io)
![README_preview.png](static/README_preview.png)
* themes/hugo-theme-learn 을 바탕으로 몇몇 부분을 수정해서 운영하고있다.
  * `themes/hugo-theme-learn/layouts/partials/custom-sidebar-profile.html`에 프로필을 추가.
  * `themes/hugo-theme-learn/layouts/partials/menu.html`에서 해당 사이드바 프로필을 import
  * `themes/hugo-theme-learn/statics/css/hugo-theme.css`에서 일부 변경
  * `themes/hugo-theme-learn/statics/css/theme-umi0410.css`에서 custom theme 정의
  * submodule을 통해 theme을 깔끔하게 이용하고 싶었지만, customization을 위해서는 수정사항을 적용하려다보니 단순 clone을 이용하게됨.

* 개발은 dev 브랜치에서
* default branch는 master=>dev로 변경했음.
* 배포 자동화
  * branch:dev에서 게시글 작성
  * branch:dev에 push할 경우 Github Action을 통해 빌드
  * 빌드된 파일은 branch:master에 배포~! (Github Page가 master 브랜치를 바라보기 때문.)
* 재미삼아 개발 중~!

## Refs

* [Hugo theme learn doc](https://learn.netlify.app/en/)

* [Useful short codes](https://learn.netlify.app/en/shortcodes/)

* [Markdown image resizing](https://learn.netlify.app/en/cont/markdown/#resizing-image)