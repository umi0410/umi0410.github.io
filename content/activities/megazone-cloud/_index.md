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

안녕하세요. 이번에 메가존 클라우드에서 데브옵스 인턴으로 근무를 하게되었던 박진수라고 합니다.
너무 좋은 팀원들과 많은 경험을 하며 단시간에 성장할 수 있었고, 일하는 동안 매 순간 순간이 너무
행복했었기에 이렇게 인턴 후기를 작성해봅니다. 기술적인 얘기는 이곳 저곳에 많으니
항상 제가 가장 중요시하는 **"느낀 점"** 과 **"배운 점"** 을 위주로 적어보겠습니다!!
글은 아래의 순서로 진행해보겠습니다.

* 지원동기
* 진행했던 업무 
* 느낀 점

## 지원동기
저는 AWS와 배포에 관해 흥미가 있어 종종 AWSKRUG라는 그룹의 너소모임에 가서 핸즈온이나 세미나에
참여하곤 했는데요. 그곳에서 저희 팀원들을 만나게 되었고, 그것이 인연이 되어
채용으로까지 이어질 수 있었습니다.

평소 AUSG([https://velog.io/@ausg](https://velog.io/@ausg)) 라는 대학생 AWS 사용자 모임의
일원으로서 참여하며 AWS를 이용한 클라우드 인프라, CI/CD 파이프라인 구축을 통한 자동화, 컨테이너 등에
관심이 많았는데, 마침 메가존 클라우드의 저희 CloudOne 팀에서는 팀의 모든 마이크로서비스 및 기타 서비스들을
EKS라는 AWS의 Managed Kubernetes Cluster Service 위에 자동화시켜 배포를 하고있었고,
개발 인프라를 이전하려던 참이었기에 저의 관심사와 향후 목적에 잘 부합했습니다.

## 진행했던 업무들

> 링크 형식이므로 링크를 눌러 읽어주시면 됩니다.

* [2020.04: `Stargate` 라는 개발 인프라를 구축](stargate-infra)
  * terraform을 통한 개발 인프라 구축
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
