---
title: "Megazone Cloud DevOps"
menuTitle: "Megazone Cloud DevOps 인턴"
date: 2020-09-04T12:46:54+09:00
chapter: false
pre: "<b></b>"
weight: 1
description: |
  이번에 메가존 클라우드에서 데브옵스 인턴으로 근무를 하게되었던 박진수라고 합니다.
  너무 좋은 팀원들과 많은 경험을 하며 단시간에 성장할 수 있었고,
  일하는 동안 매 순간 순간이 너무 행복했었기에 이렇게 인턴 후기를 작성해봅니다.
---
{{%toc%}}

![mzc_logo](/activities/megazone-cloud/mzcloud-logo.png?width=200px)

안녕하세요. 이번에 **메가존 클라우드**에서 **데브옵스 인턴**으로 근무를 하게되었던 **박진수**라고 합니다.
너무 좋은 팀원들과 많은 경험을 하며 단시간에 성장할 수 있었고, 일하는 동안 매 순간 순간이 너무
행복했었기에 이렇게 인턴 후기를 작성해봅니다. 한 학기를 쉬고 2020.04.13~2020.08.31 까지 인턴으로 근무 했고,
다시 2020-2학기부터는 학교 복학을 하게되었습니다.
기술적인 얘기는 이곳 저곳에 많으니 항상 제가 가장 중요시하는
**"느낀 점"** 과 **"배운 점"** 을 위주로 적어보겠습니다!!
글은 위의 목록의 순서로 진행해보겠습니다.

## 지원동기

저는 AWS와 배포에 관해 흥미가 있어 종종 [AWSKRUG](https://awskrug.github.io/) 라는 그룹의 소모임에 가서 핸즈온이나 세미나에
참여하곤 했는데요. 그곳에서 저희 팀원들을 만나게 되었고, 그것이 인연이 되어
채용으로까지 이어질 수 있었습니다.

평소 [AUSG](https://velog.io/@ausg) 라는 대학생 AWS 사용자 모임의
일원으로서 참여하며 `AWS` 를 이용한 클라우드 인프라, `CI/CD` 파이프라인 구축을 통한 자동화, `컨테이너` 등에
관심이 많았는데, 마침 메가존 클라우드의 저희 CloudOne 팀에서는 팀의 모든 마이크로서비스 및 기타 서비스들을
`EKS`라는 `AWS`의 Managed Kubernetes Cluster Service 위에 자동화시켜 배포를 하고있었고,
개발 인프라를 이전하려던 참이었기에 저의 관심사와 향후 목적에 잘 부합했습니다.

## Megazone Cloud: CloudOne Team?

> [Megazone Cloud](https://www.megazone.com/): 국내 최초 & 최대 AWS 프리미어 컨설팅 파트너, AWS 컨설팅, 구축, 운영 및 빌링 서비스 제공. 메가존클라우드는 2009년부터 클라우드를 차세대 핵심 사업으로 성장시키며 ‘클라우드 이노베이터(Cloud Innovator)’로서 고객님들의 클라우드 전환의 과정마다 최선의 선택을 하실 수 있도록 다양한 서비스를 제공하고 있습니다. - 출처(https://www.megazone.com/)
<!-- ![spaceone_preview_1](spaceone_preview_1.png) -->
![spaceone_preview_2](spaceone_preview_2.png)
<p align="center">SpaceONE Preview</p>
메가존 클라우드에 대한 설명은 위와 같고, 제가 근무했던 CloudOne Team에 대해 간단히 소개해보겠습니다.
CloudOne 팀은 SpaceONE 이라는 Multi Cloud 에 대한 오픈소스 CMP(Cloud Management Platform)를 개발하는 팀입니다.
AWS, Azure, GCP, IDC에 대한 다양한 관리, 모니터링을 지원하는 것이 목표이고, 빠르게 발전되어나가고있습니다! 
주요 내용들은 아래와 같은 키워드를 갖고 있습니다.

* `AWS` `EKS`를 이용한 `Kubernetes` 환경, 마이크로서비스 아키텍쳐
* Composition API를 이용하는 최신 `Vue` 기술
* `Terraform`을 통한 `IaC`
* 자동화된 `CI/CD`, 다양한 배포 전략, 인프라 `모니터링`
* HTTP2를 이용한 `gRPC` API

## 진행했던 업무들

> 링크 형식이므로 링크를 눌러 읽어주시면 됩니다.

* [2020.04: Stargate 라는 개발 인프라를 구축](stargate-infra)
  * terraform 을 통한 개발 인프라 구축
  * EKS 클러스터에 jenkins, spinnaker, grafana 등등의 개발 도구들을 배포함
  * 종종 발생하는 장애 상황을 멘토님과 트러블슈팅함
* [2020.05: 개발 CI/CD 파이프라인 구축](ci-cd-pipeline)
  * Experiment 환경에 배포 => Test, CI => Development 환경에 배포를 자동화
  * Spinnaker와 Jenkins, Github Action을 이용한 자동화 파이프라인 구축
* [2020.06: Argo Project들을 PoC](argo-poc)
  * argo cd, argo(workflow), argo-event 등의 다양한 프로젝트에 대한 PoC를 진행
* [2020.07: spaceone-helm Helm3 Chart 개발](spaceone-helm)
  * 개별로 배포되고 운영되던 우리 팀의 서비스인 SpaceONE을 패키지로 배포할 수 있게해주는 Helm Chart를 개발
* [2020.08: spacectl 설계, 개발 참여](spacectl)
  * 우리 팀의 서비스인 SpaceONE에 대한 API 작업을 수행하는 CLI 도구에 대한 설계와 개발에 참여함.

## 느낀 점

그 동안 혼자 개발 및 평소 관심분야였던 클라우드, 컨테이너 등등의 주제로 공부해왔었는데,
과연 이 내용들이 정말 실무에 도움이 될 지, 제가 잘 나아가고 있는 건지 확신이 들지 않았습니다.
하지만 `CloudOne`팀에서 다양한 경험을 하면서 '제가 공부해온 길이 틀리지만은 않았구나'라는 느낌을 받을
수 있었고, 보완해야할 부분들은 보완하면서 불확실한 마음이 아닌 확신과 열정을 가진 자세로
좀 더 몰두할 수 있을 것 같습니다!
 
배우고 느낀 내용이 너무 많아 글이 다소 길어졌습니다. 기술을 거부감 없이 접하되 
기술이 다가 아닌, 팀원들을 위해 솔선수범하는 개발자, 
팀원들의 생각을 읽어줄 수 있고, 잘 이해해줄 수 있는 개발자가 되기 위해 노력해야겠단 생각이 듭니다.
쉽진 않겠지만, 나아가고 싶은 방향은 정해진 것 같아 다행입니다!