---
lab:
  title: Azure AI 스튜디오의 모델 카탈로그 살펴보기
---

# Azure AI 스튜디오의 모델 카탈로그 살펴보기

Azure AI Studio의 모델 카탈로그는 다양한 모델을 탐색하고 사용할 수 있는 중앙 리포지토리 역할을 하므로 생성형 AI 시나리오를 쉽게 만들 수 있습니다.

이 연습에서는 Azure AI 스튜디오의 모델 카탈로그를 살펴봅니다.

> **참고**: Azure AI 스튜디오는 작성 당시 미리 보기로 제공되며 현재 개발 중입니다. 서비스의 일부 요소는 설명된 대로 정확하게 작동하지 않을 수 있으며 일부 기능은 예상대로 작동하지 않을 수 있습니다.

> 이 연습을 완료하려면 Azure OpenAI service에 대한 액세스를 위해 Azure 구독이 승인되어야 합니다.

이 연습은 약 **25**분 정도 소요됩니다.

## 새 Azure AI 허브 만들기

프로젝트를 호스트하려면 Azure 구독에 Azure AI 허브가 필요합니다. 프로젝트를 만드는 동안 이 리소스를 만들거나 미리 프로비전할 수 있습니다(이 연습에서 수행할 작업).

1. 웹 브라우저에서 [https://ai.azure.com](https://ai.azure.com)을(를) 열고 Azure 자격 증명을 사용하여 로그인합니다.

1. 관리 섹션에서 모든 허브를 선택한 다음 **+ 새 허브**를 선택합니다. 다음 설정을 사용하여 새 허브를 만듭니다.
    - **허브 이름**: *고유 이름*
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: *고유한 이름으로 새 리소스 그룹을 만들거나 기존 리소스 그룹 선택*
    - **위치**: *다음 지역 중 하나를 **임의로** 선택합니다.*\*
        - 오스트레일리아 동부
        - 캐나다 동부
        - 미국 동부
        - 미국 동부 2
        - 프랑스 중부
        - 일본 동부
        - 미국 중북부
        - 스웨덴 중부
        - 스위스 북부
        - 영국 남부
    - **Azure AI 서비스 또는 Azure OpenAI 연결**: 새 AI 서비스를 만들거나 기존 AI 서비스를 사용하려면 선택합니다.
    - **Azure AI 검색 연결**: 연결 건너뛰기

    > \* Azure OpenAI 리소스는 지역 할당량에 따라 테넌트 수준에서 제한됩니다. 나열된 지역에는 이 연습에 사용된 모델 형식에 대한 기본 할당량이 포함되어 있습니다. 지역을 임의로 선택하면 다른 사용자와 테넌트를 공유하는 시나리오에서 단일 지역이 할당량 한도에 도달할 위험이 줄어듭니다. 연습 후반부에 할당량 한도에 도달하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다.

1. **만들기**를 실행합니다. 첫 번째 허브 만들기를 완료하는 데 몇 분 정도 걸릴 수 있습니다. 허브를 만드는 동안 다음 AI 리소스도 생성됩니다. 
    - AI 서비스
    - 스토리지 계정
    - 주요 자격 증명 모음

1. Azure AI 허브를 만든 후에는 다음 이미지와 유사하게 표시됩니다.

    ![Azure AI 스튜디오의 Azure AI 허브 스크린샷.](./media/azure-ai-overview.png)

## 프로젝트 만들기

Azure AI 허브는 하나 이상의 *프로젝트*를 정의할 수 있는 공동 작업 영역을 제공합니다. Azure AI 허브에 프로젝트를 만들어 보겠습니다.

1. Azure AI 스튜디오의 **빌드** 페이지에서 **+ 새 프로젝트**를 선택합니다. 그런 다음 **새 프로젝트 만들기** 마법사에서 다음 설정을 사용하여 프로젝트를 만듭니다.

    - **프로젝트 이름**: *프로젝트의 고유한 이름*
    - **허브**: *AI Hub*

1. 프로젝트가 만들어질 때까지 기다립니다. 준비가 되면 다음 이미지와 유사하게 표시됩니다.

    ![Azure AI 스튜디오의 프로젝트 세부 정보 페이지 스크린샷.](./media/azure-ai-project.png)

1. 왼쪽 창에서 페이지를 보고 각 섹션을 확장하여 수행할 수 있는 작업과 프로젝트에서 관리할 수 있는 리소스를 확인합니다.

## 모델 배포

이제 **Azure AI 스튜디오**를 통해 사용할 모델을 배포할 준비가 되었습니다. 배포되면 모델을 사용하여 자연어 콘텐츠를 생성하게 됩니다.

1. Azure AI 스튜디오에서 다음 설정을 사용하여 새 배포를 만듭니다.

    - **모델**: gpt-35-turbo
    - **배포 유형**: 표준
    - **연결된 Azure OpenAI 리소스**: *Azure OpenAI 연결*
    - **모델 버전**: 기본값으로 자동 업데이트
    - **배포 이름**: ‘원하는 고유한 이름’**
    - **고급 옵션**
        - **콘텐츠 필터**: 기본값
        - **분당 토큰 속도 제한**: 5K

> **참고**: 각 Azure AI 스튜디오 모델은 기능과 성능의 다양한 균형을 위해 최적화되어 있습니다. 이 연습에서는 자연어 생성 및 채팅 시나리오에 뛰어난 성능을 발휘하는 **GPT 3.5 Turbo** 모델을 사용할 예정입니다.

## 채팅 플레이그라운드의 모델 테스트

대화형 상호 작용에서 모델이 어떻게 작동하는지 살펴보겠습니다.

1. 왼쪽 창에서 **플레이그라운드**로 이동합니다.

1. **채팅** 모드의 **채팅 세션** 섹션에 다음 프롬프트를 입력합니다.

    ```
   <Placeholder>
    ```

1. 모델은 아마도 몇 가지 텍스트로 응답할 것입니다.

## 시스템 프롬프트 메시지 수정

채팅 플레이그라운드에서 시스템 프롬프트를 수정하여 모델의 동작을 변경할 수 있습니다. 시스템 프롬프트는 응답 생성을 시작하기 전에 모델이 받는 초기 메시지입니다.

1. **시스템 메시지** 섹션에서 시스템 메시지를 다음 텍스트로 변경합니다.

    ```
    <Placeholder>
    ```

1. 시스템 메시지에 변경 내용을 적용합니다.

1. **채팅 세션** 섹션에서 다음 프롬프트를 다시 입력합니다.

    ```
   <Placeholder>
    ```

8. 출력을 관찰합니다. 이 출력은 ...


## 채팅 플레이그라운드에 데이터 추가

<Placeholder>

## 정리

Azure OpenAI 리소스 사용이 완료되면 [Azure Portal](https://portal.azure.com/?azure-portal=true)에서 배포 또는 전체 리소스를 삭제해야 합니다.
