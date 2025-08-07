---
lab:
  title: 생성형 AI 채팅 앱 만들기
  description: Azure AI 파운드리 SDK를 사용하여 프로젝트에 연결하고 언어 모델을 사용하여 채팅하는 앱을 빌드하는 방법을 알아봅니다.
---

# 생성형 AI 채팅 앱 만들기

이 연습에서는 Azure AI Foundry Python SDK를 사용하여 프로젝트에 연결하고 언어 모델로 채팅하는 간단한 채팅 앱을 만듭니다.

> **참고**: 이 연습은 사전 출시된 SDK 소프트웨어를 기반으로 하며, 변경될 수 있습니다. 필요한 경우 특정 버전의 패키지를 사용했으며, 이는 최신 버전이 반영되지 않을 수 있습니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

이 연습은 Azure AI Foundry Python SDK를 기반으로 하지만 다음을 포함하여 여러 언어별 SDK를 사용하여 AI 채팅 애플리케이션을 개발할 수 있습니다. 포함 사항:

- [Python용 Azure AI 프로젝트](https://pypi.org/project/azure-ai-projects)
- [Microsoft .NET용 Azure AI 프로젝트](https://www.nuget.org/packages/Azure.AI.Projects)
- [JavaScript용 Azure AI 프로젝트](https://www.npmjs.com/package/@azure/ai-projects)

이 연습은 약 **40**분 정도 소요됩니다.

## Azure AI 파운드리 프로젝트에서 모델 배포

Azure AI 파운드리 프로젝트에 모델을 배포하는 것부터 시작해 보겠습니다.

1. 웹 브라우저에서 [Azure AI 파운드리 포털](https://ai.azure.com)(`https://ai.azure.com`)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 처음 로그인할 때 열리는 팁이나 빠른 시작 창을 닫고, 필요한 경우 왼쪽 위에 있는 **Azure AI 파운드리** 로고를 사용하여 다음 이미지와 유사한 홈페이지로 이동합니다(**도움말** 창이 열려 있는 경우 닫습니다).

    ![Azure AI Foundry 포털의 스크린샷.](./media/ai-foundry-home.png)

1. 홈페이지의 **모델 및 기능 탐색** 섹션에서 프로젝트에 사용할 `gpt-4o` 모델을 검색합니다.
1. 검색 결과에서 **GPT-4O** 모델을 선택하여 세부 정보를 확인한 다음 해당 모델의 페이지 상단에서 **이 모델 사용**을 선택합니다.
1. 프로젝트를 만들라는 메시지가 표시되면 프로젝트의 유효한 이름을 입력하고 **고급 옵션**을 펼칩니다.
1. **사용자 지정**을 선택하고 프로젝트에 대해 다음 설정을 지정합니다.
    - **Azure AI 파운드리 리소스**: *Azure AI 파운드리 리소스의 유효한 이름*
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: ‘리소스 그룹 만들기 또는 선택’
    - **지역**: **AI 서비스 지원 위치 *선택***\*

    > \* 일부 Azure AI 리소스는 지역 모델 할당량에 의해 제한됩니다. 연습 후반부에 할당량 한도를 초과하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다.

1. **만들기**를 선택한 다음, 프로젝트가 만들어질 때까지 기다립니다. 메시지가 표시되면 **글로벌 표준** 배포 유형을 사용하여 gpt-4o 모델을 배포하고 배포 세부 정보를 사용자 지정하여 **분당 토큰 속도 제한**을 50K(또는 50K 미만인 경우 사용 가능한 최대값)로 설정합니다.

    > **참고**: TPM을 줄이면 사용 중인 구독에서 사용 가능한 할당량을 과도하게 사용하지 않을 수 있습니다. 이 연습에 사용되는 데이터는 50,000TPM이면 충분합니다. 사용 가능한 할당량이 이 수치 이하이면 연습을 완료할 수 있지만 속도 제한을 초과하는 경우 오류가 발생할 수 있습니다.

1. 프로젝트가 생성되면 채팅 플레이그라운드가 자동으로 열리므로 모델을 테스트할 수 있습니다.
1. **설정** 창에서 모델 배포의 이름을 기록합니다(**GPT-4o**이어야 함). **모델 및 엔드포인트** 페이지에서 배포를 확인하면 이를 확인할 수 있습니다(왼쪽 탐색 창에서 해당 페이지를 열면 됩니다).
1. 왼쪽 탐색 창에서 **개요**를 선택하면 다음과 같은 프로젝트의 메인 페이지가 표시됩니다.

    ![Azure AI 파운드리 프로젝트 개요 페이지의 스크린샷.](./media/ai-foundry-project.png)

## 모델과 채팅할 클라이언트 응용 프로그램 만들기

이제 모델을 배포했으므로 Azure AI Foundry 및 Azure OpenAI SDK를 사용하여 이 모델과 채팅하는 애플리케이션을 개발할 수 있습니다.

### 애플리케이션 구성 준비

1. Azure AI 파운드리 포털에서 프로젝트의 **개요** 페이지를 봅니다.
1. **엔드포인트 및 키** 영역에서 **Azure AI Foundry** 라이브러리가 선택되어 있는지 확인하고 **Azure AI Foundry 프로젝트 엔드포인트**를 확인합니다. 이 엔드포인트를 사용하여 클라이언트 애플리케이션에서 프로젝트와 모델에 연결할 수 있습니다.

    > **참고**: Azure OpenAI 엔드포인트를 사용할 수도 있습니다!

1. 새 브라우저 탭을 엽니다(Azure AI 파운드리 포털을 기존 탭에서 열어 두기). 그런 다음 새 탭에서 [Azure Portal](https://portal.azure.com)(`https://portal.azure.com`)을 열고 메시지가 나타나면 Azure 자격 증명을 사용하여 로그인합니다.

    Azure Portal 홈페이지를 보려면 환영 알림을 닫습니다.

1. 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 구독에 저장소가 없는 ***PowerShell*** 환경을 선택합니다.

    Cloud Shell은 Azure Portal 하단의 창에서 명령줄 인터페이스를 제공합니다. 보다 쉽게 작업할 수 있도록 이 창의 크기를 조정하거나 최대화할 수 있습니다.

    > **참고**: 이전에 *Bash* 환경을 사용하는 Cloud Shell을 만든 경우 ***PowerShell***로 전환합니다.

1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다(코드 편집기를 사용하는 데 필요).

    **<font color="red">계속하기 전에 Cloud Shell의 클래식 버전으로 전환했는지 확인합니다.</font>**

1. Cloud Shell 창에서 다음 명령을 입력하여 이 연습의 코드 파일이 포함된 GitHub 리포지토리를 복제합니다(명령을 입력하거나 클립보드에 복사한 다음 명령줄을 마우스 오른쪽 단추로 클릭하고 일반 텍스트로 붙여넣기).

    ```
   rm -r mslearn-ai-foundry -f
   git clone https://github.com/microsoftlearning/mslearn-ai-studio mslearn-ai-foundry
    ```

    > **팁**: CloudShell에 명령을 입력하면 출력이 화면 버퍼의 많은 부분을 차지할 수 있습니다. `cls` 명령을 입력해 화면을 지우면 각 작업에 더 집중할 수 있습니다.

1. 리포지토리가 복제된 후 채팅 애플리케이션 코드 파일이 포함된 폴더로 이동하여 해당 파일을 확인하세요.

    ```
   cd mslearn-ai-foundry/labfiles/chat-app/python
   ls -a -l
    ```

    폴더에는 코드 파일과 애플리케이션 설정을 위한 구성 파일, 프로젝트 런타임, 패키지 요구 사항을 정의하는 파일이 포함되어 있습니다.

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 사용할 라이브러리를 설치합니다.

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-ai-projects openai
    ```
1. 제공된 구성 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 **your_project_endpoint** 자리 표시자를 Azure AI 파운드리 포털의 **개요** 페이지에서 복사한 프로젝트의 **Azure AI Foundry 프로젝트 엔드포인트**로 바꾸고, **your_model_deployment** 자리 표시자를 gpt-4 모델 배포 이름으로 바꿉니다.
1. 자리 표시자를 바꾼 후 코드 편집기에서 **CTRL+S** 명령 또는 **마우스 오른쪽 단추 클릭 > 저장**을 사용하여 변경 내용을 저장한 다음 **CTRL+Q** 명령 또는 **마우스 오른쪽 단추 클릭 > 종료**를 사용하여 Cloud Shell 명령줄을 열어둔 채 코드 편집기를 닫습니다.

### 프로젝트에 연결하고 모델과 채팅하는 코드를 작성합니다.

> **팁**: 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다.

1. 제공된 코드 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code chat-app.py
    ```

1. 코드 파일에서 파일 상단에 추가된 기존 문에 주목하여 필요한 SDK 네임스페이스를 가져옵니다. 그런 다음 **참조 추가** 주석을 찾아 이전에 설치한 라이브러리의 네임스페이스를 참조하도록 다음 코드를 추가합니다.

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from openai import AzureOpenAI
    ```

1. **main** 함수의 **구성 설정 가져오기** 주석 아래에서 해당 코드가 구성 파일에서 정의한 프로젝트 연결 문자열 및 모델 배포 이름 값을 로드한다는 점에 유의합니다.
1. **프로젝트 클라이언트 초기화** 주석을 찾아 다음 코드를 추가하여 Azure AI Foundry 프로젝트에 연결합니다.

    > **팁**: 코드의 들여쓰기 수준을 올바르게 유지하도록 주의하세요.

    ```python
   # Initialize the project client
   project_client = AIProjectClient(            
            credential=DefaultAzureCredential(
                exclude_environment_credential=True,
                exclude_managed_identity_credential=True
            ),
            endpoint=project_endpoint,
        )
    ```

1. **채팅 클라이언트 가져오기** 주석을 찾아 다음 코드를 추가하여 모델과 채팅할 클라이언트 개체를 만듭니다.

    ```python
   # Get a chat client
   openai_client = project_client.get_openai_client(api_version="2024-10-21")
    ```

1. 주석 **시스템 메시지로 프롬프트 초기화**를 찾아 다음 코드를 추가하여 시스템 프롬프트로 메시지 모음을 초기화합니다.

    ```python
   # Initialize prompt with system message
   prompt = [
            {"role": "system", "content": "You are a helpful AI assistant that answers questions."}
        ]
    ```

1. 코드에는 사용자가 "종료"를 입력할 때까지 프롬프트를 입력할 수 있도록 하는 루프가 포함되어 있습니다. 그런 다음 루프 섹션에서 **채팅 완료 가져오기** 주석을 찾아 다음 코드를 추가하여 프롬프트에 사용자 입력을 추가하고 모델에서 완료를 검색한 다음 프롬프트에 완료를 추가합니다(향후 반복을 위한 채팅 기록 유지 목적).

    ```python
   # Get a chat completion
   prompt.append({"role": "user", "content": input_text})
   response = openai_client.chat.completions.create(
            model=model_deployment,
            messages=prompt)
   completion = response.choices[0].message.content
   print(completion)
   prompt.append({"role": "assistant", "content": completion})
    ```

1. **CTRL+S** 명령을 사용하여 변경 내용을 코드 파일에 저장합니다.

### Azure에 로그인하고 앱 실행

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 Azure에 로그인합니다.

    ```
   az login
    ```

    **<font color="red">Cloud Shell 세션이 이미 인증되었더라도 Azure에 로그인해야 합니다.</font>**

    > **참고**: 대부분의 시나리오에서는 *az login*을 사용하는 것만으로도 충분합니다. 그러나 여러 테넌트에 구독이 있는 경우 *--tenant* 매개 변수를 사용하여 테넌트 지정해야 할 수 있습니다. 자세한 내용은 [Sign into Azure interactively using the Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)를 참조하세요.
    
1. 메시지가 표시되면 지침에 따라 새 탭에서 로그인 페이지를 열고 제공된 인증 코드와 Azure 자격 증명을 입력합니다. 그런 다음 명령줄에서 로그인 프로세스를 완료하고 메시지가 표시되면 Azure AI 파운드리 허브가 포함된 구독을 선택합니다.
1. 로그인한 후 다음 명령을 입력하여 애플리케이션을 실행합니다.

    ```
   python chat-app.py
    ```

1. 메시지가 표시되면 `What is the fastest animal on Earth?`과 같은 질문을 입력하고 생성형 AI 모델의 응답을 검토합니다.
1. `Where can I see one?` 또는 `Are they endangered?` 등의 후속 질문을 해보세요. 대화는 각 반복에 대한 컨텍스트로 채팅 기록을 사용하여 계속되어야 합니다.
1. 완료되면 `quit`를 입력하여 프로그램을 종료합니다.

> **팁**: 속도 제한을 초과하여 앱이 실패하는 경우. 몇 초 정도 기다렸다가 다시 시도하세요. 구독에서 사용할 수 있는 할당량이 부족한 경우 모델이 응답하지 않을 수 있습니다.

## 요약

이 연습에서는 Azure AI 파운드리 SDK를 사용하여 Azure AI 파운드리 프로젝트에 배포한 생성형 AI 모델에 대한 클라이언트 응용 프로그램을 만들었습니다.

## 정리

Azure AI Foundry 포털 탐색을 완료한 경우 불필요한 Azure 비용이 발생하지 않도록 이 연습에서 만든 리소스를 삭제해야 합니다.

1. [Azure Portal](https://portal.azure.com)을 열고 이 연습에 사용된 리소스를 배포한 리소스 그룹의 내용을 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
