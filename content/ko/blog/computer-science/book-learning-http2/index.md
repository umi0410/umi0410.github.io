---
title: '[Book] 러닝 HTTP/2'
date: 2023-06-11T22:00:00+09:00
weight: 17
categories:
  - Book
  - Network
image: preview.png
---
> _⚠️ 이 글은 **기계 번역** 위주로 번역되었습니다._
> 
> _원문은 [(en) [**Book**] **Learning HTTP/2**](../../../en/blog/computer-science/book-learning-http2)를 참고해주세요._ 

## 요약

★★★★☆ - 적극 추천합니다! 그러나 다른 책이나 기사로 약간 대체할 수 있습니다.

이 책은 HTTP 프로토콜의 진행 상황에 대한 유용한 실제 사례와 명확한 설명을 제공합니다. 또한 HTTP 2와 HTTP 1을 명확하게 비교합니다.

이 책에 대한 링크는 [_Learning HTTP/2_](https://www.oreilly.com/library/view/learning-http2/9781491962435/) 입니다 .

## 감상평

- 이 책에 HTTP 프로토콜의 역사가 포함되어 있다는 점을 높이 평가합니다.
    - 이와 같은 기술 서적에서 내가 가장 중요하게 생각하는 것은 실제로 역사적 맥락을 포함하는 것입니다. 기술의 사양과 특징은 인터넷에 있는 동영상이나 기사를 통해 알 수 있지만 기술의 역사를 이해하는 것은 책을 통해서만 가능합니다.
  여기에는 버전 0.9, 1.0 및 1.1에서 2.0으로 HTTP 프로토콜의 진행 상황이 포함되었습니다.
- HTTP/2 헤더에 대한 자세한 설명과 관련 예제가 매우 유용하다는 것을 알았습니다.
- 책은 읽기가 매우 쉬웠습니다. 느린 읽기 속도에도 불구하고 나는 며칠 안에 그것을 끝낼 수 있었습니다.
- 취업을 준비하는 학생이나 모르거나 무의식적으로 HTTP 2를 사용하는 엔지니어에게 이 책을 적극 추천합니다.
    - 나도 HTTP2에 대해 잘 몰랐지만 이제 이 주제에 대해 어느 정도 자신감을 얻었습니다.
- 나중에 이 책을 몇 번 더 다시 볼 생각입니다!
- 이 책을 통해 HTTP 2에 대해 어느 정도 친숙해질 수 있었지만 솔직히 아직 완전히 이해하지는 못했다.

## 요약

### HTTP 프로토콜의 역사
1. HTTP 0.9의 탄생
    - 제한된 기능을 가진 간단한 프로토콜
    - GET 방식만 지원
    - 헤더가 포함되어 있지 않습니다.
    - 열악한 기능에도 불구하고 HTTP 0.9가 널리 사용되었습니다.
2. HTTP 1.0의 탄생
    - HTTP 0.9 이후 몇 년 후에 개발되었습니다.
    - 헤더, 응답 코드, 리디렉션, 오류, 조건부 요청, 콘텐츠 인코딩 및 압축 및 다양한 방법을 포함하여 HTTP0.9에 비해 크게 향상된 기능을 도입했습니다.
    - 연결을 유지할 수 없습니다.
    - 헤더 Host는 필수가 아닌 선택 사항입니다.
    - 제한된 캐싱 기능.
3. HTTP 1.1의 탄생
    - 20년 넘게 웹 커뮤니케이션을 지배했습니다.
    - 지시문을 사용하여 연결을 유지할 수 있습니다 Connection.
    - 헤더 를 도입하면 Host단일 IP 주소로 여러 웹 서비스를 제공하는 가상 호스팅이 가능합니다.
    - 헤더 Upgrade는 더 높은 수준의 프로토콜에 대한 협상을 허용합니다.
    - 향상된 캐싱 기능
4. HTTP 2.0의 탄생
    - 멀티플렉싱 - 동일한 도메인 이름과 인증서를 사용하여 동일한 대상에 대한 여러 요청에 대해 단일 TCP 연결을 사용할 수 있습니다.
    - 프레이밍 - 전송되는 데이터의 단위. 데이터 전송은 프레임 단위로 발생합니다.
    - 헤더 압축 - HPACK을 사용하여 유사한 헤더를 압축하여 전송을 최적화합니다.

### HTTP 1에서 HTTP 2로 전환

HTTP 1에서 HTTP 2로 서비스를 전환할 때 HTTP 1의 성능 최적화 팁이 HTTP 2의 성능을 저하시킬 수 있는 특정 경우를 고려해야 합니다.

- 더 적은 요청을 위한 파일 통합 패턴 - HTTP 2에서는 1개의 TCP 연결로도 각 파일을 동시에 요청할 수 있기 때문에 파일을 통합할 필요가 없습니다.
- 증가된 동시 연결을 위해 도메인 샤딩을 사용하는 패턴 - HTTP 2에서는 일반적으로 단일 TCP 연결로 충분하므로 도메인 샤딩 패턴이 덜 필요합니다.
- HTTP 쿠키 없이 특정 도메인을 사용하는 패턴 - HTTP 2에서 이러한 특정 도메인은 HPACK이 효과적으로 HTTP 헤더를 잘 압축하기 때문에 효율적이지 않습니다.
- 여러 이미지를 거대한 이미지로 통합한 다음 메모리에서 제자리에서 자르는 것을 의미하는 Spriting - HTTP 2에서는 여러 콘텐츠를 동시에 요청할 수 있으므로 spriting의 필요성이 줄어듭니다.

### HTTP 2에서 사용되는 헤더

HTTP 2의 구성 헤더와 관련된 내용이 너무 방대해서 이 기사에서 이 부분을 건너뛰기로 결정했습니다.

### 기타 내용 관련

기술적으로 TLS는 필수는 아니지만 TLS를 사용하는 것이 좋습니다.
