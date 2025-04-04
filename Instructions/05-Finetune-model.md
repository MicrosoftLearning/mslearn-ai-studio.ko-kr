---
lab:
  title: 언어 모델 미세 조정
  description: 사용자 고유의 추가 학습 데이터를 사용하여 모델을 미세 조정하고 해당 동작을 사용자 지정하는 방법을 알아봅니다.
---

# 언어 모델 미세 조정

언어 모델이 특정 방식으로 작동하도록 하려는 경우 프롬프트 엔지니어링을 사용하여 원하는 동작을 정의할 수 있습니다. 원하는 동작의 일관성을 개선하려는 경우 모델을 미세 조정하고, 이를 프롬프트 엔지니어링 접근 방식과 비교하여 어떤 방법이 가장 적합한지 평가할 수 있습니다.

이 연습에서는 사용자 지정 채팅 애플리케이션 시나리오에 사용할 언어 모델을 Azure AI Foundry로 미세 조정합니다. 미세 조정된 모델과 베이스 모델을 비교하여 미세 조정된 모델이 사용자의 요구 사항에 더 적합한지 평가할 수 있습니다.

여행사에서 일하면서 사람들의 휴가 계획을 도와주는 채팅 애플리케이션을 개발 중이라고 가정해 보겠습니다. 목적지와 활동을 제안하는 간단하면서도 영감을 주는 채팅을 만드는 것이 목표입니다. 채팅은 데이터 원본에 연결되어 있지 않으므로 고객과의 신뢰를 확보하기 위해 호텔, 항공편 또는 레스토랑에 대한 구체적인 추천을 **제공하지 않아야** 합니다.

이 연습은 약 **60**분 정도 소요됩니다\*.

