---
lab:
  title: 사용자 고유의 데이터를 사용하는 생성형 AI 앱 만들기
  description: RAG(검색 증강 생성) 모델을 사용하여 사용자 고유의 데이터로 프롬프트를 그라운딩하는 채팅 앱 빌드 방법을 알아봅니다.
---

# 사용자 고유의 데이터를 사용하는 생성형 AI 앱 만들기

RAG(검색 증강 생성)은 사용자 지정 데이터 소스의 데이터를 생성형 AI 모델에 대한 프롬프트에 통합하는 애플리케이션을 빌드하는 데 사용되는 기술입니다. RAG는 언어 모델을 사용하여 입력을 해석하고 적절한 응답을 생성하는 채팅 기반 애플리케이션인 생성형 AI 앱을 개발하는 데 일반적으로 사용되는 패턴입니다.

이 연습에서는 Azure AI 파운드리를 사용하여 사용자 지정 데이터를 생성형 AI 솔루션에 통합합니다.

이 연습에는 약 **45**분이 소요됩니다.

> **참고**: 이 연습은 출시 전 서비스를 기반으로 하며, 변경될 수 있습니다.

## Azure AI 파운드리 허브 및 프로젝트 만들기

이 연습에서 사용할 Azure AI 파운드리의 기능에는 Azure AI 파운드리 *허브* 리소스를 기반으로 하는 프로젝트가 필요합니다.

