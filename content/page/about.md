---
title: About me
slug: /about
description: 안녕하세요. Jinsu Park(umi0410)에 대해 소개합니다!
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
  
  Golang, Cloud services, MSA, 컨테이너와 자동화, 깔끔한 아키텍쳐에 관심이 많습니다.
  
  어떤 기술을 배우거나 적용시킬 때 "언제", "왜" 그 기술이 필요한지에 대해 생각해보는 것을 좋아합니다.
  
  특히나 요즘은 Golang을 이용한 Go스러운 개발과 Messaging Service에 관심이 많습니다. 
  
  진행 중인 프로젝트에서는 MSA와 K8s, ArgoCD, Github Action, Golang, Python등을 적용하며 서비스 설계와 다양한 기술들을 공부 중입니다.

---

## 관심사

* 깔끔하고 튼튼한 백엔드 구조 및 설계
  * TDD, Mocking
  * 테스트 및 배포 자동화
  * Golang
  * Java Spring Boot
  * 마이크로서비스 아키텍쳐
  * 메시지 큐
* 컨테이너
* 클라우드 인프라
  * 상황에 맞는 AWS 인프라 설계
  * CI/CD 파이프라인
  
---

## 📅 활동 내용

### [배틀팡](https://battlepang.com) 백엔드 개발 [2021.03~xxxx.xx]

![battlepang-preview.png](/media/battlepang-preview.png)

> 짧은 영상을 바탕으로한 대결 서비스 [배틀팡](https://battlepang.com)에서 AWS 인프라 관리 및 Spring boot 위주로 전체 백엔드 개발 중입니다.

* `Spring Boot`를 통한 백엔드 개발
* `TDD`를 도입
  * 프론트엔드 구현 없이 백엔드만으로도 기능을 테스트할 수 있어야했고 테스트 코드를 적극 활용해 테스트를 자동화
  * 동영상 제공 서비스(Vimeo), 푸시 알림(FCM, Firebase Cloud Messaging)과 같은 external한 기능들은 필요한 경우에만 테스트하고 일반적인 비즈니스 로직을 Test할 때에는 `Mocking`을 적용해서 간단히 테스트
* `AWS`를 이용한 클라우드 아키텍쳐 설계 및 관리
  * 평소 쿠버네티스 환경에 익숙했지만 초기 스타트업의 상황에 맞춰 `Elastic Beanstalk` 바탕의 아키텍쳐 설계.
  * Cloudwatch의 기본 metric과 Cloudwatch agent를 이용한 추가적인 metric을 이용해 `Cloudwatch` Dashboard를 작성하고 모니터링
* `Jenkins`를 통해 배포를 자동화
  * 평소 사이드 프로젝트에서는 간단히 Github action을 이용했지만 실무용으로서는 Jenkins가 좋을 것 같아 Jenkins를 선택.
    * 배포 및 작업 로그가 한 눈에 확인됨
    * 다양한 캐싱이 기본적으로 가능
* (예정) 결제 시스템 개발, 결제 오류에 대한 취소 로직 개발

---

### [쿠뮤](https://github.com/search?q=topic%3Akhumu+org%3Akhu-dev&type=Repositories) 프로젝트 리드, 데브옵스 엔지니어, 백엔드 개발 [2020.10~xxxx.xx]

![khumu-argo.png](/media/khumu-argo.png)
> 경희대학교 커뮤니티 서비스 [쿠뮤](https://github.com/search?q=topic%3Akhumu+org%3Akhu-dev&type=Repositories) 개발 중

* 컨테이너 단위의 마이크로서비스 아키텍쳐 설계 및 구축
  * 한 프로젝트에서 `Golang`, Java `Spring boot`, Python `django` 등의 언어와 프레임워크를 도입하면서 다양한 경험을 해볼 수 있었음
* 대학생 사이드 프로젝트에서 쿠버네티스를 사용하면서도 비용 최적화
* `ArgoCD`와 `Github Action`을 이용한 CI/CD
  * 데브옵스 인턴 때 경험했던 `ArgoCD`와 `Spinnaker` 중 ArgoCD ArgoCD를 사이드 프로젝트에 도입하기로 선택
    * ArgoCD가 좀 더 IaC로 관리하기 쉬웠고, 구축하기도 편했다. GitOps를 바탕으로 컨테이너를 배포하기 아주 편했음.
    * Spinnaker에는 CI와의 연동이나 추가적인 기능들도 많았지만 가볍게 Github Action으로 CI를 하고 있는 우리에겐 필요 없는 기능이 많았다.
* 부분적으로 `TDD`를 도입
  * Golang으로 구현한 댓글 서비스, Spring Boot로 구현한 알림 서비스에서 TDD를 도입.
  * 비교적 Django로 구현한 커뮤니티 메인 서비스는 단순 CRUD 위주였기 때문에 테스트 코드 없이 진행할만 했음.

---

### [Megazone Cloud에서 DevOps 인턴](/blog/megazone-cloud/index/) [2020.04~2020.08]

* `terraform`을 이용해 AWS 개발 인프라를 IaC로 구축 및 관리
  * terraform을 통해 IaC를 처음 접하게 되었고, 선언적인 설정과 멱등성이 가져다주는 안정성, 중요성을 느끼게 됨.
  * 단순히 짜여진 아키텍쳐를 구축하는 것뿐만 아니라 좀 더 최적화할 수 있는 방안들이나 설계하는 법도 배워갈 수 있었음.
* `Kubernetes`(AWS EKS) 위에서 `마이크로서비스`들을 컨테이너로 관리
* Spinnaker, Github Action, Jenkins를 통한 CI/CD 파이프라인 자동화
  * 다양한 기술을 접해보면서 새로운 기술을 배우는 것에 대한 자신감이 생겼다.
* 팀에서 개발 중인 서비스를 Helm chart로 패키징
  * SaaS 형태로만 이용 가능했던 서비스를 구축형으로 이용할 수 있도록 함
  * Helm chart가 꾸준히 개발되어 후에는 사내의 SaaS도 Helm으로 관리되고 있는 듯함.

---

### 기타 활동들
* **[오픈 소스 컨트리뷰톤: 오픈스택 팀 멘티](/experiences/open-source/open-source-contributhon-2020)** [2020.08~2020.09]
* **한국 AWS Communitiy KRUG의 대학생 그룹 [AUSG](https://ausg.me)에서 3기로 활동 중** [2019.06~]
* **경희대학교 소프트웨어 페스티발 웹/앱 최우수상 수상** [2019.11]



  


