---
lab:
  title: 'Azure AI 스튜디오에서 언어 모델을 탐색하고, 배포하고, 채팅합니다.'
---

# Azure AI 스튜디오에서 언어 모델을 탐색하고, 배포하고, 채팅합니다.

Azure AI Studio의 모델 카탈로그는 다양한 모델을 탐색하고 사용할 수 있는 중앙 리포지토리 역할을 하므로 생성형 AI 시나리오를 쉽게 만들 수 있습니다.

이 연습에서는 Azure AI 스튜디오의 모델 카탈로그를 살펴봅니다.

이 연습은 약 **25**분 정도 소요됩니다.

## 시작하기 전에

이 연습을 완료하려면 Azure OpenAI service에 대한 액세스를 위해 Azure 구독이 승인되어야 합니다. [등록 양식을](https://learn.microsoft.com/legal/cognitive-services/openai/limited-access) 작성하여 Azure OpenAI 모델에 대한 액세스를 요청합니다.

## Azure AI 허브 만들기

프로젝트를 호스팅하려면 Azure 구독에 Azure AI 허브가 있어야 합니다. 프로젝트를 만드는 동안 이 리소스를 만들거나 미리 프로비전할 수 있습니다(이 연습에서 수행할 작업).

1. **관리** 섹션에서 **모든 허브**를 선택한 다음 **+ 새 허브**를 선택합니다. 다음 설정을 사용하여 새 허브를 만듭니다.
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
    - **Azure AI 서비스 또는 Azure OpenAI 연결**: *새 AI 서비스를 만들거나 기존 서비스를 사용하도록 선택합니다*.
    - **Azure AI 검색 연결**: 연결 건너뛰기

    > \* Azure OpenAI 리소스는 지역 할당량에 따라 테넌트 수준에서 제한됩니다. 나열된 지역에는 이 연습에 사용된 모델 형식에 대한 기본 할당량이 포함되어 있습니다. 지역을 임의로 선택하면 다른 사용자와 테넌트를 공유하는 시나리오에서 단일 지역이 할당량 한도에 도달할 위험이 줄어듭니다. 연습 후반부에 할당량 한도에 도달하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다.

    Azure AI 허브가 만들어지면 다음 이미지와 유사하게 표시됩니다.

    ![Azure AI 스튜디오의 Azure AI 허브 세부 정보 스크린샷.](./media/azure-ai-resource.png)

1. 새 브라우저 탭을 열고(Azure AI 스튜디오 탭을 열어둔 채로) [https://portal.azure.com](https://portal.azure.com?azure-portal=true)에서 Azure Portal로 이동하여 메시지가 표시되면 Azure 자격 증명으로 로그인합니다.
1. Azure AI 허브를 만든 리소스 그룹으로 이동하여 생성된 Azure 리소스를 확인합니다.

    ![Azure Portal의 Azure AI 허브 및 관련 리소스 스크린샷.](./media/azure-portal.png)

1. Azure AI 스튜디오 브라우저 탭으로 돌아갑니다.
1. 페이지의 왼쪽에 있는 창에서 Azure AI 허브의 각 페이지를 확인하고, 생성 및 관리할 수 있는 아티팩트를 확인합니다. **연결** 페이지에서 Azure OpenAI 및 AI 서비스에 대한 연결이 이미 만들어졌는지 확인합니다.

## 프로젝트 만들기

Azure AI 허브는 하나 이상의 *프로젝트*를 정의할 수 있는 공동 작업 영역을 제공합니다. Azure AI 허브에 프로젝트를 만들어 보겠습니다.

1. Azure AI 스튜디오에서 방금 만든 허브에 있는지 확인합니다(화면 상단의 경로를 확인하여 위치를 확인할 수 있음).
1. 왼쪽 메뉴를 사용하여 **모든 프로젝트**로 이동합니다.
1. **+ 새 프로젝트**를 선택합니다.
1. **새 프로젝트 만들기** 마법사에서 다음 설정을 사용하여 프로젝트를 만듭니다.
    - **현재 허브**: *AI 허브*
    - **프로젝트 이름**: *프로젝트의 고유한 이름*
1. 프로젝트가 만들어질 때까지 기다립니다. 준비가 되면 다음 이미지와 유사하게 표시됩니다.

    ![Azure AI 스튜디오의 프로젝트 세부 정보 페이지 스크린샷.](./media/azure-ai-project.png)

1. 왼쪽 창에서 페이지를 보고 각 섹션을 확장하여 수행할 수 있는 작업과 프로젝트에서 관리할 수 있는 리소스를 확인합니다.

## 모델 벤치마크를 사용하여 모델 선택

모델을 배포하기 전에 모델 벤치마크를 탐색하여 요구 사항에 가장 적합한 모델을 결정할 수 있습니다.

여행 도우미 역할을 하는 사용자 지정 Copilot 만들기를 원한다고 가정해봅니다. 특히, Copilot이 비자 요구 사항, 일기 예보, 지역 관광 명소, 문화 규범과 같은 여행 관련 문의에 대한 지원을 제공하기를 원합니다.

Copilot은 실제로 정확한 정보를 제공해야 하므로 근거가 중요합니다. 그 다음, Copilot의 대답은 읽고 이해하기 쉬워야 합니다. 따라서 유창성과 일관성이 높은 모델을 선택하려고 합니다.

1. Azure AI 스튜디오의 왼쪽 메뉴를 사용하여 **시작** 섹션에서 **모델 벤치마크**로 이동합니다.
    **품질 벤치마크** 탭에서 이미 시각화된 일부 차트를 찾아 다른 모델을 비교할 수 있습니다.
1. 표시된 모델을 필터링합니다.
    - **Tasks**: 질문 답변
    - **컬렉션**: Azure OpenAI
    - **메트릭**: 일관성, 유창성, 근거
1. 결과 차트와 비교 테이블을 탐색합니다. 탐색할 때 다음 질문에 대답할 수 있습니다.
    - Do you notice a difference in performance between GPT-3.5 and GPT-4 models?
    - Is there a difference between versions of the same model?
    - How do the 32k variants differ from the base models?

Azure OpenAI 컬렉션에서 GPT-3.5와 GPT-4 모델 중에서 선택할 수 있습니다. 두 모델을 배포하고 사용 사례와 비교하는 방법을 살펴보겠습니다.

## Azure OpenAI 모델 배포

이제 모델 벤치마크를 통해 옵션을 살펴보셨으므로 언어 모델을 배포할 준비가 되었습니다. 모델 카탈로그를 찾아보고 카탈로그에서 배포하거나, **배포** 페이지를 통해 모델을 배포할 수 있습니다. 두 옵션을 모두 살펴보겠습니다.

### 모델 카탈로그에서 모델 배포

먼저 모델 카탈로그에서 모델을 배포해 보겠습니다. 사용 가능한 모든 모델을 필터링하려는 경우 이 옵션을 사용할 수 있습니다.

1. 왼쪽 메뉴를 사용하여 **시작** 섹션 아래의 **모델 카탈로그** 페이지로 이동합니다.
1. 다음 설정을 사용하여 Azure AI에서 큐레이팅한 `gpt-35-turbo` 모델을 검색하고 배포합니다.
    - **배포 이름**: *모델 배포에 대한 고유한 이름으로, GPT-3.5 모델임을 나타냅니다.*
    - **모델 버전**: *기본 버전 선택*
    - **배포 유형**: 표준
    - **연결된 Azure OpenAI 리소스**: *허브를 만들 때 만든 기본 연결을 선택합니다*.
    - **분당 토큰 속도 제한(천 )**: 5K
    - **콘텐츠 필터**: 기본값

### 배포를 통해 모델 배포

배포하려는 모델을 정확히 알고 있는 경우 배포를 통해 수행하는 것이 좋습니다.

1. 왼쪽 메뉴를 사용하여 **구성 요소** 섹션 아래의 **배포** 페이지로 이동합니다.
1. **모델 배포** 탭에서 다음 설정을 사용하여 새 배포를 만듭니다.
    - **모델**: gpt-4
    - **배포 이름**: *모델 배포에 대한 고유한 이름으로, GPT-4 모델임을 나타냅니다.*
    - **모델 버전**: *기본 버전 선택*
    - **배포 유형**: 표준
    - **연결된 Azure OpenAI 리소스**: *허브를 만들 때 만든 기본 연결을 선택합니다*.
    - **분당 토큰 속도 제한(천 )**: 5K
    - **콘텐츠 필터**: 기본값

    > **참고**: 모델 벤치마크를 보여 주는 일부 모델을 발견했을 수 있지만 모델 카탈로그의 옵션으로는 볼 수 없습니다. 모델 가용성은 위치마다 다릅니다. 사용자 위치는 AI 허브 수준에서 지정되며, **위치 도우미**를 사용하여 배포하려는 모델을 지정하면 배포할 수 있는 위치 목록을 가져올 수 있습니다.

## 채팅 플레이그라운드에서 모델 테스트

이제 비교할 두 가지 모델이 있으므로 대화형 상호 작용에서 모델이 어떻게 동작하는지 살펴보겠습니다.

1. 왼쪽 메뉴를 사용하여 **프로젝트 플레이그라운드** 섹션 아래의 **채팅** 페이지로 이동합니다.
1. **채팅 플레이그라운드**에서 GPT-3.5 배포를 선택합니다.
1. 채팅 창에 `What can you do?`와 같은 쿼리를 입력하고 응답을 확인합니다.
    대답은 매우 일반적입니다. 여행 도우미 역할을 하는 사용자 지정 Copilot 만들기를 원합니다. 질문에서 원하는 도움말 종류를 지정할 수 있습니다.
1. 채팅 창에서 쿼리 `Imagine you're a travel assistant, what can you help me with?`를 입력합니다. 답변은 이미 더 구체적입니다. 최종 사용자가 Copilot과 상호 작용할 때마다 필요한 컨텍스트를 제공하지 않도록 할 수 있습니다. 전역 지침을 추가하기 위해 시스템 메시지를 편집할 수 있습니다.
1. 다음 프롬프트를 사용하여 시스템 메시지를 업데이트합니다.

   ```
   You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms.
   ```

1. **변경 내용 적용**을 선택하고 **채팅 지우기**를 선택합니다.
1. 채팅 창에서 쿼리 `What can you do?`를 입력하고 새 응답을 봅니다. 이전에 받은 답변과 어떻게 다른지 관찰합니다. 이제 대답은 여행에 특정합니다.
1. `I'm planning a trip to London, what can I do there?`를 질문하여 대화를 계속합니다. Copilot에게 여행 관련 정보를 많이 제공합니다. 출력을 계속 개선할 수 있습니다. 예를 들어 대답이 더 간결하게 표시되도록 할 수 있습니다.
1. 메시지의 끝에 `Answer with a maximum of two sentences.`를 추가하여 시스템 메시지를 업데이트합니다. 변경 내용을 적용하고, 채팅을 지우고, `I'm planning a trip to London, what can I do there?`를 질문하여 채팅을 다시 테스트합니다. 또한 단순히 질문에 대답하는 대신 Copilot과 대화를 계속할 수도 있습니다.
1. 메시지의 끝에 `End your answer with a follow-up question.`을 추가하여 시스템 메시지를 업데이트합니다. 변경 사항을 적용하고, 채팅을 지우고, `I'm planning a trip to London, what can I do there?`를 질문하여 채팅을 다시 테스트합니다.
1. **배포**를 GPT-4 모델로 변경하고 이 섹션의 모든 단계를 반복합니다. 모델이 출력에서 어떻게 달라질 수 있는지 확인합니다.
1. 마지막으로 쿼리 `Who is the prime minister of the UK?`에서 두 모델을 모두 테스트합니다. 이 질문에 대한 성능은 모델의 근거(응답이 실제로 정확한지 여부)와 관련이 있습니다. 성능은 모델 벤치마크의 결론과 관련이 있나요?

이제 두 모델을 모두 살펴보았으므로 사용 사례에 대해 지금 선택할 모델을 고려합니다. 처음에는 모델의 출력이 다를 수 있으며 다른 모델보다 한 모델을 선호할 수 있습니다. 그러나 시스템 메시지를 업데이트한 후에는 차이가 최소화된 것을 알 수 있습니다. 비용 최적화 관점에서 GPT-4 모델보다 GPT-3.5 모델을 선택할 수 있습니다. 성능은 매우 유사합니다.

## 정리

Azure AI 스튜디오 탐색을 완료한 경우 불필요한 Azure 비용이 발생하지 않도록 이 연습에서 만든 리소스를 삭제해야 합니다.

1. Azure Portal이 포함된 브라우저 탭으로 돌아가서(또는 새 브라우저 탭에서 [Azure Portal](https://portal.azure.com?azure-portal=true)을 다시 열고) 이 연습에 사용된 리소스를 배포한 리소스 그룹의 콘텐츠를 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.