1. 웹 브라우저에서 [Azure AI 파운드리 포털](https://ai.azure.com)(`https://ai.azure.com`)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 처음 로그인할 때 열리는 팁이나 빠른 시작 창을 닫고, 필요한 경우 왼쪽 위에 있는 **Azure AI 파운드리** 로고를 사용하여 다음 이미지와 유사한 홈페이지로 이동합니다(**도움말** 창이 열려 있는 경우 닫습니다).

    ![Azure AI Foundry 포털의 스크린샷.](./media/ai-foundry-home.png)

1. 브라우저에서 `https://ai.azure.com/managementCenter/allResources`로 이동한 다음 **만들기**를 선택합니다. 그런 다음 새 **AI 허브 리소스**를 만드는 옵션을 선택합니다.
1. **프로젝트 만들기** 마법사에서 프로젝트의 유효한 이름을 입력하고, 기존 허브가 제안되는 경우 새 허브를 만드는 옵션을 선택한 다음 **고급 옵션**을 확장하여 프로젝트에 대해 다음 설정을 지정합니다.
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: ‘리소스 그룹 만들기 또는 선택’
    - **허브 이름**: 허브에서 유효한 이름
    - **위치**: 미국 동부 2 또는 스웨덴 중부\*

    > \* 일부 Azure AI 리소스는 지역 모델 할당량에 의해 제한됩니다. 연습 후반부에 할당량 한도를 초과하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다.

1. 프로젝트가 만들어질 때까지 기다립니다.

## 모델 배포

솔루션을 구현하려면 다음 두 가지 모델이 필요합니다.

- 효율적인 인덱싱 및 처리를 위해 텍스트 데이터를 벡터화하는 *임베딩* 모델입니다.
- 데이터에 따라 질문에 대한 자연어 응답을 생성할 수 있는 모델입니다.

1. Azure AI Foundry 포털의 프로젝트에 있는 왼쪽 탐색 창에서 **내 자산** 아래 **모델 + 엔드포인트** 페이지를 선택합니다.
1. 모델 배포 마법사에서 **사용자 지정**을 선택하여 다음 설정으로 **text-embedding-ada-002** 모델의 새 배포를 만듭니다.

    - **배포 이름**: *모델 배포에 대한 고유한 이름*
    - **배포 유형**: 글로벌 표준
    - **모델 버전**: *기본 버전 선택*
    - **연결된 AI 리소스**: *이전에 만든 리소스 선택*
    - **분당 토큰 속도 제한(천 단위)**: 50K *(또는 50K 이하인 경우 구독에서 사용 가능한 최대치)*
    - **콘텐츠 필터**: DefaultV2

    > **참고**: 배포하려는 모델에 사용할 수 있는 할당량이 현재 AI 리소스 위치에 없는 경우 다른 위치를 선택하여 새 AI 리소스를 만들고 프로젝트에 연결하라는 메시지가 표시됩니다.

1. **모델 + 엔드포인트** 페이지로 돌아가 이전 단계를 반복하여 TPM 속도 제한이 **50K**(50K 미만인 경우 구독에서 사용할 수 있는 최대값)인 최신 버전의 **글로벌 표준** 배포로 **gpt-4o** 모델을 배포합니다.

    > **참고**: TPM(분당 토큰)을 줄이면 사용 중인 구독에서 사용 가능한 할당량을 과도하게 사용하지 않을 수 있습니다. 이 연습에 사용되는 데이터는 50,000TPM이면 충분합니다.

## 프로젝트에 데이터 추가

앱 데이터는 가상의 여행사 *Margie's Travel*의 PDF 형식으로 된 여행 브로슈어 세트로 구성됩니다. 프로젝트에 이를 추가해 보겠습니다.

1. 새 브라우저 탭의 `https://github.com/MicrosoftLearning/mslearn-ai-studio/raw/main/data/brochures.zip`에서 [압축된 브로셔 아카이브](https://github.com/MicrosoftLearning/mslearn-ai-studio/raw/main/data/brochures.zip)를 다운로드하고 로컬 파일 시스템의 **브로셔**라는 폴더에 압축을 풉니다.
1. Azure AI Foundry 포털의 프로젝트에 있는 왼쪽 탐색 창에서 **내 자산** 아래 **데이터 + 인덱스** 페이지를 선택합니다.
1. **+ 새 데이터**를 선택합니다.
1. **데이터 추가** 마법사에서 드롭다운 메뉴를 확장하여 **파일/폴더 업로드**를 선택합니다.
1. **폴더 업로드**를 선택하고 **brochures** 폴더를 업로드합니다. 폴더의 모든 파일이 나열될 때까지 기다립니다.
1. **다음**을 선택하고 데이터 이름을 `brochures`(으)로 설정합니다.
1. 폴더가 업로드될 때까지 기다린 후 폴더에 여러 개의 .pdf 파일이 포함되어 있는지 확인합니다.

## 데이터에 대한 인덱스 만들기

이제 프로젝트에 데이터 원본을 추가했으므로 이 데이터 원본을 사용하여 Azure AI 검색 리소스에서 인덱스를 만들 수 있습니다.

1. Azure AI Foundry 포털의 프로젝트에 있는 왼쪽 탐색 창에서 **내 자산** 아래 **데이터 + 인덱스** 페이지를 선택합니다.
1. **인덱스** 탭에서 다음 설정을 사용하여 새 인덱스를 추가합니다.
    - **원본 위치**:
        - **데이터 원본**: Azure AI 파운드리의 데이터
            - ***brochures** 데이터 원본을 선택합니다.*
    - **인덱스 구성**:
        - **Azure AI 검색 서비스 선택**: *다음 설정으로 새 Azure AI 검색 리소스를 만듭니다*.
            - **구독**: *Azure 구독*
            - **리소스 그룹**: *AI 허브와 동일한 리소스 그룹*
            - **서비스 이름**: *AI 검색 리소스의 유효한 이름*
            - **위치**: *AI 허브와 동일한 위치*
            - **가격 책정 계층**: 기본
            
            AI 검색 리소스가 생성될 때까지 기다립니다. 그런 다음 Azure AI 파운드리로 돌아가서 **다른 Azure AI 검색 리소스 연결**을 선택하고 방금 만든 AI 검색 리소스에 연결을 추가하여 인덱스 구성을 마칩니다.
 
        - **벡터 인덱스**: `brochures-index`
        - **가상 머신**: 자동 선택
    - **검색 설정**:
        - **벡터 설정**: 이 검색 리소스에 벡터 검색을 추가합니다.
        - **Azure OpenAI 연결**: *허브에 대한 기본 Azure OpenAI 리소스를 선택합니다.*
        - **포함 모델**: text-embedding-ada-002
        - **포함 모델 배포**: * text-embedding-ada-002 *모델*의 배포*

1. 구독에서 사용 가능한 컴퓨팅 리소스에 따라 다소 시간이 걸릴 수 있는 벡터 인덱스를 만들 인덱싱 프로세스가 완료될 때까지 기다립니다.

    인덱스 만들기 작업은 다음 작업으로 구성됩니다.

    - 브로슈어 데이터에 텍스트 토큰을 해독, 청크 및 포함합니다.
    - Azure AI 검색 인덱스 만들기
    - 인덱스 자산 등록

    > **팁**: 인덱스가 생성되기를 기다리는 동안 다운로드한 브로셔를 살펴보고 내용을 알아보는 것은 어떨까요?

## 플레이그라운드에서 인덱스 테스트

RAG 기반 프롬프트 흐름에서 인덱스를 사용하기 전에 인덱스를 사용하여 생성형 AI 응답에 영향을 줄 수 있는지 확인해 보겠습니다.

1. 왼쪽 탐색 창에서 **플레이그라운드** 페이지를 선택하고 **채팅** 플레이그라운드를 엽니다.
1. 채팅 플레이그라운드 페이지의 설정 창에서 **gpt-4o** 모델 배포가 선택되어 있는지 확인합니다. 그런 다음 기본 채팅 세션 패널에서 프롬프트 `Where can I stay in New York?`를 제출합니다.
1. 응답을 검토합니다. 이때 응답은 인덱스의 데이터 없이 모델에서 생성된 일반적인 답변이어야 합니다.
1. 설정 창에서 **데이터 추가** 필드를 확장한 다음 **brochures-index** 프로젝트 인덱스를 추가하고 **하이브리드(벡터+키워드)** 검색 유형을 선택합니다.

   > **팁**: 경우에 따라 새로 만든 인덱스를 즉시 사용할 수 없습니다. 브라우저를 새로 고치면 일반적으로 도움이 되지만 인덱스를 찾을 수 없는 문제가 계속 발생하는 경우 인덱스가 인식될 때까지 기다려야 할 수도 있습니다.

1. 인덱스가 추가되고 채팅 세션이 다시 시작된 후 프롬프트 `Where can I stay in New York?`를 다시 제출합니다.
1. 응답을 검토합니다. 응답은 인덱스에 있는 데이터를 기반으로 해야 합니다.

<!-- DEPRECATED STEPS

## Create a RAG client app with the Azure AI Foundry and Azure OpenAI SDKs

Now that you have a working index, you can use the Azure AI Foundry and Azure OpenAI SDKs to implement the RAG pattern in a client application. Let's explore the code to accomplish this in a simple example.

> **Tip**: You can choose to develop your RAG solution using Python or Microsoft C#. Follow the instructions in the appropriate section for your chosen language.

### Prepare the application configuration

1. In the Azure AI Foundry portal, view the **Overview** page for your project.
1. In the **Project details** area, note the **Project connection string**. You'll use this connection string to connect to your project in a client application.
1. Return to the browser tab containing the Azure portal (keeping the Azure AI Foundry portal open in the existing tab).
1. Use the **[\>_]** button to the right of the search bar at the top of the page to create a new Cloud Shell in the Azure portal, selecting a ***PowerShell*** environment with no storage in your subscription.

    The cloud shell provides a command-line interface in a pane at the bottom of the Azure portal. You can resize or maximize this pane to make it easier to work in.

    > **Note**: If you have previously created a cloud shell that uses a *Bash* environment, switch it to ***PowerShell***.

1. In the cloud shell toolbar, in the **Settings** menu, select **Go to Classic version** (this is required to use the code editor).

    **<font color="red">Ensure you've switched to the classic version of the cloud shell before continuing.</font>**

1. In the cloud shell pane, enter the following commands to clone the GitHub repo containing the code files for this exercise (type the command, or copy it to the clipboard and then right-click in the command line and paste as plain text):

    ```
    rm -r mslearn-ai-foundry -f
    git clone https://github.com/microsoftlearning/mslearn-ai-studio mslearn-ai-foundry
    ```

    > **Tip**: As you paste commands into the cloudshell, the output may take up a large amount of the screen buffer. You can clear the screen by entering the `cls` command to make it easier to focus on each task.

1. After the repo has been cloned, navigate to the folder containing the chat application code files:

    > **Note**: Follow the steps for your chosen programming language.

    **Python**

    ```
   cd mslearn-ai-foundry/labfiles/rag-app/python
    ```

    **C#**

    ```
   cd mslearn-ai-foundry/labfiles/rag-app/c-sharp
    ```

1. In the cloud shell command-line pane, enter the following command to install the libraries you'll use:

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-ai-projects azure-identity openai
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --prerelease
   dotnet add package Azure.AI.OpenAI --prerelease
    ```
    

1. Enter the following command to edit the configuration file that has been provided:

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    The file is opened in a code editor.

1. In the code file, replace the following placeholders: 
    - **your_project_connection_string**: Replace with the connection string for your project (copied from the project **Overview** page in the Azure AI Foundry portal).
    - **your_gpt_model_deployment** Replace with the name you assigned to your **gpt-4o** model deployment.
    - **your_embedding_model_deployment**: Replace with the name you assigned to your **text-embedding-ada-002** model deployment.
    - **your_index**: Replace with your index name (which should be `brochures-index`).
1. After you've replaced the placeholders, in the code editor, use the **CTRL+S** command or **Right-click > Save** to save your changes and then use the **CTRL+Q** command or **Right-click > Quit** to close the code editor while keeping the cloud shell command line open.

### Explore code to implement the RAG pattern

1. Enter the following command to edit the code file that has been provided:

    **Python**

    ```
   code rag-app.py
    ```

    **C#**

    ```
   code Program.cs
    ```

1. Review the code in the file, noting that it:
    - Uses the Azure AI Foundry SDK to connect to your project (using the project connection string)
    - Creates an authenticated Azure OpenAI client from your project connection.
    - Retrieves the default Azure AI Search connection from your project so it can determine the endpoint and key for your Azure AI Search service.
    - Creates a suitable system message.
    - Submits a prompt (including the system and a user message based on the user input) to the Azure OpenAI client, adding:
        - Connection details for the Azure AI Search index to be queried.
        - Details of the embedding model to be used to vectorize the query\*.
    - Displays the response from the grounded prompt.
    - Adds the response to the chat history.

    \* *The query for the search index is based on the prompt, and is used to find relevant text in the indexed documents. You can use a keyword-based search that submits the query as text, but using a vector-based search can be more efficient - hence the use of an embedding model to vectorize the query text before submitting it.*

1. Use the **CTRL+Q** command to close the code editor without saving any changes, while keeping the cloud shell command line open.

### Run the chat application

1. In the cloud shell command-line pane, enter the following command to run the app:

    **Python**

    ```
   python rag-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. When prompted, enter a question, such as `Where should I go on vacation to see architecture?` and review the response from your generative AI model.

    Note that the response includes source references to indicate the indexed data in which the answer was found.

1. Try a follow-up question, for example `Where can I stay there?`

1. When you're finished, enter `quit` to exit the program. Then close the cloud shell pane.

-->

## 정리

불필요한 Azure 비용 및 리소스 사용률을 방지하려면 이 연습에서 배포한 리소스를 제거해야 합니다.

1. Azure AI Foundry 탐색을 마쳤다면 [Azure Portal](https://portal.azure.com)(`https://portal.azure.com`)로 돌아가서, 필요한 경우 Azure 자격 증명을 사용하여 로그인합니다. 그런 다음 Azure AI 검색 및 Azure AI 리소스를 프로비전한 리소스 그룹의 리소스를 삭제합니다.
