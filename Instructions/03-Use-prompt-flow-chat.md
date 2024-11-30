---
lab:
  title: Azure AI Foundry 포털에서 프롬프트 흐름으로 사용자 지정 Copliot 빌드
---

# Azure AI Foundry 포털에서 프롬프트 흐름으로 사용자 지정 Copliot 빌드

이 연습에서는 Azure AI Foundry 포털의 프롬프트 흐름을 사용하여 사용자 프롬프트와 채팅 기록을 입력으로 사용하고 Azure OpenAI의 GPT 모델을 사용하여 출력을 생성하는 사용자 지정 Copilot을 만들어 보겠습니다.

이 연습은 약 **30**분 정도 소요됩니다.

## Azure AI Foundry 포털에서 AI 허브 및 프로젝트 만들기

먼저 Azure AI 허브 내에서 Azure AI Foundry 포털 프로젝트를 만듭니다.

1. 웹 브라우저에서 [https://ai.azure.com](https://ai.azure.com)을(를) 열고 Azure 자격 증명을 사용하여 로그인합니다.
1. 홈페이지에서 **+ 프로젝트 만들기**를 선택합니다.
1. **프로젝트 만들기** 마법사에서는 프로젝트를 통해 자동으로 만들어지는 모든 Azure 리소스를 보거나, **만들기**를 선택하기 전에 **사용자 지정**을 선택하여 다음 설정을 사용자 지정할 수 있습니다.

    - **허브 이름**: *고유 이름*
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: *새 리소스 그룹*
    - **위치**: **선택 도움말**을 선택한 다음 위치 도우미 창에서 **gpt-35-turbo**를 선택하고 추천 지역을 사용합니다.\*
    - **Azure AI 서비스 또는 Azure OpenAI 연결**: (신규) *선택한 허브 이름으로 자동 채우기*
    - **Azure AI 검색 연결**: 연결 건너뛰기

    > \* Azure OpenAI 리소스는 지역 할당량에 따라 테넌트 수준에서 제한됩니다. 위치 도우미에 나열된 지역에는 이 연습에 사용된 모델 유형에 대한 기본 할당량이 포함되어 있습니다. 지역을 무작위로 선택하면 단일 지역이 할당량 한도에 도달할 위험이 줄어듭니다. 연습 후반부에 할당량 한도에 도달하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다. [지역별 모델 가용성](https://learn.microsoft.com/azure/ai-services/openai/concepts/models#gpt-35-turbo-model-availability)에 대해 자세히 알아보기

1. **사용자 지정**을 선택한 경우 **다음**을 선택하고 구성을 검토합니다.
1. **만들기**를 선택하고 프로세스가 완료될 때까지 기다립니다.

## GPT 모델 배포

프롬프트 흐름에서 언어 모델을 사용하려면 먼저 모델을 배포해야 합니다. Azure AI Foundry 포털을 사용하면 흐름에서 사용할 수 있는 OpenAI 모델을 배포할 수 있습니다.

1. 왼쪽 탐색 창의 **내 자산** 아래에서 **모델 + 엔드포인트** 페이지를 선택합니다.
1. 다음 설정을 사용하여 **gpt-35-turbo**의 새 배포를 만듭니다.
    - **배포 이름**: *모델 배포에 대한 고유한 이름*
    - **배포 유형**: 표준
    - **모델 버전**: *기본 버전 선택*
    - **AI 리소스**: *이전에 만든 리소스 선택*
    - **분당 토큰 속도 제한(천 )**: 5K
    - **콘텐츠 필터**: DefaultV2
    - **동적 할당량 사용**: 사용할 수 없음
1. 모델이 배포될 때까지 기다립니다. 배포가 준비되면 **플레이그라운드에서 열기**를 선택합니다.
1. 채팅 창에서 쿼리 `What can you do?`를 입력합니다.

    도우미에 대한 구체적인 지침이 없으므로 답변은 일반적이라는 점에 유의하세요. 작업에 집중하기 위해 시스템 프롬프트를 변경할 수 있습니다.

1. **모델 지침 및 컨텍스트 제공** 메시지를 다음과 같이 변경합니다.

   ```md
   **Objective**: Assist users with travel-related inquiries, offering tips, advice, and recommendations as a knowledgeable travel agent.

   **Capabilities**:
   - Provide up-to-date travel information, including destinations, accommodations, transportation, and local attractions.
   - Offer personalized travel suggestions based on user preferences, budget, and travel dates.
   - Share tips on packing, safety, and navigating travel disruptions.
   - Help with itinerary planning, including optimal routes and must-see landmarks.
   - Answer common travel questions and provide solutions to potential travel issues.
    
   **Instructions**:
   1. Engage with the user in a friendly and professional manner, as a travel agent would.
   2. Use available resources to provide accurate and relevant travel information.
   3. Tailor responses to the user's specific travel needs and interests.
   4. Ensure recommendations are practical and consider the user's safety and comfort.
   5. Encourage the user to ask follow-up questions for further assistance.
   ```

1. **변경 내용 적용**을 선택합니다.
1. 채팅 창에서 이전 `What can you do?`과 동일한 쿼리를 입력합니다. 응답에서 변경 사항을 확인합니다.

이제 배포된 GPT 모델에 대한 시스템 메시지를 사용해 보았으므로 프롬프트 흐름으로 작업하여 애플리케이션을 추가로 사용자 지정할 수 있습니다.

## Azure AI Foundry 포털에서 채팅 흐름 만들기 및 실행

템플릿에서 새 흐름을 만들거나 플레이그라운드의 구성에 따라 흐름을 만들 수 있습니다. 이미 플레이그라운드에서 실험을 했으므로 이 옵션을 사용하여 새 흐름을 만듭니다.

<details>  
    <summary><b>문제 해결 팁</b>: 사용 권한 오류</summary>
    <p>새 프롬프트 흐름을 만들 때 사용 권한 오류가 표시되는 경우 다음을 시도하여 문제를 해결합니다.</p>
    <ul>
        <li>Azure Portal에서 AI 서비스 리소스를 선택합니다.</li>
        <li>리소스 관리의 ID 탭에서 시스템에서 할당된 관리 ID인지 확인합니다.</li>
        <li>관련된 스토리지 계정으로 이동합니다. IAM 페이지에서 역할 할당 <em>스토리지 Blob 데이터 독자</em>를 추가합니다.</li>
        <li><strong>액세스 권한 할당 대상</strong>에서 <strong>관리 ID</strong>, <strong>+구성원 선택</strong>, <strong>모든 시스템 할당 관리 ID</strong>를 선택합니다.</li>
        <li>새 설정을 검토하고 할당하여 저장하고 이전 단계를 다시 시도합니다.</li>
    </ul>
</details>

1. **채팅 플레이그라운드**의 상단 바에서 **프롬프트 흐름**을 선택합니다.
1. 폴더 이름으로 `Travel-Chat`를 입력합니다.

    간단한 채팅 흐름이 만들어집니다. 두 가지 입력(채팅 기록 및 사용자의 질문), 배포된 언어 모델과 연결할 LLM 노드 및 채팅의 응답을 반영하는 출력이 있습니다.

    흐름을 테스트하려면 컴퓨팅이 필요합니다.

1. 위쪽 막대에서 **컴퓨팅 세션 시작**을 선택합니다.
1. 컴퓨팅 세션을 시작하는 데 1~3분이 걸립니다.
1. **채팅**이라는 LLM 노드를 찾습니다. 프롬프트에는 채팅 플레이그라운드에서 지정한 시스템 프롬프트가 이미 포함되어 있습니다.

    여전히 LLM 노드를 배포된 모델에 연결해야 합니다.

1. LLM 노드 섹션의 **연결**에서 AI 허브를 만들 때 생성한 연결을 선택합니다.
1. **Api**에서 **채팅**를 선택합니다.
1. **deployment_name**에서 배포한 **gpt-35-turbo** 모델을 선택합니다.
1. For **response_format**, select **{"type":"text"}**.
1. 프롬프트 필드를 검토하고 다음과 같은지 확인합니다.

   ```yml
   system:
   **Objective**: Assist users with travel-related inquiries, offering tips, advice, and recommendations as a knowledgeable travel agent.

   **Capabilities**:
   - Provide up-to-date travel information, including destinations, accommodations, transportation, and local attractions.
   - Offer personalized travel suggestions based on user preferences, budget, and travel dates.
   - Share tips on packing, safety, and navigating travel disruptions.
   - Help with itinerary planning, including optimal routes and must-see landmarks.
   - Answer common travel questions and provide solutions to potential travel issues.

   **Instructions**:
   1. Engage with the user in a friendly and professional manner, as a travel agent would.
   2. Use available resources to provide accurate and relevant travel information.
   3. Tailor responses to the user's specific travel needs and interests.
   4. Ensure recommendations are practical and consider the user's safety and comfort.
   5. Encourage the user to ask follow-up questions for further assistance.

   {% for item in chat_history %}
   user:
   {{item.inputs.question}}
   assistant:
   {{item.outputs.answer}}
   {% endfor %}

   user:
   {{question}}
   ```

### 흐름 테스트 및 배포

이제 흐름을 개발했으므로 채팅 창을 사용하여 흐름을 테스트할 수 있습니다.

1. 컴퓨팅 세션이 실행 중인지 확인합니다.
1. **저장**을 선택합니다.
1. **채팅**을 선택하여 흐름을 테스트합니다.
1. 쿼리 `I have one day in London, what should I do?`를 입력하고 출력을 검토합니다.

    만든 흐름의 동작에 만족하면 흐름을 배포할 수 있습니다.

1. **배포**를 선택하여 다음 설정으로 플로우를 배포합니다.
    - **기본 설정**
        - **엔드포인트**: 새로 만들기
        - **엔드포인트 이름**: *고유 이름 입력*
        - **배포 이름**: *고유 이름 입력*
        - **가상 머신**: Standard_DS3_v2
        - **인스턴트 개수**: 3
        - **추론 데이터 수집**: 사용됨
    - **고급 설정**:
        - *기본 설정 사용*
1. Azure AI Foundry 포털의 프로젝트에 있는 왼쪽 탐색 창에서 **내 자산** 아래 **모델 + 엔드포인트** 페이지를 선택합니다.
1. 기본적으로 **모델 배포** 목록이 표시되며, 여기에는 배포한 언어 모델과 배포한 흐름이 포함됩니다. 배포가 나열되고 성공적으로 생성되기까지 다소 시간이 걸릴 수 있습니다.
1. 배포가 성공하면 선택합니다. 그런 다음 **테스트** 페이지에서 프롬프트 `What is there to do in San Francisco?`를 입력하고 응답을 검토합니다.
1. 프롬프트 `Where else could I go?`를 입력하고 응답을 검토합니다.
1. 엔드포인트에 대한 **사용** 페이지를 보고, 엔드포인트용 클라이언트 애플리케이션을 빌드하는 데 사용할 수 있는 연결 정보와 샘플 코드가 포함되어 있어 사용자 지정 Copilot로 프롬프트 흐름 솔루션을 애플리케이션에 통합할 수 있다는 점에 유의하세요.

## Azure 리소스 삭제

Azure AI Foundry 포털 탐색을 마치면 불필요한 Azure 비용을 피하기 위해 생성한 리소스를 삭제해야 합니다.

- `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)로 이동합니다.
- Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
- 이 연습을 위해 만든 리소스 그룹을 선택합니다.
- 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
- 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음 **삭제**를 선택합니다.
