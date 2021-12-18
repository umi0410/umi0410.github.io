---
title: About me
slug: /about
description: 안녕하세요. 백엔드 개발자 Jinsu Park(umi0410)에 대해 소개합니다!
weight: 1
chapter: false
pre: ""
noDisqus: true
menu:
    main: 
        weight: -90
        pre: user
---

## 😄 소개

* **이름**: 박진수
* **학력**: 경희대학교 컴퓨터공학과
* **자기소개**
  
  **Golang**, **Spring**, **클라우드 인프라**, **MSA**, **컨테이너**와 **자동화**, 대규모 트래픽과 확장성을 고려한 **설계** 등에 관심이 많습니다.
  
  어떤 기술을 배우거나 적용시킬 때 **"언제", "왜" 그 기술이 필요한지**에 대해 생각해보는 것을 좋아합니다. 그런 **생각을 자유롭게 공유하는 자리를 사랑**합니다 :)

  특히나 요즘은 대용량 트래픽, 대규모 프로젝트에 대비한 **캐싱**이나 **데이터베이스 설계**, **CQRS**에 관심이 많습니다.

  위와 같은 주제로 **자유롭게 의견을 공유하는 것을 좋아하는 팀원들을 만나 좋은 케미를 내며 개발해나가는 게 꿈**입니다.

---

## 기술 스택 및 관심사

* 깔끔하고 튼튼한 백엔드 구조 및 설계
  * Golang
  * Java Spring Boot
  * MSA, DDD, CQRS, 메시지 큐, 이벤트 드리븐 아키텍쳐
  * TDD, Mocking
  * 테스트 및 배포 자동화
* 컨테이너
* 클라우드 인프라
  * 상황에 맞는 AWS 인프라 설계
  * CI/CD 파이프라인
---

## 💼 경력 사항

### [배틀팡](https://battlepang.com) 개발 리드 [2021.03~2021.12]

![battlepang.drawio.png](/media/battlepang.drawio.png)
![battlepang-preview.png](/media/battlepang_preview.png)

> 짧은 영상을 바탕으로한 대결 서비스 [배틀팡](https://battlepang.com)에서 AWS 인프라 관리 및 Spring boot 위주로 전체 백엔드 개발 중입니다.

* `TDD`를 바탕으로 `Spring Boot` RESTful API 서버 개발
* `JPA`를 이용하며 복잡한 참조 관계로 인한 성능 저하를 다양한 방법으로 해결
  * 조회 시 매번 특정 값을 계산하지 않고 배치 방식로 특정 값의 스냅샷을 저장
  * 배치 방식의 In-Query 또는 페치 조인 이용
  * 불필요한 Eager Loading 제거
* `AWS`를 이용한 클라우드 아키텍쳐 설계 및 관리
  * 평소 쿠버네티스 환경에 익숙했지만 초기 스타트업의 상황에 맞춰 `Elastic Beanstalk` 바탕의 아키텍쳐 설계.
* `Jenkins`와 `Github Action`을 통해 테스트/빌드/배포를 자동화

---

### [Megazone Cloud에서 DevOps 인턴](/blog/megazone-cloud/index/) [2020.04~2020.08]

![megazone.drawio.png](/media/megazone.drawio.png)

* `terraform`을 이용해 AWS 개발 인프라를 IaC로 구축 및 관리
  * terraform을 통해 IaC를 처음 접하게 되었고, 선언적인 설정과 멱등성이 가져다주는 안정성, 중요성을 느끼게 됨.
  * 단순히 짜여진 아키텍쳐를 구축하는 것뿐만 아니라 좀 더 최적화할 수 있는 방안들이나 설계하는 법도 배워갈 수 있었음.
* `Kubernetes`(AWS EKS) 위에서 `마이크로서비스`들을 컨테이너로 관리
* `Spinnaker`, `Github Action`, `Jenkins`를 통한 CI/CD 파이프라인 자동화
  * 다양한 기술을 접해보면서 새로운 기술을 배우는 것에 대한 자신감이 생겼다.
* 팀에서 개발 중인 서비스를 Helm chart로 패키징
  * SaaS 형태로만 이용 가능했던 서비스를 구축형으로 이용할 수 있도록 함
  * Helm chart가 꾸준히 개발되어 후에는 사내의 SaaS도 Helm으로 관리되고 있는 듯함.

## 💻 사이드 프로젝트

### [쿠뮤](https://github.com/search?q=topic%3Akhumu+org%3Akhu-dev&type=Repositories) 프로젝트 리드, 데브옵스 엔지니어, 백엔드 개발 [2020.10~xxxx.xx]

![khumu-argo.png](/media/khumu.png)

> 경희대학교 커뮤니티 서비스 [쿠뮤](https://github.com/search?q=topic%3Akhumu+org%3Akhu-dev&type=Repositories) 개발 중

* 컨테이너 단위의 마이크로서비스 아키텍쳐 설계 및 구축
  * 한 프로젝트에서 `Golang`, Java `Spring boot`, Python `Django` 등의 언어와 프레임워크를 도입하면서 다양한 경험을 해볼 수 있었음
* 대학생 사이드 프로젝트에서 쿠버네티스를 사용하면서도 비용 최적화
* `ArgoCD`와 `Github Action`을 이용한 CI/CD
  * 데브옵스 인턴 때 경험했던 `ArgoCD`와 `Spinnaker` 중 ArgoCD ArgoCD를 사이드 프로젝트에 도입하기로 선택
    * ArgoCD가 좀 더 IaC로 관리하기 쉬웠고, 구축하기도 편했다. GitOps를 바탕으로 컨테이너를 배포하기 아주 편했음.
    * Spinnaker에는 CI와의 연동이나 추가적인 기능들도 많았지만 가볍게 Github Action으로 CI를 하고 있는 우리에겐 필요 없는 기능이 많았다.
* 부분적으로 `TDD`를 도입
  * Golang으로 구현한 댓글 서비스, Spring Boot로 구현한 알림 서비스에서 TDD를 도입.
  * 비교적 Django로 구현한 커뮤니티 메인 서비스는 단순 CRUD 위주였기 때문에 테스트 코드 없이 진행할만 했음.

---

### 눈바디에서 백엔드, 클라우드 인프라 담당 [2021.08~xxxx.xx]

![depromeet-backend.png](/media/depromeet-backend.png)
![depromeet_store.png](/media/depromeet_store.png)

> [개발 동아리 디프만](https://www.depromeet.com/)에서 10기 백엔드 포지션으로 활동했습니다.

- `Golang` 위주의 `MSA`
    - 인증, 인가, 요청 전달 등을 관리하는 API Gateway 서버
    - RESTful API 작업을 위한 API 서버
    - 동영상 및 이미지를 처리하는 Python 서버
- ElasticBeanstalk과 Lambda 위주의 서버 설계
- `TDD`, 유닛테스트, Mocking을 적극적으로 사용

---

## 기타 활동들

* **[개발 동아리 디프만](https://www.depromeet.com/)에서 10기 백엔드로 활동** [2021.08~2021.12]
* **[오픈 소스 컨트리뷰톤: 오픈스택 팀 멘티](/experiences/open-source/open-source-contributhon-2020)** [2020.08~2020.09]
* **한국 AWS Communitiy KRUG의 대학생 그룹 [AUSG](https://ausg.me)에서 3기로 활동 중** [2019.06~]
* **경희대학교 소프트웨어 페스티발 웹/앱 최우수상 수상** [2019.11]



  


