# NER(명명된 엔터티 인식)에 프롬프트 흐름 사용

텍스트에서 중요한 정보를 추출하는 것을 NER(명명된 엔터티 인식)이라고 합니다. 엔터티는 지정된 텍스트에서 사용자가 관심을 가질 만한 키워드입니다.

![엔티티 추출](./media/get-started-prompt-flow-use-case.gif)

LLM(대규모 언어 모델)을 사용하여 NER을 수행할 수 있습니다. 텍스트를 입력으로 사용하고 엔터티를 출력하는 애플리케이션을 만들려면 프롬프트 흐름이 있는 LLM 노드를 사용하는 흐름을 만들 수 있습니다.

이 연습에서는 Azure AI Foundry 포털의 프롬프트 흐름을 사용하여 엔터티 형식과 텍스트를 입력으로 예상하는 LLM 애플리케이션을 만듭니다. AZURE OpenAI에서 LLM 노드를 통해 GPT 모델을 호출하여 지정된 텍스트에서 필요한 엔터티를 추출하고, 결과를 정리하고, 추출된 엔터티를 출력합니다.

![연습 개요](./media/get-started-lab.png)

먼저 Azure AI Foundry 포털에서 프로젝트를 만들어 필요한 Azure 리소스를 만들어야 합니다. 그런 다음 Azure OpenAI service를 사용하여 GPT 모델을 배포할 수 있습니다. 필요한 리소스가 있으면 흐름을 만들 수 있습니다. 마지막으로 흐름을 실행하여 테스트하고 샘플 출력을 확인합니다.

## Azure AI Foundry 포털에서 프로젝트 만들기

먼저 Azure AI Foundry 포털 프로젝트와 이를 지원하는 Azure AI 허브를 만듭니다.

