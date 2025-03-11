---
lab:
  title: 생성형 AI 성능 평가
  description: 모델 및 채팅 흐름을 평가하여 채팅 앱의 성능과 적절하게 응답하는 기능을 최적화하는 방법을 알아봅니다.
---

# 생성형 AI 성능 평가

이 연습에서는 Azure AI Foundry 포털을 통해 AI 애플리케이션의 성능을 평가하고 비교하기 위한 기본 제공 및 사용자 지정 평가를 탐색합니다.

이 연습은 약 **30**분 정도 소요됩니다.

## Azure AI Foundry에서 AI 허브 및 프로젝트 만들기

먼저 Azure AI 허브 내에서 Azure AI Foundry 프로젝트를 만듭니다.

1. 웹 브라우저에서 [https://ai.azure.com](https://ai.azure.com)을(를) 열고 Azure 자격 증명을 사용하여 로그인합니다.
1. **홈** 페이지를 선택한 다음 **+ 새 프로젝트**를 선택합니다.
1. **프로젝트 만들기** 마법사에서 프로젝트의 고유 이름을 지정한 다음 **사용자 지정**을 선택하고 다음 설정을 지정합니다.
    - **허브 이름**: *고유 이름*
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: *새 리소스 그룹*
    - **위치**: **선택 도움말**을 선택한 다음 위치 도우미 창에서 **gpt-35-turbo**를 선택하고 추천 지역을 사용합니다.\*
    - **Azure AI 서비스 또는 Azure OpenAI 연결**: (신규) *선택한 허브 이름으로 자동 채우기*
    - **Azure AI 검색 연결**: 연결 건너뛰기

    > \* Azure OpenAI 리소스는 지역 할당량에 따라 테넌트 수준에서 제한됩니다. 위치 도우미에 나열된 지역에는 이 연습에 사용된 모델 유형에 대한 기본 할당량이 포함되어 있습니다. 지역을 무작위로 선택하면 단일 지역이 할당량 한도에 도달할 위험이 줄어듭니다. 연습 후반부에 할당량 한도에 도달하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다. [지역별 모델 가용성](https://learn.microsoft.com/azure/ai-services/openai/concepts/models#gpt-35-turbo-model-availability)에 대해 자세히 알아보기

1. **다음**을 선택하여 구성을 검토합니다.
1. **프로젝트 만들기**를 선택하고 프로세스가 완료될 때까지 기다립니다.

## GPT 모델 배포

프롬프트 흐름에서 언어 모델을 사용하려면 먼저 모델을 배포해야 합니다. Azure AI Foundry 포털을 사용하면 흐름에서 사용할 수 있는 OpenAI 모델을 배포할 수 있습니다.

1. 왼쪽 메뉴를 사용하여 **자산** 섹션 아래의 **모델 + 엔드포인트** 페이지로 이동합니다.
1. **+ 모델 배포** 버튼을 선택하고 **베이스 모델 배포** 옵션을 선택합니다.
1. **모델 배포** 마법사에서 **사용자 지정**을 선택하여 다음 설정으로 **gpt-35-turbo** 모델의 새 배포를 만듭니다.
    - **배포 이름**: *모델 배포에 대한 고유한 이름*
    - **배포 유형**: 표준
    - **모델 버전**: *기본 버전 선택*
    - **AI 리소스**: *이전에 만든 리소스 선택*
    - **분당 토큰 속도 제한(천 )**: 5K
    - **콘텐츠 필터**: DefaultV2
    - **동적 할당량 사용**: 사용할 수 없음

    > **참고**: 배포하려는 모델에 사용할 수 있는 할당량이 현재 AI 리소스 위치에 없는 경우 다른 위치를 선택하여 새 AI 리소스를 만들고 프로젝트에 연결하라는 메시지가 표시됩니다.

1. 모델이 배포될 때까지 기다립니다. 배포가 준비되면 **플레이그라운드에서 열기**를 선택합니다.
1. **모델 지침 및 컨텍스트 제공** 텍스트 상자에서 내용을 다음과 같이 변경합니다.

   ```
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
1. 채팅(기록) 창에서 `What can you do?` 쿼리를 입력하여 언어 모델이 예상대로 작동하는지 확인합니다.

이제 업데이트된 시스템 메시지가 포함된 배포된 모델이 있으므로 모델을 평가할 수 있습니다.

## Azure AI Foundry 포털에서 언어 모델 수동 평가

테스트 데이터를 기반으로 모델 응답을 수동으로 검토할 수 있습니다. 수동으로 검토하면 여러 입력을 한 번에 하나씩 테스트하여 모델이 예상대로 수행되는지 여부를 평가할 수 있습니다.

1. **채팅 플레이그라운드**의 위쪽 표시줄에서 **평가** 드롭다운을 선택하고 **수동 평가**를 선택합니다.
1. **시스템 메시지**를 위에서 사용한 것과 동일한 메시지로 변경합니다(여기에 다시 포함됨).

   ```
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

1. **수동 평가 결과** 섹션에서 출력을 검토할 5개의 입력을 추가합니다. 다음 5개의 질문을 5개의 개별 **입력**으로 입력합니다.

   `Can you provide a list of the top-rated budget hotels in Rome?`

   `I'm looking for a vegan-friendly restaurant in New York City. Can you help?`

   `Can you suggest a 7-day itinerary for a family vacation in Orlando, Florida?`

   `Can you help me plan a surprise honeymoon trip to the Maldives?`

   `Are there any guided tours available for the Great Wall of China?`

1. 위쪽 막대에서 **실행**을 선택하여 입력으로 추가한 모든 질문에 대한 출력을 생성합니다.
1. 이제 응답 오른쪽 하단에 있는 엄지 손가락 위로 또는 아래로 아이콘을 선택하여 각 질문에 대한 출력을 수동으로 검토할 수 있습니다. 각 응답을 평가할 때는 평가에 엄지손가락을 하나 이상 올리거나 내리는 응답을 포함해야 합니다.
1. 위쪽 막대에서 **결과 저장**을 선택합니다. 결과의 이름으로 `manual_evaluation_results`를 입력합니다.
1. 왼쪽 메뉴를 사용하여 **평가**로 이동합니다.
1. **수동 평가** 탭을 선택하면 방금 저장한 수동 평가를 찾을 수 있습니다. 이전에 만든 수동 평가를 탐색하고 중단한 부분부터 계속 진행하며 업데이트된 평가를 저장할 수 있습니다.

## 기본 제공 메트릭을 사용하여 채팅 앱 평가

프롬프트 흐름을 사용하여 채팅 애플리케이션을 만든 경우 일괄 처리를 실행하고 기본 제공 메트릭을 통해 흐름의 성능을 평가하는 방법으로 흐름을 평가할 수 있습니다.

![평가를 위한 입력 데이터 세트 생성 다이어그램.](./media/diagram-dataset-evaluation.png)

채팅 흐름을 평가하기 위해 사용자 쿼리 및 채팅 응답이 평가를 위한 입력으로 제공됩니다.

시간을 절약하기 위해 프롬프트 흐름에서 처리되는 여러 입력의 결과를 포함하는 일괄 처리 출력 데이터 세트를 만들었습니다. 각 결과는 다음 단계에서 평가할 데이터 세트에 저장됩니다.

1. **자동 평가** 탭을 선택하고 <details> 설정을 사용하여 **새 평가**를 만듭니다.  
      <summary><b>문제 해결 팁</b>: 사용 권한 오류</summary>
        <p>새 프롬프트 흐름을 만들 때 사용 권한 오류가 표시되는 경우 다음을 시도하여 문제를 해결합니다.</p>
        <ul>
          <li>Azure Portal에서 AI 서비스 리소스를 선택합니다.</li>
          <li>리소스 관리 아래의 ID 탭에서 시스템에서 할당된 관리 ID인지 확인합니다.</li>
          <li>관련된 스토리지 계정으로 이동합니다. IAM 페이지에서 역할 할당 <em>스토리지 Blob 데이터 독자</em>를 추가합니다.</li>
          <li><strong>액세스 권한 할당 대상</strong>에서 <strong>관리 ID</strong>, <strong>+구성원 선택</strong>, <strong>모든 시스템 할당 관리 ID</strong>를 선택합니다.</li>
          <li>새 설정을 검토하고 할당하여 저장하고 이전 단계를 다시 시도합니다.</li>
        </ul>
    </details>

    - **평가할 항목**: 데이터 세트
    - **평가 이름**: *고유한 이름 입력*
    - **다음**을 선택합니다.
    - **평가할 데이터 선택**: 데이터 세트 추가
        - `https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/main/data/travel-qa.jsonl`에서 [유효성 검사 데이터 세트](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/main/data/travel-qa.jsonl)를 다운로드하여 JSONL 파일로 저장하고 UI에 업로드합니다.

    > **참고**: 디바이스는 기본적으로 파일을 .txt 파일로 저장할 수 있습니다. 모든 파일을 선택하고 .txt 접미사를 제거하여 파일을 JSONL 형식으로 저장합니다.

    - **다음**을 선택합니다.
    - **메트릭 선택**: 일관성, 유창성
    - **연결**: *AI 서비스 연결*
    - **배포 이름/모델**: *배포된 GPT-3.5 모델*
    - **쿼리**: 데이터 원본으로 **쿼리**를 선택합니다.
    - **응답**: 데이터 원본으로 **응답**을 선택합니다.
      
1. **다음**을 선택한 다음 데이터를 검토하고 **제출**을 선택하여 새 평가를 제출합니다.
1. 평가가 완료될 때까지 기다렸다가 새로 고침해야 할 수도 있습니다.
1. 방금 만든 평가 실행을 선택합니다.
1. **메트릭 대시보드** 및 **메트릭 상세 결과**를 살펴보세요.

## Azure 리소스 삭제

Azure AI Foundry 탐색을 마치면 불필요한 Azure 비용을 피하기 위해 생성한 리소스를 삭제해야 합니다.

- `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)로 이동합니다.
- Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
- 이 연습을 위해 만든 리소스 그룹을 선택합니다.
- 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
- 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음 **삭제**를 선택합니다.
