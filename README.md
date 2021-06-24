# [umi0410.github.io](https://umi0410.github.io)
![README_preview.png](static/README_preview.png)

## About this blog

* 글 적는 법
  * Project root에서
  ```bash
  $ hugo new --kind chapter {{*.md_FILE_REL_PATH_ABOUT_/content/}}
  ```
  * 개발용(초안)으로 적는 글은 .Page.Params의 draft: true로 설정 후 `hugo serve -D`를 통해
    draft 상태인 글도 볼 수 있다. 
* 배포 자동화
  * branch:master에서 게시글 작성
  * branch:**에 push할 경우 Github Action을 통해 빌드
  * 빌드된 파일은 branch:gh-pages에 배포~! (Github Page가 gh-pages 브랜치를 바라보기 때문.)
* `/content/page/` 의 `.md` 글들은 weight에 따라 사이드바에 표시된다.
* 좀 헷갈리긴 하는데 Archives 페이지의 제목을 Categories로 변경해서 사용 중이다. Archives가 더 예쁘게 나와서...?

## Customization
themes/hugo-theme-learn 을 바탕으로 몇몇 부분을 수정해서 운영하고있다.
submodule을 통해 원본 theme은 유지하고 `layout` directory를 수정함으로써 오버라이드하거나 커스터마이징 중이다.

* 아이콘 추가
  * https://tablericons.com/ 에서 아이콘 검색 후 `/assets/icons` 에 `stroke=currentColor`만 변경한 뒤 붙여넣는다.
* .gitignore
  * `/resources/_gen`에 자꾸 임시 생성 파일들이 생기는 것 같다. gitignore해줬다.


## Refs and Tips

* [hugo-theme-learn docs](https://learn.netlify.app/en/)

* [Useful short codes](https://learn.netlify.app/en/shortcodes/)

* [Markdown image resizing](https://learn.netlify.app/en/cont/markdown/#resizing-image)

* local에서 disqus가 이용되지 않는 경우
  * [localtest.me란](https://superuser.com/questions/1280827/why-does-the-registered-domain-name-localtest-me-resolve-to-127-0-0-1)
  * trusted domains 에 [localtest.me](localtest.me) 추가하기
* [google search console](https://search.google.com/search-console/sitemaps) 설정
  * static/robots.txt 작성