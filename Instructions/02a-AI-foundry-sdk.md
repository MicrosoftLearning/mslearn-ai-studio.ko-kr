---
lab:
  title: 생성형 AI 채팅 앱 만들기
  description: Azure AI 파운드리 SDK를 사용하여 프로젝트에 연결하고 언어 모델을 사용하여 채팅하는 앱을 빌드하는 방법을 알아봅니다.
---

# 생성형 AI 채팅 앱 만들기

이 연습에서는 Azure AI 파운드리 SDK를 사용하여 프로젝트에 연결하고 언어 모델로 채팅하는 간단한 채팅 앱을 만듭니다.

이 연습은 약 **40**분 정도 소요됩니다.

> **참고**: 이 연습은 변경될 수 있는 시험판 SDK를 기반으로 합니다. 필요한 경우 특정 버전의 패키지를 사용했으며, 이는 최신 버전이 반영되지 않을 수 있습니다.

## Azure AI 파운드리 프로젝트 만들기

먼저 Azure AI 파운드리 프로젝트를 만들어 보겠습니다.

1. 웹 브라우저에서 [Azure AI 파운드리 포털](https://ai.azure.com)(`https://ai.azure.com`)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 처음 로그인할 때 열리는 팁이나 빠른 시작 창을 닫고 필요한 경우 왼쪽 위에 있는 **Azure AI 파운드리** 로고를 사용하여 다음 이미지와 유사한 홈 페이지로 이동합니다.

    ![Azure AI Foundry 포털의 스크린샷.](./media/ai-foundry-home.png)

1. 홈페이지에서 **+ 프로젝트 만들기**를 선택합니다.
1. **프로젝트 만들기** 마법사에서 적절한 프로젝트 이름(예`my-ai-project`: )을 입력한 다음, 프로젝트를 지원하기 위해 자동으로 만들어지는 Azure 리소스를 검토합니다.
1. **사용자 지정**을 선택하고 허브에 대해 다음 설정을 지정합니다.
    - **허브 이름**: *고유한 이름 - 예: `my-ai-hub`*
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: *고유한 이름(예: `my-ai-resources`)으로 새 리소스 그룹을 만들거나 기존 리소스 그룹 선택*
    - **위치**: **선택 도움말**을 선택한 다음 위치 도우미 창에서 **gpt-4**를 선택하고 추천 지역을 사용합니다.\*
    - **Azure AI Services 또는 Azure OpenAI** 연결: *적절한 이름으로 새 AI Services 리소스를 만들거나(예 `my-ai-services`:) 기존 리소스를 사용합니다.*
    - **Azure AI 검색 연결**: 연결 건너뛰기

    > \* 연습 후반부에 지역 할당량 한도에 도달하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다.

1. **다음**을 선택하여 구성을 검토합니다. **만들기**를 선택하고 프로세스가 완료될 때까지 기다립니다.
1. 프로젝트를 만들 때 표시되는 팁을 모두 닫고 Azure AI 파운드리 포털에서 프로젝트 페이지를 검토합니다. 이 페이지는 다음 이미지와 유사합니다.

    ![Azure AI 파운드리 포털의 Azure AI 프로젝트 세부 정보 스크린샷.](./media/ai-foundry-project.png)

## 생성형 AI 모델 배포

이제 채팅 애플리케이션을 지원하기 위해 생성형 AI 언어 모델을 배포할 준비가 되었습니다. 이 예제에서는 OpenAI GPT-4 모델을 사용하지만 원칙은 모든 모델에 동일합니다.

1. Azure AI 파운드리 프로젝트 페이지의 오른쪽 위에 있는 도구 모음에서 **미리 보기 기능** 아이콘을 사용하여 **Azure AI 모델 유추 서비스 기능에 모델 배포**를 사용하도록 설정합니다. 이 기능을 사용하면 애플리케이션 코드에서 사용할 Azure AI 유추 서비스에서 모델 배포를 사용할 수 있습니다.
1. 프로젝트 왼쪽 창의 **내 자산** 섹션에서 **모델 + 엔드포인트** 페이지를 선택합니다.
1. **모델 + 엔드포인트** 페이지의 **모델 배포** 탭의 **+ 모델 배포** 메뉴에서 **기본 모델 배포**를 선택합니다.
1. 목록에서 **gpt-4** 모델을 검색하고 선택한 후 확인합니다.
1. 배포 세부 정보에서 **사용자 지정**을 선택하여 다음 설정으로 모델을 배포합니다.
    - **배포 이름**: *모델 배포에 대한 고유한 이름(예: `gpt-4`*)
    - **배포 유형**: 글로벌 표준
    - **모델 버전**: *기본 버전 선택*
    - **연결된 AI 리소스**: *Azure OpenAI 리소스 연결*
    - **분당 토큰 속도 제한(천 개)**: 5K(*그 이하인 경우 사용 가능한 최대치*)
    - **콘텐츠 필터**: DefaultV2
    - **동적 할당량 사용**: 사용할 수 없음

    > **참고**: TPM을 줄이면 사용 중인 구독에서 사용 가능한 할당량을 과도하게 사용하지 않을 수 있습니다. 이 연습에 사용된 데이터는 5,000TPM이면 충분합니다.

1. 배포 프로비전 상태가 **완료**될 때까지 기다립니다.

## 모델과 채팅할 클라이언트 응용 프로그램 만들기

이제 모델을 배포했으므로 Azure AI 파운드리 및 Azure AI 모델 추론 SDK를 사용하여 이 모델과 채팅하는 애플리케이션을 개발할 수 있습니다.

> **팁**: Python 또는 Microsoft C#을 사용하여 솔루션을 개발하도록 선택할 수 있습니다. 선택한 언어의 해당 섹션에 있는 지침을 따릅니다.

### 애플리케이션 구성 준비

1. Azure AI 파운드리 포털에서 프로젝트의 **개요** 페이지를 봅니다.
1. **프로젝트 세부 정보** 영역에서 **프로젝트 연결 문자열**을 확인합니다. 이 연결 문자열 사용하여 클라이언트 응용 프로그램에서 프로젝트에 연결합니다.
1. 새 브라우저 탭을 엽니다(Azure AI 파운드리 포털을 기존 탭에서 열어 두기). 그런 다음 새 탭에서 [Azure Portal](https://portal.azure.com)(`https://portal.azure.com`)을 열고 메시지가 나타나면 Azure 자격 증명을 사용하여 로그인합니다.
1. 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 ***PowerShell*** 환경을 선택합니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

    > **참고**: 이전에 *Bash* 환경을 사용하는 Cloud Shell을 만든 경우 ***PowerShell***로 전환합니다.

1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다(코드 편집기를 사용하는 데 필요).

    **<font color="red">계속하기 전에 Cloud Shell의 클래식 버전으로 전환했는지 확인합니다.</font>**

1. PowerShell 창에서 다음 명령을 입력하여 이 연습이 포함된 GitHub 리포지토리를 복제합니다.

    ```
    rm -r mslearn-ai-foundry -f
    git clone https://github.com/microsoftlearning/mslearn-ai-studio mslearn-ai-foundry
    ```

    > **팁**: CloudShell에 명령을 붙여넣을 때, 출력이 화면 버퍼의 많은 부분을 차지할 수 있습니다. `cls` 명령을 입력해 화면을 지우면 각 작업에 더 집중할 수 있습니다.

1. 리포지토리가 복제된 후 채팅 애플리케이션 코드 파일이 포함된 폴더로 이동합니다.

    선택한 프로그래밍 언어에 따라 아래 명령을 사용합니다.

    **Python**

    ```
   cd mslearn-ai-foundry/labfiles/chat-app/python
    ```

    **C#**

    ```
   cd mslearn-ai-foundry/labfiles/chat-app/c-sharp
    ```

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 사용할 라이브러리를 설치합니다.

    **Python**

    ```
   pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --version 1.0.0-beta.3
   dotnet add package Azure.AI.Inference --version 1.0.0-beta.3
    ```
    

1. 제공된 구성 파일을 편집하려면 다음 명령을 입력합니다.

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 **your_project_endpoint** 자리 표시자를 프로젝트의 연결 문자열(Azure AI 파운드리 포털의 프로젝트 **개요** 페이지에서 복사함)로 바꾸고, **your_model_deployment** 자리 표시자를 GPT-4 모델 배포에 할당한 이름으로 바꿉니다.
1. 자리 표시자를 바꾼 후 코드 편집기에서 **CTRL+S** 명령 또는 **마우스 오른쪽 단추 클릭 > 저장**을 사용하여 변경 내용을 저장한 다음 **CTRL+Q** 명령 또는 **마우스 오른쪽 단추 클릭 > 종료**를 사용하여 Cloud Shell 명령줄을 열어둔 채 코드 편집기를 닫습니다.

### 프로젝트에 연결하고 모델과 채팅하는 코드를 작성합니다.

> **팁**: 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다.

1. 제공된 코드 파일을 편집하려면 다음 명령을 입력합니다.

    **Python**

    ```
   code chat-app.py
    ```

    **C#**

    ```
   code Program.cs
    ```

1. 코드 파일에서 파일 상단에 추가된 기존 문에 주목하여 필요한 SDK 네임스페이스를 가져옵니다. 그런 다음 **참조 추가** 주석을 찾아 이전에 설치한 라이브러리의 네임스페이스를 참조하도록 다음 코드를 추가합니다.

    **Python**

    ```python
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.inference.models import SystemMessage, UserMessage, AssistantMessage
    ```

    **C#**

    ```csharp
   using Azure.Identity;
   using Azure.AI.Projects;
   using Azure.AI.Inference;
    ```

1. **main** 함수의 **구성 설정 가져오기** 주석 아래에서 해당 코드가 구성 파일에서 정의한 프로젝트 연결 문자열 및 모델 배포 이름 값을 로드한다는 점에 유의합니다.
1. **프로젝트 클라이언트 초기화** 주석을 찾아 다음 코드를 추가하여 현재 로그인한 Azure 자격 증명을 사용하여 Azure AI 파운드리 프로젝트에 연결합니다.

    > **팁**: 코드의 들여쓰기 수준을 올바르게 유지하도록 주의하세요.

    **Python**

    ```python
   projectClient = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```csharp
   var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
    ```

1. **채팅 클라이언트 가져오기** 주석을 찾아 다음 코드를 추가하여 모델과 채팅할 클라이언트 개체를 만듭니다.

    **Python**

    ```python
   chat = projectClient.inference.get_chat_completions_client()
    ```

    **C#**

    ```csharp
   ChatCompletionsClient chat = projectClient.GetChatCompletionsClient();
    ```

    > **참고**: 이 코드는 Azure AI 파운드리 프로젝트 클라이언트를 사용하여 프로젝트와 연결된 기본 Azure AI 모델 추론 서비스 엔드포인트에 대한 보안 연결을 만듭니다. 또한 Azure AI 모델 추론 SDK를 사용하여 Azure AI 파운드리 포털 또는 Azure Portal의 해당 Azure AI 서비스 리소스 페이지에서 서비스 연결을 위해 표시되는 엔드포인트 URI를 지정하고 인증 키 또는 Entra 자격 증명 토큰을 사용하여 엔드포인트에 *직접* 연결할 수도 있습니다. Azure AI 모델 추론 서비스 연결에 대한 자세한 내용은 [Azure AI 모델 추론 API](https://learn.microsoft.com/azure/machine-learning/reference-model-inference-api)를 참조합니다.

1. 주석 **시스템 메시지로 프롬프트 초기화**를 찾아 다음 코드를 추가하여 시스템 프롬프트로 메시지 모음을 초기화합니다.

    **Python**

    ```python
   prompt=[
            SystemMessage("You are a helpful AI assistant that answers questions.")
        ]
    ```

    **C#**

    ```csharp
    var prompt = new List<ChatRequestMessage>(){
                    new ChatRequestSystemMessage("You are a helpful AI assistant that answers questions.")
                };
    ```

1. 코드에는 사용자가 "종료"를 입력할 때까지 프롬프트를 입력할 수 있도록 하는 루프가 포함되어 있습니다. 그런 다음 루프 섹션에서 **채팅 완료 가져오기** 주석을 찾아 다음 코드를 추가하여 프롬프트에 사용자 입력을 추가하고 모델에서 완료를 검색한 다음 프롬프트에 완료를 추가합니다(향후 반복을 위한 채팅 기록 유지 목적).

    **Python**

    ```python
   prompt.append(UserMessage(input_text))
   response = chat.complete(
       model=model_deployment,
       messages=prompt)
   completion = response.choices[0].message.content
   print(completion)
   prompt.append(AssistantMessage(completion))
    ```

    **C#**

    ```csharp
   prompt.Add(new ChatRequestUserMessage(input_text));
   var requestOptions = new ChatCompletionsOptions()
   {
       Model = model_deployment,
       Messages = prompt
   };

   Response<ChatCompletions> response = chat.Complete(requestOptions);
   var completion = response.Value.Content;
   Console.WriteLine(completion);
   prompt.Add(new ChatRequestAssistantMessage(completion));
    ```

1. **CTRL+S** 명령을 사용하여 변경 내용을 코드 파일에 저장합니다.

### 채팅 애플리케이션 실행

1. 코드 편집기 아래의 Cloud Shell 명령줄 창에서 다음 명령을 입력하여 앱을 실행합니다.

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. 메시지가 표시되면 `What is the fastest animal on Earth?`과 같은 질문을 입력하고 생성형 AI 모델의 응답을 검토합니다.
1. `Where can I see one?` 또는 `Are they endangered?` 등의 후속 질문을 해보세요. 대화는 각 반복에 대한 컨텍스트로 채팅 기록을 사용하여 계속되어야 합니다.
1. 완료되면 `quit`를 입력하여 프로그램을 종료합니다.

## OpenAI SDK 사용

클라이언트 앱은 Azure AI 모델 추론 SDK를 사용하여 빌드됩니다. 즉, Azure AI 모델 추론 서비스에 배포된 모든 모델과 함께 사용할 수 있습니다. 배포한 모델은 OpenAI SDK로 사용할 수 있는 OpenAI GPT 모델입니다.

OpenAI SDK를 사용하여 채팅 애플리케이션을 구현하는 방법을 확인하기 위해 몇 가지 코드를 수정해 보겠습니다.

1. 코드 폴더의 Cloud Shell 명령줄(*python* 또는 *c-sharp*)에 다음 명령을 입력하여 필요한 패키지를 설치합니다.

    **Python**

    ```
   pip install openai
    ```

    **C#**

    ```
   dotnet add package Azure.AI.Projects --version 1.0.0-beta.5
   dotnet add package Azure.AI.OpenAI --prerelease
    ```

> **참고**: Azure AI 모델 추론 SDK와의 일부 비호환성에 대한 임시 해결 방법으로 Azure.AI.Projects 패키지의 다른 시험판 버전이 필요합니다.

1. 코드 파일(*chat-app.py* 또는 *Program.cs*)이 아직 열려 있지 않으면 다음 명령을 입력하여 코드 편집기에서 엽니다.

    **Python**

    ```
   code chat-app.py
    ```

    **C#**

    ```
   code Program.cs
    ```

1. 코드 파일 상단에 다음 참조를 추가합니다.

    **Python**

    ```python
   import openai
    ```

    **C#**

    ```csharp
   using OpenAI.Chat;
   using Azure.AI.OpenAI;
    ```

1. **채팅 클라이언트 가져오기** 주석을 찾아 클라이언트 개체를 만드는 데 사용된 코드를 다음과 같이 수정합니다.

    **Python**

    ```python
   openai_client = projectClient.inference.get_azure_openai_client(api_version="2024-10-21")
    ```

    **C#**

    ```csharp
   ChatClient openaiClient = projectClient.GetAzureOpenAIChatClient(model_deployment);
    ```

    > **참고**: 이 코드는 Azure AI 파운드리 프로젝트 클라이언트를 사용하여 프로젝트와 연결된 기본 Azure OpenAI 서비스 엔드포인트에 대한 보안 연결을 만듭니다. 또한 Azure OpenAI SDK를 사용하여 Azure AI 파운드리 포털 또는 Azure Portal의 해당 Azure OpenAI 또는 AI 서비스 리소스 페이지에서 서비스 연결을 위해 표시되는 엔드포인트 URI를 지정하고 인증 키 또는 Entra 자격 증명 토큰을 사용하여 엔드포인트에 *직접* 연결할 수도 있습니다. Azure OpenAI Service 연결에 대한 자세한 내용은 [Azure OpenAI 지원 프로그래밍 언어](https://learn.microsoft.com/azure/ai-services/openai/supported-languages)를 참조하세요.

1. **시스템 메시지로 프롬프트 초기화** 주석을 찾아 다음과 같이 시스템 프롬프트로 메시지 모음을 초기화하도록 코드를 수정합니다.

    **Python**

    ```python
   prompt=[
       {"role": "system", "content": "You are a helpful AI assistant that answers questions."}
   ]
    ```

    **C#**

    ```csharp
    var prompt = new List<ChatMessage>(){
       new SystemChatMessage("You are a helpful AI assistant that answers questions.")
   };
    ```

1. **채팅 완료 가져오기** 주석을 찾아 사용자 입력을 프롬프트에 추가하도록 코드를 수정하고, 모델에서 완료를 검색한 다음 다음과 같이 프롬프트에 완료를 추가합니다.

    **Python**

    ```python
   prompt.append({"role": "user", "content": input_text})
   response = openai_client.chat.completions.create(
       model=model_deployment,
       messages=prompt)
   completion = response.choices[0].message.content
   print(completion)
   prompt.append({"role": "assistant", "content": completion})
    ```

    **C#**

    ```csharp
   prompt.Add(new UserChatMessage(input_text));
   ChatCompletion completion = openaiClient.CompleteChat(prompt);
   var completionText = completion.Content[0].Text;
   Console.WriteLine(completionText);
   prompt.Add(new AssistantChatMessage(completionText));
    ```

1. **CTRL+S** 명령을 사용하여 변경 내용을 코드 파일에 저장합니다.

1. 코드 편집기 아래의 Cloud Shell 명령줄 창에서 다음 명령을 입력하여 앱을 실행합니다.

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. 이전과 같이 질문을 제출하여 앱을 테스트합니다. 완료되면 `quit`를 입력하여 프로그램을 종료합니다.

    > **참고**: Azure AI 모델 추론 SDK와 OpenAI SDK는 유사한 클래스 및 코드 구성을 사용하므로 코드 변경이 거의 필요하지 않습니다. Azure AI 모델 추론 서비스 엔드포인트에 배포된 *모든* 모델과 함께 Azure AI 모델 추론 SDK를 사용할 수 있습니다. OpenAI SDK는 OpenAI 모델에서만 작동하지만 Azure AI 모델 추론 서비스 엔드포인트 또는 Azure OpenAI 엔드포인트에 배포된 모델에 사용할 수 있습니다.  

## 요약

이 연습에서는 Azure AI 파운드리, Azure AI 모델 추론 및 Azure OpenAI SDK를 사용하여 Azure AI 파운드리 프로젝트에 배포한 생성형 AI 모델용 클라이언트 애플리케이션을 만들었습니다.

## 정리

Azure AI Foundry 포털 탐색을 완료한 경우 불필요한 Azure 비용이 발생하지 않도록 이 연습에서 만든 리소스를 삭제해야 합니다.

1. Azure Portal이 포함된 브라우저 탭으로 돌아가서(또는 새 브라우저 탭의 `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)을 다시 열고) 이 연습에 사용된 리소스를 배포한 리소스 그룹의 콘텐츠를 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
