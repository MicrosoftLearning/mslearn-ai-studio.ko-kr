---
title: Azure OpenAI 연습
permalink: index.html
layout: home
---

# Azure AI 스튜디오를 사용한 생성형 AI 애플리케이션 개발

다음 연습은 개발자가 채팅 기반 "코파일럿"과 같은 생성형 AI 애플리케이션을 빌드하는 데 사용하는 일반적인 패턴과 기술을 살펴보고 Azure AI 서비스(특히 Azure OpenAI Service 및 Azure AI 스튜디오)를 사용하여 이 패턴을 구현하는 방법을 알아보는 실습 학습 환경을 제공하기 위해 설계되었습니다.

이 연습은 단독으로 완료할 수도 있지만 원래 [Microsoft Learn](https://learn.microsoft.com/training/paths/create-custom-copilots-ai-studio/)의 모듈에 대한 보충 자료로 설계되었으며, 여기에서 이 연습의 기반이 되는 몇 가지 기본 개념에 대해 자세히 알아볼 수 있습니다.

> **참고**: 연습을 완료하려면 Azure AI 스튜디오에서 사용하는 Azure 리소스를 프로비전하고 Azure OpenAI GPT 모델을 배포 및 사용하기에 충분한 권한과 할당량이 있는 Azure 구독이 필요합니다. 아직 계정이 없는 경우 [Azure 계정](https://azure.microsoft.com/free)에 가입합니다. 첫 30일 동안 사용할 수 있는 신규 사용자용 무료 평가판 옵션이 있으며, 크레딧도 제공합니다.

## 연습

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}