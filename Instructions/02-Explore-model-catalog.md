---
lab:
  title: 언어 모델 선택 및 배포
  description: 생성형 AI 애플리케이션은 하나 이상의 언어 모델을 기반으로 구축됩니다. 생성형 AI 프로젝트에 적합한 모델을 찾고 선택하는 방법을 알아봅니다.
---

# 언어 모델 선택 및 배포

Azure AI Foundry의 모델 카탈로그는 다양한 모델을 탐색하고 사용할 수 있는 중앙 리포지토리 역할을 하므로 생성형 AI 시나리오를 쉽게 만들 수 있습니다.

이 연습에서는 Azure AI 파운드리 포털의 모델 카탈로그를 살펴보고 문제 해결을 지원하는 생성형 AI 애플리케이션의 잠재 모델을 비교합니다.

이 연습은 약 **25**분 정도 소요됩니다.

> **참고**: 이 연습에 사용된 일부 기술은 미리 보기이거나 현재 개발 중에 있습니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

## 모델 살펴보기

먼저 Azure AI 파운드리 포털에 로그인하여 사용 가능한 모델 몇 가지를 살펴봅시다.

1. 웹 브라우저에서 [Azure AI 파운드리 포털](https://ai.azure.com)(`https://ai.azure.com`)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 처음 로그인할 때 열리는 팁이나 빠른 시작 창을 닫고, 필요한 경우 왼쪽 위에 있는 **Azure AI 파운드리** 로고를 사용하여 다음 이미지와 유사한 홈페이지로 이동합니다(**도움말** 창이 열려 있는 경우 닫습니다).

    ![Azure AI Foundry 포털의 스크린샷.](./media/ai-foundry-home.png)

1. 홈페이지의 정보를 리뷰합니다.
1. 홈페이지의 **모델 및 기능 탐색** 섹션에서 프로젝트에 사용할 `gpt-4o` 모델을 검색합니다.
1. 검색 결과에서 **GPT-4o** 모델을 선택하면 세부 정보를 확인할 수 있습니다.
1. 설명을 읽고 **세부 정보** 탭에서 제공되는 다른 정보를 검토합니다.

    ![gpt-4o 모델 세부 정보 페이지의 스크린샷.](./media/gpt4-details.png)

1. **gpt-4o** 페이지의 **벤치마크** 탭에서 몇 가지 표준 성능 벤치마크에서 모델이 유사한 시나리오에 사용되는 다른 모델과 어떻게 비교되는지 확인합니다.

    ![gpt-4o 모델 벤치마크 페이지의 스크린샷.](./media/gpt4-benchmarks.png)

1. 모델 카탈로그로 돌아가려면 **GPT-4o** 페이지 제목 옆의 뒤로 화살표(**&larr;**)를 사용합니다.
1. `Phi-4-mini-instruct`를 검색하여 **Phi-4-mini-instruct** 모델에 대한 세부 정보 및 벤치마크를 확인합니다.

## 모델 비교

생성형 AI 채팅 애플리케이션을 구현하는 데 사용할 수 있는 두 가지 모델을 검토해 보았습니다. 이제 이 두 모델의 메트릭을 시각적으로 비교해 보겠습니다.

1. 모델 카탈로그로 돌아가려면 뒤로 화살표(**&larr;**)를 사용합니다.
1. **모델 비교**를 선택합니다. 모델 비교를 위한 시각적 차트가 몇 가지 일반적인 모델과 함께 표시됩니다.

    ![모델 비교 페이지의 스크린샷.](./media/compare-models.png)

1. **비교할 모델** 창에서 *질문 답변*과 같이 인기 있는 작업을 선택하면 특정 작업에 일반적으로 사용되는 모델을 자동으로 선택할 수 있습니다.
1. **모든 모델 지우기**(&#128465;) 아이콘을 사용하여 미리 선택한 모델을 모두 제거합니다.
1. **+ 비교할 모델** 단추를 사용하여 **gpt-4o** 모델을 목록에 추가합니다. 그런 다음, 같은 버튼을 사용하여 **Phi-4-mini-instruct** 모델을 목록에 추가합니다.
1. **품질 지수**(모델 품질을 나타내는 표준화된 점수)와 **비용**을 기준으로 모델을 비교한 차트를 검토합니다. 차트에서 모델을 나타내는 지점 위에 마우스를 올려놓으면 모델의 특정 값을 볼 수 있습니다.

    ![gpt-4o 및 Phi-4-mini-instruct의 모델 비교 차트 스크린샷](./media/comparison-chart.png)

1. **X축** 드롭다운 메뉴에서 **품질** 아래에 있는 다음 메트릭을 선택하고 다음으로 전환하기 전에 각 결과 차트를 관찰합니다.
    - 정확도(Accuracy)
    - 품질 인덱스

    벤치마크에 따르면 GPT-4o 모델이 전반적인 성능은 가장 우수하지만 비용이 더 높은 것으로 보입니다.

1. 비교할 모델 목록에서 **GPT-4o** 모델을 선택하면 해당 벤치마크 페이지가 다시 열립니다.
1. **GPT-4o** 모델 페이지에서 **개요** 탭을 선택하면 모델 세부 정보를 볼 수 있습니다.

## Azure AI 파운드리 프로젝트 만들기

모델을 사용하려면 Azure AI 파운드리 *프로젝트*를 만들어야 합니다.

1. **GPT-4o** 모델 개요 페이지 상단에서 **이 모델 사용**을 선택합니다.
1. 프로젝트를 만들라는 메시지가 표시되면 프로젝트의 유효한 이름을 입력하고 **고급 옵션**을 펼칩니다.
1. **고급 옵션** 섹션에서 프로젝트에 대한 다음 설정을 지정합니다.
    - **Azure AI 파운드리 리소스**: *Azure AI 파운드리 리소스의 유효한 이름*
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: ‘리소스 그룹 만들기 또는 선택’
    - **지역**: **AI 서비스 지원 위치 *선택***\*

    > \* 일부 Azure AI 리소스는 지역 모델 할당량에 의해 제한됩니다. 연습 후반부에 할당량 한도를 초과하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다.

1. **만들기**를 선택하고 선택한 GPT-4 모델 배포를 포함한 프로젝트가 생성될 때까지 기다립니다.
1. 프로젝트가 생성되면 채팅 플레이그라운드가 자동으로 열리므로 모델을 테스트할 수 있습니다.

    ![Azure AI 파운드리 프로젝트 채팅 플레이그라운드의 스크린샷.](./media/ai-foundry-chat-playground.png)

## *gpt-4o* 모델과 채팅

이제 모델 배포가 있으므로 플레이그라운드를 사용하여 테스트할 수 있습니다.

1. 채팅 플레이그라운드의 **설정** 창에서 **GPT-4o** 모델이 선택되어 있는지 확인하고 **모델 지침 및 컨텍스트 제공** 필드에서 `You are an AI assistant that helps solve problems.`(으)로 시스템 프롬프트를 설정합니다.
1. **변경 내용 적용**을 선택하여 시스템 프롬프트를 업데이트합니다.

1. 채팅창에 다음 쿼리를 입력합니다.

    ```
   I have a fox, a chicken, and a bag of grain that I need to take over a river in a boat. I can only take one thing at a time. If I leave the chicken and the grain unattended, the chicken will eat the grain. If I leave the fox and the chicken unattended, the fox will eat the chicken. How can I get all three things across the river without anything being eaten?
    ```

1. 응답을 봅니다. 그런 다음 다음의 후속 쿼리를 입력합니다.

    ```
   Explain your reasoning.
    ```

## 또 다른 모델 배포

프로젝트를 만들 때 선택한 **GPT-4o** 모델이 자동으로 배포되었습니다. 또한 고려했던 ***Phi-4-mini-instruct** 모델을 배포해 보겠습니다.

1. 왼쪽 탐색 표시줄의 **내 자산** 섹션에서 **모델 + 엔드포인트**를 선택합니다.
1. **모델 배포** 탭의 **+ 모델 배포** 드롭다운 목록에서 **베이스 모델 배포**를 선택합니다. 그런 다음 `Phi-4-mini-instruct`을(를) 검색하고 선택을 확인합니다.
1. 모델 라이선스에 동의합니다.
1. 다음 설정으로 **Phi-4-mini-instruct** 모델을 배포합니다.
    - **배포 이름**: *모델 배포에 대한 유효한 이름*
    - **배포 유형**: 글로벌 표준
    - **배포 세부 정보**: *기본 설정 사용*

1. 배포가 완료될 때가지 기다립니다.

## *Phi-4* 모델과 채팅

이제 플레이그라운드에서 새 모델과 채팅해 보겠습니다.

1. 탐색 모음에서 **플레이그라운드**를 선택합니다. 그런 다음 **채팅 플레이그라운드**를 선택합니다.
1. 채팅 플레이그라운드의 **설정** 창에서 **Phi-4-mini-instruct** 모델이 선택되어 있는지 확인하고 채팅 상자에서 첫 번째 줄을 `System message: You are an AI assistant that helps solve problems.`으로 제공합니다(gpt-4o 모델을 테스트하는 데 사용한 것과 동일한 시스템 프롬프트이지만 시스템 메시지 설정이 없으므로 컨텍스트에 대한 첫 번째 채팅에서 제공).
1. 채팅 창의 새 줄(시스템 메시지 아래)에서 다음 쿼리를 입력합니다.

    ```
   I have a fox, a chicken, and a bag of grain that I need to take over a river in a boat. I can only take one thing at a time. If I leave the chicken and the grain unattended, the chicken will eat the grain. If I leave the fox and the chicken unattended, the fox will eat the chicken. How can I get all three things across the river without anything being eaten?
    ```

1. 응답을 봅니다. 그런 다음 다음의 후속 쿼리를 입력합니다.

    ```
   Explain your reasoning.
    ```

## 추가 비교 수행

1. **설정** 창의 드롭다운 목록을 사용하여 모델 간에 전환하고 다음 퍼즐을 통해 두 모델을 모두 테스트합니다(올바른 답변은 40입니다!).

    ```
   I have 53 socks in my drawer: 21 identical blue, 15 identical black and 17 identical red. The lights are out, and it is completely dark. How many socks must I take out to make 100 percent certain I have at least one pair of black socks?
    ```

## 모델에 대한 고찰

적절한 응답을 생성하는 능력과 비용 측면에서 차이가 있을 수 있는 두 가지 모델을 비교했습니다. 모든 생성형 시나리오에서 수행해야 하는 작업에 대한 적합성과 처리해야 할 것으로 예상되는 요청 수에 대한 모델 사용 비용 간의 적절한 균형을 갖춘 모델을 찾아야 합니다.

모델 카탈로그에 제공되는 세부 정보와 벤치마크는 모델을 시각적으로 비교할 수 있는 기능과 함께 생성형 AI 솔루션의 후보 모델을 식별할 때 유용한 출발점을 제공합니다. 그런 다음 채팅 플레이그라운드에서 다양한 시스템 및 사용자 프롬프트를 사용하여 후보 모델을 테스트할 수 있습니다.

## 정리

Azure AI Foundry 포털 탐색을 완료한 경우 불필요한 Azure 비용이 발생하지 않도록 이 연습에서 만든 리소스를 삭제해야 합니다.

1. [Azure Portal](https://portal.azure.com)을 열고 이 연습에 사용된 리소스를 배포한 리소스 그룹의 내용을 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