> \***참고**: 이 소요 시간은 평균 환경을 기반으로 하는 예측값입니다. 미세 조정은 클라우드 인프라 리소스에 따라 달라지며, 데이터 센터 용량 및 동시 수요에 따라 프로비전하는 데 걸리는 시간은 달라질 수 있습니다. 이 연습의 일부 활동은 완료하는 데 시간이 <u>오래</u> 걸려 인내가 필요할 수 있습니다. 시간이 오래 걸릴 경우 [Azure AI 파운드리 미세 조정 설명서](https://learn.microsoft.com/azure/ai-studio/concepts/fine-tuning-overview)를 검토하거나 휴식을 취하는 것도 도움이 됩니다.

## Azure AI Foundry 포털에서 AI 허브 및 프로젝트 만들기

먼저 Azure AI 허브 내에서 Azure AI Foundry 포털 프로젝트를 만듭니다.

1. 웹 브라우저에서 [Azure AI 파운드리 포털](https://ai.azure.com)(`https://ai.azure.com`)을 열고 Azure 자격 증명을 사용하여 로그인합니다.
1. 홈페이지에서 **+ 프로젝트 만들기**를 선택합니다.
1. **새 프로젝트 만들기** 마법사에서 다음 설정을 사용하여 프로젝트를 만듭니다.
    - **프로젝트 이름**: *프로젝트의 고유한 이름*
    - **사용자 지정**을 선택합니다.
        - **허브**: *기본 이름으로 자동 채우기*
        - **구독**: *로그인 계정으로 자동 채우기*
        - **리소스 그룹**: (신규) *프로젝트 이름으로 자동 채우기*
        -  **위치**: **선택 도움말**을 선택한 다음 위치 도우미 창에서 **gpt-4-finetune**을 선택하고 추천 지역을 사용합니다.\*
        - **Azure AI 서비스 또는 Azure OpenAI 연결**: (신규) *선택한 허브 이름으로 자동 채우기*
        - **Azure AI 검색 연결**: 연결 건너뛰기

    > \* Azure OpenAI 리소스는 지역 할당량에 따라 테넌트 수준에서 제한됩니다. 위치 도우미에 나열된 지역에는 이 연습에 사용된 모델 유형에 대한 기본 할당량이 포함되어 있습니다. 연습 후반부에 할당량 한도에 도달하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다. [미세 조정 모델 지역](https://learn.microsoft.com/azure/ai-services/openai/concepts/models?tabs=python-secure%2Cglobal-standard%2Cstandard-chat-completions#fine-tuning-models) 자세히 알아보기

1. 구성을 검토하고 프로젝트를 만듭니다.
1. 프로젝트가 만들어질 때까지 기다립니다.

## GPT-4 모델 미세 조정

모델을 미세 조정하는 데 시간이 걸리므로, 지금 미세 조정 작업을 시작했지만 비교 목적으로 미세 조정되지 않은 기본 모델을 살펴본 후 다시 시작할 수 있습니다.

1. `https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/refs/heads/main/data/travel-finetune-hotel.jsonl`에서 [학습 데이터 세트](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/refs/heads/main/data/travel-finetune-hotel.jsonl)를 다운로드하여 로컬에 JSONL 파일로 저장합니다.

    > **참고**: 디바이스는 기본적으로 파일을 .txt 파일로 저장할 수 있습니다. 모든 파일을 선택하고 .txt 접미사를 제거하여 파일을 JSONL 형식으로 저장합니다.

1. 왼쪽 메뉴를 사용하여 **빌드 및 사용자 지정** 섹션 아래의 **미세 조정** 페이지로 이동합니다.
1. 새 미세 조정 모델을 추가하려면 버튼을 선택하고`gpt-4` 모델을 선택한 다음 **다음**을 선택합니다.
1. 다음 구성을 사용하여 모델을 **미세 조정**합니다.
    - **모델 버전**: *기본 버전 선택*
    - **모델 접미사**: `ft-travel`
    - **연결된 AI 리소스**: *허브를 만들 때 만들어진 연결을 선택합니다. 기본적으로 선택되어 있을 것입니다.*
    - **학습 데이터**: 파일 업로드

    <details>  
    <summary><b>문제 해결 팁</b>: 사용 권한 오류</summary>
    <p>사용 권한 오류가 표시되는 경우 다음 문제 해결 방법을 시도합니다.</p>
    <ul>
        <li>Azure Portal에서 AI 서비스 리소스를 선택합니다.</li>
        <li>리소스 관리의 ID 탭에서 시스템에서 할당된 관리 ID인지 확인합니다.</li>
        <li>관련된 스토리지 계정으로 이동합니다. IAM 페이지에서 역할 할당 <em>스토리지 Blob 데이터 소유자</em>를 추가합니다.</li>
        <li><strong>액세스 할당</strong>에서 <strong>관리 ID</strong>, <strong>+ 구성원 선택</strong>, <strong>모든 시스템 할당 관리 ID</strong>를 차례로 선택한 후 Azure AI 서비스 리소스를 선택합니다.</li>
        <li>새 설정을 검토하고 할당하여 저장하고 이전 단계를 다시 시도합니다.</li>
    </ul>
    </details>

    - **파일 업로드**: 이전 단계에서 다운로드한 JSONL 파일을 선택합니다.
    - **유효성 검사 데이터**: 없음
    - **작업 매개 변수**: *기본 설정 유지*
1. 미세 조정이 시작되고 완료하는 데 다소 시간이 걸릴 수 있습니다.

> **참고**: 미세 조정 및 배포에는 상당한 시간이 걸릴 수 있으므로(30분 이상) 주기적으로 다시 확인해야 할 수 있습니다. 기다리는 동안 다음 단계를 계속 진행할 수 있습니다.

## 베이스 모델 채팅

미세 조정 작업이 완료될 때까지 기다리는 동안 GPT-4 베이스 모델로 채팅하여 성능을 평가해 보겠습니다.

1. 왼쪽 메뉴를 사용하여 **자산** 섹션 아래의 **모델 + 엔드포인트** 페이지로 이동합니다.
1. **+ 모델 배포** 버튼을 선택하고 **베이스 모델 배포** 옵션을 선택합니다.
1. 다음 설정을 사용하여 `gpt-4`모델을 배포합니다.
    - **배포 이름**: *모델의 고유한 이름입니다. 기본값을 사용할 수 있습니다.*
    - **배포 유형**: 표준
    - **분당 토큰 속도 제한(천 )**: 5K
    - **콘텐츠 필터**: 기본값

> **참고**: 배포하려는 모델에 사용할 수 있는 할당량이 현재 AI 리소스 위치에 없는 경우 다른 위치를 선택하여 새 AI 리소스를 만들고 프로젝트에 연결하라는 메시지가 표시됩니다.

1. 배포가 완료되면 **플레이그라운드에서 열기** 버튼을 선택합니다.
1. 배포한 `gpt-4` 베이스 모델이 설정 창에서 선택되어 있는지 확인합니다.
1. 채팅 창에 `What can you do?`와 같은 쿼리를 입력하고 응답을 확인합니다.
    대답은 매우 일반적입니다. 사람들이 여행하는 데 영감을 주는 채팅 애플리케이션을 만들고 싶습니다.
1. 다음 프롬프트를 사용하여 설정 창의 시스템 메시지를 업데이트합니다.

    ```md
    You are an AI assistant that helps people plan their holidays.
    ```

1. **변경 내용 적용**을 선택한 다음 **채팅 지우기**를 선택하고 `What can you do?` 질문을 다시 던지면 도우미가 여행에 필요한 항공권, 호텔, 렌터카 예약을 도와줄 수 있다고 대답할 수 있습니다. 이러한 동작을 피하고 싶을 수 있습니다.
1. 새 프롬프트를 사용하여 시스템 메시지를 다시 업데이트합니다.

    ```md
    You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms.
    You should not provide any hotel, flight, rental car or restaurant recommendations.
    Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday.
    ```

1. **변경 내용 적용**을 선택하고 **채팅 지우기**를 선택합니다.
1. 채팅 애플리케이션을 계속 테스트하여 검색된 데이터에 근거하지 않은 정보를 제공하지는 않는지 확인합니다. 예를 들어 다음 질문을 하고 모델의 답변을 검토하여 모델이 응답하는 데 사용하는 톤과 쓰기 스타일에 특히 주의를 기울입니다.
   
    `Where in Rome should I stay?`
    
    `I'm mostly there for the food. Where should I stay to be within walking distance of affordable restaurants?`

    `What are some local delicacies I should try?`

    `When is the best time of year to visit in terms of the weather?`

    `What's the best way to get around the city?`

## 학습 파일 검토

기본 모델은 충분히 잘 작동하는 것처럼 보이지만 사용자는 생성형 AI 앱에서 특정 대화 스타일을 찾고 있을 수 있습니다. 미세 조정에 사용되는 학습 데이터는 원하는 응답 종류의 명시적 예제를 만들 수 있는 기회를 제공합니다.

1. 이전에 다운로드한 JSONL 파일을 엽니다(모든 텍스트 편집기에서 열 수 있습니다).
1. 학습 데이터 파일에서 JSON 문서 목록을 검사합니다. 첫 번째 항목은 다음과 유사해야 합니다(가독성을 위해 형식이 지정됨).

    ```json
    {"messages": [
        {"role": "system", "content": "You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms. You should not provide any hotel, flight, rental car or restaurant recommendations. Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday."},
        {"role": "user", "content": "What's a must-see in Paris?"},
        {"role": "assistant", "content": "Oh la la! You simply must twirl around the Eiffel Tower and snap a chic selfie! After that, consider visiting the Louvre Museum to see the Mona Lisa and other masterpieces. What type of attractions are you most interested in?"}
        ]}
    ```

    목록의 각 예제 상호 작용에는 기본 모델로 테스트한 것과 동일한 시스템 메시지, 여행 쿼리와 관련된 사용자 프롬프트 및 응답이 포함됩니다. 학습 데이터의 응답 스타일은 미세 조정된 모델이 응답 방법을 학습하는 데 도움이 됩니다.

## 미세 조정된 모델 배포

미세 조정이 성공적으로 완료되면 미세 조정된 모델을 배포할 수 있습니다.

1. **빌드 및 사용자 지정**의 **미세 조정** 페이지로 이동하여 미세 조정 작업과 그 상태를 확인합니다. 계속 실행 중인 경우 배포된 기본 모델과 채팅을 계속하거나 휴식을 취하도록 선택할 수 있습니다. 완료되면 계속할 수 있습니다.
1. 미세 조정된 모델을 선택합니다. **메트릭** 탭을 선택하고 미세 조정 메트릭을 탐색합니다.
1. 다음 구성을 사용하여 미세 조정된 모델을 배포합니다.
    - **배포 이름**: *모델의 고유한 이름입니다. 기본값을 사용할 수 있습니다.*
    - **배포 유형**: 표준
    - **분당 토큰 속도 제한(천 )**: 5K
    - **콘텐츠 필터**: 기본값
1. 배포가 완료될 때까지 기다렸다가 테스트할 수 있으며, 시간이 걸릴 수 있습니다. **프로비전 상태**가 성공할 때까지 확인합니다(업데이트된 상태를 보려면 브라우저를 새로 고쳐야 할 수 있음).

## 미세 조정된 모델 테스트

이제 미세 조정된 모델을 배포했으므로 배포된 베이스 모델을 테스트했던 것처럼 테스트할 수 있습니다.

1. 배포가 준비되면 미세 조정된 모델로 이동하고 **플레이그라운드에서 열기**를 선택합니다.
1. 시스템 메시지에 다음 지침이 포함되어 있는지 확인합니다.

    ```md
    You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms.
    You should not provide any hotel, flight, rental car or restaurant recommendations.
    Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday.
    ```

1. 미세 조정된 모델을 테스트하여 현재 동작이 더 일관성 있게 유지되는지 평가합니다. 예를 들어 다음 질문을 다시 하고 모델의 답변을 확인합니다.
   
    `Where in Rome should I stay?`
    
    `I'm mostly there for the food. Where should I stay to be within walking distance of affordable restaurants?`

    `What are some local delicacies I should try?`

    `When is the best time of year to visit in terms of the weather?`

    `What's the best way to get around the city?`

1. 응답을 검토한 후 기본 모델의 응답과 어떻게 비교하나요?

## 정리

Azure AI Foundry 탐색을 마치면 불필요한 Azure 비용이 발생하는 것을 방지하기 위해 생성한 리소스를 삭제해야 합니다.

- `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)로 이동합니다.
- Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
- 이 연습을 위해 만든 리소스 그룹을 선택합니다.
- 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
- 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음 **삭제**를 선택합니다.
