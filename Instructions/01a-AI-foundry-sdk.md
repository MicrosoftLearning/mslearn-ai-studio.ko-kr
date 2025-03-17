---
lab:
  title: Azure AI 파운드리 SDK로 채팅 앱 만들기
---

# Azure AI 파운드리 SDK로 채팅 앱 만들기

이 연습에서는 Azure AI 파운드리 SDK를 사용하여 간단한 채팅 앱을 만듭니다.

이 연습에는 약 **20**분이 소요됩니다.

## Azure AI 파운드리 프로젝트 만들기

먼저 Azure AI 파운드리 프로젝트를 만들어 보겠습니다.

1. 웹 브라우저에서 [Azure AI 파운드리 포털](https://ai.azure.com)(`https://ai.azure.com`)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 처음 로그인할 때 열리는 팁이나 빠른 시작 창을 닫고, 필요한 경우 왼쪽 위에 있는 **Azure AI 파운드리** 로고를 사용하여 다음 이미지와 유사한 홈페이지로 이동합니다.

    ![Azure AI Foundry 포털의 스크린샷.](./media/ai-foundry-home.png)

1. 홈페이지에서 **+ 프로젝트 만들기**를 선택합니다.
1. **프로젝트 만들기** 마법사에서 적합한 프로젝트 이름(예: `my-ai-project`)을 입력한 다음 프로젝트를 지원하기 위해 자동으로 만들어지는 Azure 리소스를 검토합니다.
1. **사용자 지정**을 선택하고 허브에 대해 다음 설정을 지정합니다.
    - **허브 이름**: *고유 이름 - 예: `my-ai-hub`*
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: *고유한 이름(예: `my-ai-resources`)으로 새 리소스 그룹을 만들거나 기존 리소스 그룹을 선택합니다*.
    - **위치**: 다음 목록에서 임의의 지역을 선택합니다.\*
        - 미국 동부
        - 미국 동부 2
        - 미국 중북부
        - 미국 중남부
        - 스웨덴 중부
        - 미국 서부
        - 미국 서부 3
    - **Azure AI 서비스 또는 Azure OpenAI 연결**: *적절한 이름(예: `my-ai-services`)으로 새 AI 서비스 리소스를 만들거나 기존 리소스를 사용합니다.*
    - **Azure AI 검색 연결**: 연결 건너뛰기

    > \* 모델 할당량은 테넌트 수준에서 지역별 할당량에 의해 제한됩니다. 임의 지역을 선택하면 여러 사용자가 동일한 테넌트에서 작업할 때 할당량 가용성을 분산할 수 있습니다. 연습 후반부에 할당량 한도에 도달하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다.

1. **다음**을 선택하여 구성을 검토합니다. **만들기**를 선택하고 프로세스가 완료될 때까지 기다립니다.
1. 프로젝트를 만들 때 표시되는 팁을 모두 닫고 Azure AI 파운드리 포털에서 프로젝트 페이지를 검토합니다. 이 페이지는 다음 이미지와 유사합니다.

    ![Azure AI 파운드리 포털의 Azure AI 프로젝트 세부 정보 스크린샷.](./media/ai-foundry-project.png)

## 생성형 AI 모델 배포

이제 채팅 애플리케이션을 지원하기 위해 생성형 AI 언어 모델을 배포할 준비가 되었습니다. 이 예에서는 Microsoft Phi-4 모델을 사용하지만 원칙은 모든 모델에 동일합니다.

1. 프로젝트 왼쪽 창의 **내 자산** 섹션에서 **모델 + 엔드포인트** 페이지를 선택합니다.
1. **모델 + 엔드포인트** 페이지 내 **모델 배포** 탭의 **+ 모델 배포** 메뉴에서 **베이스 모델 배포**를 선택합니다.
1. 목록에서 **Phi-4** 모델을 검색한 다음 선택하고 확인합니다.
1. **Azure AI 콘텐츠 보안이 적용된 서버리스 API** 배포 옵션을 선택합니다.
1. 배포 세부 정보에서 **사용자 지정**을 선택하여 다음 설정으로 모델을 배포합니다.
    - **배포 이름**: *모델 배포의 고유 이름 - 예: `phi-4-model`(나중에 필요하니 할당하는 이름을 기억하세요*)
    - **콘텐츠 필터**: 사용

## 모델과 채팅할 클라이언트 애플리케이션 만들기

이제 모델을 배포했으므로 Azure AI 파운드리 SDK를 사용하여 이 모델과 채팅하는 애플리케이션을 개발할 수 있습니다.

### 애플리케이션 구성 준비

1. Azure AI 파운드리 포털에서 프로젝트의 **개요** 페이지를 확인합니다.
1. **프로젝트 세부 정보** 영역에서 **프로젝트 연결 문자열**을 확인합니다. 이 연결 문자열을 사용하여 클라이언트 애플리케이션에서 프로젝트에 연결합니다.
1. 새 브라우저 탭을 엽니다(기존 탭에 Azure AI 파운드리 포털 열어두기). 그런 다음 새 탭에서 [Azure Portal](https://portal.azure.com)(`https://portal.azure.com`)로 이동하고, 메시지가 표시되면 Azure 자격 증명으로 로그인합니다.
1. 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 ***PowerShell*** 환경을 선택합니다. Cloud Shell은 Azure Portal 하단의 창에서 명령줄 인터페이스를 제공합니다.

    > **참고**: 이전에 *Bash* 환경을 사용하는 Cloud Shell을 만든 경우 ***PowerShell***로 전환합니다.

1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다(코드 편집기를 사용하는 데 필요).

1. PowerShell 창에서 다음 명령을 입력하여 이 연습이 포함된 GitHub 리포지토리를 복제합니다.

    ```
    rm -r mslearn-ai-foundry -f
    git clone https://github.com/microsoftlearning/mslearn-ai-studio mslearn-ai-foundry
    ```

1. 리포지토리가 복제되면 **mslearn-ai-foundry/labfiles/01a-azure-foundry-sdk/python** 폴더로 이동합니다.

    ```
    cd mslearn-ai-foundry/labfiles/01a-azure-foundry-sdk/python
    ```

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 사용할 Python 라이브러리를 설치합니다.
    - **python-dotenv**: 애플리케이션 구성 파일에서 설정을 로드하는 데 사용됩니다.
    - **azure-identity**: Entra ID 자격 증명으로 인증하는 데 사용됩니다.
    - **azure-ai-projects**: Azure AI 파운드리 프로젝트와 함께 작업하는 데 사용됩니다.
    - **azure-ai-inference**: 생성형 AI 모델과 채팅하는 데 사용됩니다.

    ```
    pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```

1. 제공된 **.env** Python 구성 파일을 편집하려면 다음 명령을 입력합니다.

    ```
    code .env
    ```

    파일이 코드 편집기에서 열립니다.

1. 코드 파일에서 **your_project_endpoint** 자리 표시자를 프로젝트의 연결 문자열(Azure AI 파운드리 포털의 프로젝트 **개요** 페이지에서 복사)로 바꾸고 **your_model_deployment** 자리 표시자를 Phi-4 모델 배포에 할당한 이름으로 바꿉니다.
1. 자리 표시자를 바꾼 후 **CTRL+S** 명령을 사용하여 변경 내용을 저장한 다음 **CTRL+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어둔 채 코드 편집기를 닫습니다.

### 프로젝트에 연결하고 모델과 채팅하는 코드를 작성합니다.

> **팁**: Python 코드 파일에 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다.

1. 다음 명령을 입력하여 제공된 **chat-app.py** Python 코드 파일을 편집합니다.

    ```
    code chat-app.py
    ```

1. 코드 파일에서 파일 상단에 추가된 기존 **import**문에 주목합니다. 그런 다음, **# AI 프로젝트 참조 추가** 주석 아래에 다음 코드를 추가하여 Azure AI 프로젝트 라이브러리를 참조합니다.

    ```python
    from azure.ai.projects import AIProjectClient
    ```

1. **main** 함수의 **# 구성 설정 가져오기** 주석 아래에서 코드가 **.env** 파일에서 정의한 프로젝트 연결 문자열 및 모델 배포 이름 값을 로드한다는 점에 유의합니다.
1. **# 프로젝트 클라이언트 초기화** 주석 아래에 다음 코드를 추가하여 현재 로그인한 Azure 자격 증명을 사용하여 Azure AI 파운드리 프로젝트에 연결합니다.

    ```python
    project = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential()
        )
    ```
    
1. **# 채팅 클라이언트 가져오기** 주석 아래에 다음 코드를 추가하여 모델과 채팅할 클라이언트 개체를 만듭니다.

    ```python
    chat = project.inference.get_chat_completions_client()
    ```

1. 이 코드에는 사용자가 "종료"를 입력할 때까지 프롬프트를 입력할 수 있도록 하는 루프가 포함되어 있습니다. 그런 다음 루프 섹션의 **# 채팅 완료 가져오기** 주석 아래에 다음 코드를 추가하여 프롬프트를 제출하고 모델에서 완료를 가져옵니다.

    ```python
    response = chat.complete(
        model=model_deployment,
        messages=[
            {"role": "system", "content": "You are a helpful AI assistant that answers questions."},
            {"role": "user", "content": input_text},
            ],
        )
    print(response.choices[0].message.content)
    ```

1. **CTRL+S** 명령을 사용하여 변경 사항을 코드 파일에 저장한 다음 **CTRL+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어둔 채 코드 편집기를 닫습니다.

### 채팅 애플리케이션 실행

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 Python 코드를 실행합니다.

    ```
    python chat-app.py
    ```

1. 메시지가 표시되면 질문(예: `What is the fastest animal on Earth?`)을 입력하고 생성형 AI 모델의 응답을 검토합니다.
1. 몇 가지 질문을 더 시도해 보세요. 완료했으면 `quit`을(를) 입력하여 프로그램을 종료합니다.

## 요약

이 연습에서는 Azure AI 파운드리 SDK를 사용하여 Azure AI 파운드리 프로젝트에 배포한 생성형 AI 모델용 클라이언트 애플리케이션을 만들었습니다.

## 정리

Azure AI Foundry 포털 탐색을 완료한 경우 불필요한 Azure 비용이 발생하지 않도록 이 연습에서 만든 리소스를 삭제해야 합니다.

1. Azure Portal이 포함된 브라우저 탭으로 돌아가서(또는 새 브라우저 탭의 `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)을 다시 열고) 이 연습에 사용된 리소스를 배포한 리소스 그룹의 내용을 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