1. 웹 브라우저에서 [https://ai.azure.com](https://ai.azure.com)을(를) 열고 Azure 자격 증명을 사용하여 로그인합니다.
1. 홈페이지에서 **+ 프로젝트 만들기**를 선택합니다.
1. **프로젝트 만들기** 마법사에서는 프로젝트를 통해 자동으로 만들어지는 모든 Azure 리소스를 보거나, **만들기**를 선택하기 전에 **사용자 지정**을 선택하여 다음 설정을 사용자 지정할 수 있습니다.

    - **프로젝트 이름**: *프로젝트의 고유한 이름*
    - **허브**: *다음 설정으로 새 허브를 만듭니다*.
    - **허브 이름**: *고유 이름*
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: *새 리소스 그룹*
    - **위치**: **선택 도움말**을 선택한 다음 위치 도우미 창에서 **gpt-4**를 선택하고 추천 지역을 사용합니다.\*
    - **Azure AI 서비스 또는 Azure OpenAI 연결**: (신규) *선택한 허브 이름으로 자동 채우기*
    - **Azure AI 검색 연결**: 연결 건너뛰기

    > \* Azure OpenAI 리소스는 지역 할당량에 따라 테넌트 수준에서 제한됩니다. 위치 도우미에 나열된 지역에는 이 연습에 사용된 모델 유형에 대한 기본 할당량이 포함되어 있습니다. 지역을 무작위로 선택하면 단일 지역이 할당량 한도에 도달할 위험이 줄어듭니다. 연습 후반부에 할당량 한도에 도달하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다. [지역별 모델 가용성](https://learn.microsoft.com/azure/ai-services/openai/concepts/models#availability)에 대해 자세히 알아보기

1. **사용자 지정**을 선택한 경우 **다음**을 선택하고 구성을 검토합니다.
1. **만들기**를 선택하고 프로세스가 완료될 때까지 기다립니다.

## GPT 모델 배포

프롬프트 흐름에서 LLM 모델을 사용하려면 먼저 모델을 배포해야 합니다. Azure AI Foundry 포털을 사용하면 흐름에서 사용할 수 있는 OpenAI 모델을 배포할 수 있습니다.

1. 왼쪽 탐색 창의 **내 자산** 아래에서 **모델 + 엔드포인트** 페이지를 선택합니다.
1. **+ 모델 배포** 및 **베이스 모델 배포**를 선택합니다.
1. 배포 세부 정보에서 **사용자 지정**을 선택하여 다음 설정으로 **gpt-4** 모델의 새 배포를 생성합니다.
   
    - **배포 이름**: *모델 배포에 대한 고유한 이름*
    - **배포 유형**: 표준
    - **모델 버전**: *기본 버전 선택*
    - **AI 리소스**: *이전에 만든 리소스 선택*
    - **분당 토큰 속도 제한(천 )**: 5K
    - **콘텐츠 필터**: DefaultV2
    - **동적 할당량 사용**: 사용할 수 없음
   
언어 모델을 배포했으므로 Azure AI Foundry 포털에서 배포한 모델을 호출하는 흐름을 만들 수 있습니다.

## Azure AI Foundry 포털에서 흐름 만들기 및 실행

이제 필요한 모든 리소스를 프로비저닝했으므로 흐름을 만들 수 있습니다.

### 새 흐름 만들기

템플릿을 사용하여 새 흐름을 만들려면 개발하려는 흐름 유형 중 하나를 선택할 수 있습니다.

1. 왼쪽의 탐색 창에서 **빌드 및 사용자 지정** 아래의 **프롬프트 흐름**을 선택합니다.
1. 새 흐름을 만들려면 **+ 만들기**를 선택합니다.
1. 새 **표준 흐름**을 만들고 `entity-recognition`를 폴더 이름으로 입력합니다.

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

한 개의 입력, 두 개의 노드, 한 개의 출력이 있는 표준 흐름이 만들어집니다. 두 개의 입력을 사용하고, 엔터티를 추출하고, LLM 노드에서 출력을 정리하고, 엔터티를 출력으로 반환하도록 흐름을 업데이트합니다.

### 자동 런타임 시작

흐름을 테스트하려면 컴퓨팅이 필요합니다. 필요한 컴퓨팅은 런타임을 통해 사용할 수 있습니다.

1. `entity-recognition` 이름을 지정한 새 흐름을 만든 후에는 스튜디오에서 흐름이 열립니다.
1. 위쪽 막대에서 **컴퓨팅 세션 시작**을 선택합니다.
1. 컴퓨팅 세션을 시작하는 데 1~3분이 걸립니다.

### 입력을 구성합니다.

흐름을 만들기 위해서는 텍스트와 텍스트에서 추출하고자 하는 엔터티의 형식이라는 두 개의 입력이 필요합니다.

1. **입력**에서 하나의 입력은 `string` 형식의 `topic` 이름으로 구성됩니다. 기존 입력을 변경하고 다음 설정으로 업데이트합니다.
    - **이름**: `entity_type`
    - **형식**: `string`
    - **값**: `job title`
1. **입력 추가**를 선택합니다.
1. 다음 설정을 갖추도록 두 번째 입력을 구성합니다.
    - **이름**: `text`
    - **형식**: `string`
    - **값**: `The software engineer is working on a new update for the application.`

### LLM 노드 구성

표준 흐름에는 LLM 도구를 사용하는 노드가 이미 포함되어 있습니다. 흐름 개요에서 노드를 찾을 수 있습니다. 기본 프롬프트는 농담을 요청합니다. 이전 섹션에서 지정한 두 입력을 기반으로 엔터티를 추출하도록 LLM 노드를 업데이트합니다.

1. `joke` 이름의 **LLM 노드**로 이동합니다.
1. `NER_LLM`로 이름 바꾸기
1. **연결**에서 AI 허브를 만들 때 생성한 연결을 선택합니다.
1. **deployment_name**을 위해 배포한 `gpt-4` 모델을 선택합니다.
1. 프롬프트 필드를 다음 코드로 바꿉니다.

   ```yml
   system:

   Your task is to find entities of a certain type from the given text content.
   If there're multiple entities, please return them all with comma separated, e.g. "entity1, entity2, entity3".
   You should only return the entity list, nothing else.
   If there's no such entity, please return "None".

   user:
   
   Entity type: {{entity_type}}
   Text content: {{text}}

   Entities:
   ```

1. **입력 유효성 검사 및 구문 분석**을 선택합니다.
1. LLM 노드 내의 **입력** 섹션에서 다음을 구성합니다 .
    - `entity_type`에 대해 `${inputs.entity_type}` 값을 선택합니다.
    - `text`에 대해 `${inputs.text}` 값을 선택합니다.

이제 LLM 노드는 엔터티 형식과 텍스트를 입력으로 사용하고, 지정한 프롬프트에 포함하고, 배포된 모델에 요청을 보냅니다.

### Python 노드 구성

모델 결과에서 주요 정보만 추출하려면 Python 도구를 사용하여 LLM 노드의 출력을 정리할 수 있습니다.

1. `echo`라는 이름의 Python 노드로 이동합니다.
1. `cleansing`로 이름 바꾸기
1.  코드를 다음으로 바꿉니다.

   ```python
   from typing import List
   from promptflow import tool
    
    
   @tool
   def cleansing(entities_str: str) -> List[str]:
       # Split, remove leading and trailing spaces/tabs/dots
       parts = entities_str.split(",")
       cleaned_parts = [part.strip(" \t.\"") for part in parts]
       entities = [part for part in cleaned_parts if len(part) > 0]
       return entities
    
   ```

1. **입력 유효성 검사 및 구문 분석**을 선택합니다.
1. Python 노드 내의 **입력** 섹션에서 `entities_str` 값을 `${NER_LLM.output}`로 설정합니다.

### 출력 구성

마지막으로 전체 흐름의 출력을 구성할 수 있습니다. 추출된 엔터티여야 하는 하나의 출력만 흐름에 적용하려고 합니다.

1. 흐름의 **출력**으로 이동합니다.
1. **이름**에 `entities`를 입력합니다.
1. **값**으로 `${cleansing.output}`을 선택합니다.

### 흐름 실행

이제 흐름을 개발했으므로 이를 실행하여 테스트할 수 있습니다. 입력에 기본값을 추가했으므로 스튜디오에서 흐름을 쉽게 테스트할 수 있습니다.

1. **실행**을 선택하여 흐름을 테스트합니다.
1. 실행이 완료될 때까지 기다립니다.
1. **출력 보기**를 선택합니다. 기본 입력에 대한 출력을 보여 주는 팝업이 표시됩니다. 필요에 따라 로그를 검사할 수도 있습니다.

## Azure 리소스 삭제

Azure AI Foundry 포털 탐색을 마치면 불필요한 Azure 비용을 피하기 위해 생성한 리소스를 삭제해야 합니다.

- `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)로 이동합니다.
- Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
- 이 연습을 위해 만든 리소스 그룹을 선택합니다.
- 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
- 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음 **삭제**를 선택합니다.
