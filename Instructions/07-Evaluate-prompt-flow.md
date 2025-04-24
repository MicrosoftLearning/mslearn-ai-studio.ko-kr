---
lab:
  title: 생성형 AI 모델 성능 평가
  description: 채팅 앱의 성능과 적절한 응답 기능을 최적화하기 위해 모델과 프롬프트를 평가하는 방법을 알아.
---

# 생성형 AI 모델 성능 평가

이 연습에서는 Azure AI 파운드리 포털에서 수동 및 자동화된 평가를 사용하여 모델의 성능을 평가합니다.

이 연습은 약 **30**분 정도 소요됩니다.

> **참고**: 이 연습에 사용된 일부 기술은 미리 보기이거나 현재 개발 중에 있습니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

## Azure AI 파운드리 프로젝트 만들기

먼저 Azure AI 파운드리 프로젝트를 만들어 보겠습니다.

1. 웹 브라우저에서 [Azure AI 파운드리 포털](https://ai.azure.com)(`https://ai.azure.com`)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 처음 로그인할 때 열리는 팁이나 빠른 시작 창을 닫고, 필요한 경우 왼쪽 위에 있는 **Azure AI 파운드리** 로고를 사용하여 다음 이미지와 유사한 홈페이지로 이동합니다(**도움말** 창이 열려 있는 경우 닫습니다).

    ![Azure AI Foundry 포털의 스크린샷.](./media/ai-foundry-home.png)

1. 홈페이지에서 **+ 프로젝트 만들기**를 선택합니다.
1. **프로젝트 만들기** 마법사에서 유효한 프로젝트 이름을 입력하고 기존 허브가 추천되면 새 허브를 만드는 옵션을 선택합니다. 그런 다음 허브 및 프로젝트를 지원하기 위해 자동으로 만들어지는 Azure 리소스를 검토합니다.
1. **사용자 지정**을 선택하고 허브에 대해 다음 설정을 지정합니다.
    - **허브 이름**: *허브에서 유효한 이름*
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: ‘리소스 그룹 만들기 또는 선택’
    - **위치**: 다음 지역 중 하나를 선택합니다\*
        - 미국 동부 2
        - 프랑스 중부
        - 영국 남부
        - 스웨덴 중부
    - **Azure AI 서비스 또는 Azure OpenAI 연결**: *새 AI 서비스 리소스 만들기*
    - **Azure AI 검색 연결**: 연결 건너뛰기

    > \* 작성 시 이러한 지역은 AI 안전 메트릭의 평가를 지원합니다. 모델 가용성은 지역 할당량에 의해 제한됩니다. 연습 후반부에 할당량 한도에 도달하는 경우 다른 지역에서 다른 프로젝트를 만들어야 할 수도 있습니다.

1. **다음**을 선택하여 구성을 검토합니다. **만들기**를 선택하고 프로세스가 완료될 때까지 기다립니다.
1. 프로젝트를 만들 때 표시되는 팁을 모두 닫고 Azure AI 파운드리 포털에서 프로젝트 페이지를 검토합니다. 이 페이지는 다음 이미지와 유사합니다.

    ![Azure AI 파운드리 포털의 Azure AI 프로젝트 세부 정보 스크린샷.](./media/ai-foundry-project.png)

## 모델 배포

이 연습에서는 gpt-4o-mini 모델의 성능을 평가합니다. 또한 gpt-4o 모델을 사용하여 AI 지원 평가 메트릭을 생성합니다.

1. 프로젝트 왼쪽 탐색 창의 **내 자산** 섹션에서 **모델 + 엔드포인트** 페이지를 선택합니다.
1. **모델 + 엔드포인트** 페이지의 **모델 배포** 탭의 **+ 모델 배포** 메뉴에서 **기본 모델 배포**를 선택합니다.
1. 목록에서 **gpt-4o** 모델을 검색하고 선택한 후 확인합니다.
1. 배포 세부 정보에서 **사용자 지정**을 선택하여 다음 설정으로 모델을 배포합니다.
    - **배포 이름**: *모델 배포에 대한 유효한 이름*
    - **배포 유형**: 글로벌 표준
    - **자동 버전 업데이트**: 사용
    - **모델 버전**: *사용 가능한 최신 버전 선택*
    - **연결된 AI 리소스**: *Azure OpenAI 리소스 연결 선택*
    - **분당 토큰 속도 제한(천 단위)**: 50K *(또는 50K 이하인 경우 구독에서 사용 가능한 최대치)*
    - **콘텐츠 필터**: DefaultV2

    > **참고**: TPM을 줄이면 사용 중인 구독에서 사용 가능한 할당량을 과도하게 사용하지 않을 수 있습니다. 이 연습에 사용되는 데이터는 50,000TPM이면 충분합니다. 사용 가능한 할당량이 이 수치 이하이면 연습을 완료할 수 있지만 속도 제한을 초과하는 경우 오류가 발생할 수 있습니다.

1. 배포가 완료될 때가지 기다립니다.
1. **모델 + 엔드포인트** 페이지로 돌아가서 이전 단계를 반복하여 동일한 설정으로** gpt-4o-mini** 모델을 배포합니다.

## 수동으로 모델 평가

테스트 데이터를 기반으로 모델 응답을 수동으로 검토할 수 있습니다. 수동 검토를 통해 다양한 입력을 테스트하여 모델이 예상대로 작동하는지 평가할 수 있습니다.

1. 새 브라우저 탭의 `https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/refs/heads/main/data/travel_evaluation_data.csv`에서 [travel_evaluation_data.csv](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/refs/heads/main/data/travel_evaluation_data.csv)를 다운로드하여 로컬 폴더에 저장합니다.
1. Azure AI 파운드리 포털 탭으로 돌아가서 탐색 창의 **평가 및 개선** 섹션에서 **평가**를 선택합니다.
1. **평가** 페이지에서 **수동 평가** 탭을 확인하고 **+ 새 수동 평가**를 선택합니다.
1. AI 여행 도우미에 대한 다음 지침으로 **시스템 메시지**를 변경합니다.

   ```
   Objective: Assist users with travel-related inquiries, offering tips, advice, and recommendations as a knowledgeable travel agent.

   Capabilities:
   - Provide up-to-date travel information, including destinations, accommodations, transportation, and local attractions.
   - Offer personalized travel suggestions based on user preferences, budget, and travel dates.
   - Share tips on packing, safety, and navigating travel disruptions.
   - Help with itinerary planning, including optimal routes and must-see landmarks.
   - Answer common travel questions and provide solutions to potential travel issues.
    
   Instructions:
   1. Engage with the user in a friendly and professional manner, as a travel agent would.
   2. Use available resources to provide accurate and relevant travel information.
   3. Tailor responses to the user's specific travel needs and interests.
   4. Ensure recommendations are practical and consider the user's safety and comfort.
   5. Encourage the user to ask follow-up questions for further assistance.
   ```

1. **구성** 섹션의 **모델** 목록에서 **gpt-4o-mini** 모델 배포를 선택합니다.
1. **수동 평가 결과** 섹션에서 **테스트 데이터 가져오기**를 선택하고 이전에 다운로드한 **travel_evaluation_data.csv** 파일을 업로드합니다. 그리고 데이터 세트 필드를 다음과 같이 매핑합니다.
    - **입력**: 질문
    - **예상 응답**: ExpectedResponse
1. 테스트 파일에서 질문과 예상 답변을 검토합니다. 이를 사용하여 모델이 생성하는 응답을 평가합니다.
1. 위쪽 막대에서 **실행**을 선택하여 입력으로 추가한 모든 질문에 대한 출력을 생성합니다. 몇 분 후 모델의 응답은 다음과 같이 새 **출력** 열에 표시되어야 합니다.

    ![Azure AI 파운드리 포털의 수동 평가 페이지 스크린샷.](./media/manual-evaluation.png)

1. 각 질문에 대한 출력을 검토하고 모델의 출력을 예상 응답과 비교한 다음 각 응답의 오른쪽 아래에서 좋아요 또는 싫어요 아이콘을 선택하여 결과를 "채점"합니다.
1. 응답을 채점한 후 목록 위의 요약 타일을 검토합니다. 그런 다음 도구 모음에서 **결과 저장**을 선택하고 적절한 이름을 할당합니다. 결과를 저장하면 나중에 다른 모델과의 추가 평가 또는 비교 시 이를 검색할 수 있습니다.

## 자동화된 평가 사용

모델 결과와 예상 응답을 수동으로 비교하는 것은 모델의 성능을 평가하는 유용한 방법이 될 수 있지만, 다양한 질문과 응답이 예상되는 시나리오에서는 시간이 많이 걸리고, 다양한 모델과 프롬프트 조합을 비교하는 데 사용할 수 있는 표준화된 메트릭이 거의 제공되지 않는다는 단점이 있습니다.

자동화된 평가는 메트릭을 계산하고 AI를 사용하여 일관성, 관련성 및 기타 요인에 대한 응답을 평가하여 이러한 단점을 해결할 수 있는 접근 방식입니다.

1. **수동 평가** 페이지 제목 옆에 있는 뒤로 화살표(**&larr;**)를 사용하여 **평가** 페이지로 돌아갑니다.
1. **자동화된 평가** 탭을 봅니다.
1. **새 평가 만들기**를 선택하고 메시지가 표시되면 **모델 및 프롬프트**를 평가하는 옵션을 선택합니다.
1. **새 평가 만들기** 페이지의 **기본 정보** 섹션에서 자동 생성된 기본 평가 이름(원하는 경우 변경할 수 있음)을 검토하고 **gpt-40-mini** 모델 배포를 선택합니다.
1. 이전에 사용한 AI 여행 도우미에 대한 동일한 지침으로 **시스템 메시지**를 변경합니다.

   ```
   Objective: Assist users with travel-related inquiries, offering tips, advice, and recommendations as a knowledgeable travel agent.

   Capabilities:
   - Provide up-to-date travel information, including destinations, accommodations, transportation, and local attractions.
   - Offer personalized travel suggestions based on user preferences, budget, and travel dates.
   - Share tips on packing, safety, and navigating travel disruptions.
   - Help with itinerary planning, including optimal routes and must-see landmarks.
   - Answer common travel questions and provide solutions to potential travel issues.
    
   Instructions:
   1. Engage with the user in a friendly and professional manner, as a travel agent would.
   2. Use available resources to provide accurate and relevant travel information.
   3. Tailor responses to the user's specific travel needs and interests.
   4. Ensure recommendations are practical and consider the user's safety and comfort.
   5. Encourage the user to ask follow-up questions for further assistance.
   ```

1. **테스트 데이터 구성** 섹션에서 GPT 모델을 사용하여 테스트 데이터를 생성하거나(그런 다음 원하는 대로 편집 및 보완할 수 있음), 기존 데이터 세트를 사용하거나 파일을 업로드할 수 있습니다. 이 연습에서는 **기존 데이터 세트 사용**을 선택한 다음, **travel_evaluation_data_csv_*xxxx...*** 데이터 세트(이전 .csv 파일 업로드 시 생성됨)를 선택합니다.
1. 데이터 세트에서 샘플 행을 검토한 다음 **데이터 열 선택** 섹션에서 다음 열 매핑을 선택합니다.
    - **쿼리**: 질문
    - **컨텍스트**: *공백으로 남겨둡니다. 컨텍스트 데이터 원본을 모델과 연결할 때 "근거성"을 평가하는 데 사용됩니다.*
    - **참값**: ExpectedAnswer
1. **평가하려는 항목 선택** 섹션에서 다음 평가 범주를 <u>모두</u> 선택합니다.
    - AI 품질(AI 지원)
    - 위험 및 안전(AI 지원)
    - AI 품질(NLP)
1. **판정용 모델 배포 선택** 목록에서 **gpt-4o** 모델을 선택합니다. 이 모델은 언어 관련 품질 및 표준 생성형 AI 비교 메트릭에 대한 ***gpt-4o-mini** 모델의 응답을 평가하는 데 사용됩니다.
1. **만들기**를 선택하고 평가 프로세스가 시작되고 완료될 때까지 기다립니다. 몇 분이 걸릴 수 있습니다.

    > **팁**: 프로젝트 사용 권한이 설정 중임을 나타내는 오류가 표시되는 경우 잠시 기다렸다가 **만들기**를 다시 선택합니다. 새로 만든 프로젝트에 대한 리소스 사용 권한이 전파되기까지 시간이 조금 걸릴 수 있습니다.

1. 평가가 완료되면 필요한 경우 아래로 스크롤하여 **메트릭 대시보드** 영역을 확인하고 **AI 품질(AI 지원)** 메트릭을 확인합니다.

    ![Azure AI 파운드리 포털의 AI 품질 메트릭 스크린샷.](./media/ai-quality-metrics.png)

    **<sup>(i)</sup>** 아이콘을 사용하여 메트릭 정의를 확인합니다.

1. **위험 및 안전** 탭에서 잠재적으로 유해한 콘텐츠와 관련된 메트릭을 확인합니다.
1. **AI 품질(NLP**) 탭에서 생성형 AI 모델에 대한 표준 메트릭을 확인합니다.
1. 필요한 경우 페이지 맨 위로 다시 스크롤하고 **데이터** 탭을 선택하여 평가의 원시 데이터를 확인합니다. 데이터에는 각 입력에 대한 메트릭과 응답을 평가할 때 gpt-4o 모델이 적용한 추론에 대한 설명이 포함되어 있습니다.

    ![Azure AI 파운드리 포털의 평가 데이터 스크린샷.](./media/evaluation-data.png)

## 정리

Azure AI Foundry 탐색을 마치면 불필요한 Azure 비용을 피하기 위해 생성한 리소스를 삭제해야 합니다.

- `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)로 이동합니다.
- Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
- 이 연습을 위해 만든 리소스 그룹을 선택합니다.
- 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
- 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음 **삭제**를 선택합니다.
