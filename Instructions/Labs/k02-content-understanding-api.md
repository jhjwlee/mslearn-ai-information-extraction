---
lab:
    title: 'Content Understanding 클라이언트 애플리케이션 개발'
    description: 'Azure AI Content Understanding REST API를 사용하여 분석기를 위한 클라이언트 앱을 개발합니다.'
---

# Content Understanding 클라이언트 애플리케이션 개발

이 실습에서는 Azure AI Content Understanding을 사용하여 명함에서 정보를 추출하는 분석기(analyzer)를 생성합니다. 그런 다음, 이 분석기를 활용하여 스캔된 명함 이미지에서 연락처 세부 정보를 추출하는 클라이언트 애플리케이션을 개발합니다.

이 실습을 완료하는 데는 약 **30**분이 소요됩니다.

## Azure AI Foundry 허브 및 프로젝트 생성

이 실습에서 사용할 Azure AI Foundry의 기능들은 Azure AI Foundry *허브(hub)* 리소스를 기반으로 하는 프로젝트가 필요합니다. 먼저 이러한 기반 환경을 구축하는 것부터 시작하겠습니다.

1.  웹 브라우저에서 `https://ai.azure.com` 주소로 [Azure AI Foundry 포털](https://ai.azure.com)을 열고 Azure 자격 증명으로 로그인합니다. 처음 로그인할 때 나타나는 팁이나 빠른 시작 창은 모두 닫아주세요. 필요한 경우, 왼쪽 상단의 **Azure AI Foundry** 로고를 클릭하여 다음과 유사한 홈페이지로 이동합니다(도움말 창이 열려 있다면 닫아주세요).


    ![Screenshot of Azure AI Foundry portal.](./media/ai-foundry-home.png)

2.  브라우저에서 `https://ai.azure.com/managementCenter/allResources`로 이동하여 **Create new**를 선택합니다. 그런 다음 **AI hub resource**를 새로 생성하는 옵션을 선택합니다.
3.  **Create a project** 마법사에서 프로젝트에 유효한 이름을 입력하고, 새 허브를 생성하는 옵션을 선택합니다. **Rename hub** 링크를 사용하여 새 허브의 이름을 지정하고, **Advanced options**를 확장한 후 프로젝트에 대해 다음 설정을 지정합니다.
    -   **Subscription**: *사용 중인 Azure 구독*
    -   **Resource group**: *기존 리소스 그룹을 선택하거나 새로 생성*
    -   **Hub name**: 허브에 사용할 유효한 이름
    -   **Location**: 다음 위치 중 하나를 선택하세요:\*
        -   Australia East
        -   Sweden Central
        -   West US

    > \*이 문서를 작성하는 시점에서 Azure AI Content Understanding은 이 지역들에서만 사용할 수 있습니다. 이 점을 반드시 확인하고 진행해주세요.

    > **팁**: 만약 **Create** 버튼이 비활성화 상태라면, 허브 이름을 고유한 영숫자 값으로 변경해보세요.

4.  프로젝트 생성이 완료될 때까지 기다린 후, 프로젝트 개요(overview) 페이지로 이동합니다.

## REST API를 사용하여 Content Understanding 분석기 생성하기

이제 REST API를 사용하여 명함 이미지에서 정보를 추출할 수 있는 분석기(analyzer)를 생성해 보겠습니다. REST API는 웹을 통해 서비스와 통신하기 위한 표준적인 방법으로, 이를 통해 Azure의 AI 기능을 프로그래밍 방식으로 제어할 수 있습니다.

1.  새 브라우저 탭을 열고(기존 Azure AI Foundry 포털 탭은 그대로 둡니다), `https://portal.azure.com` 주소로 [Azure portal](https://portal.azure.com)에 접속합니다. 필요하다면 Azure 자격 증명으로 로그인합니다.

    환영 알림이 나타나면 닫고 Azure Portal 홈페이지를 확인합니다.

2.  페이지 상단 검색창 오른쪽의 **[\>_]** 버튼을 클릭하여 Azure Portal에서 새로운 Cloud Shell을 시작합니다. 환경 선택 창이 나타나면 ***PowerShell*** 환경을 선택하고, 구독 내에 스토리지가 없다면 새로 생성합니다.

    Cloud Shell은 Azure Portal 하단 창에 명령줄 인터페이스를 제공합니다. 작업의 편의를 위해 이 창의 크기를 조절하거나 최대화할 수 있습니다.

    > **팁**: 창 크기를 조절하여 Cloud Shell에서 주로 작업하면서도, Azure Portal 페이지에 있는 키와 엔드포인트 정보를 쉽게 복사하여 코드에 붙여넣을 수 있도록 하세요.

3.  Cloud Shell 툴바의 **Settings** 메뉴에서 **Go to Classic version**을 선택합니다. (이 작업은 코드 편집기를 사용하기 위해 필요합니다.)

    **<font color="red">계속 진행하기 전에 반드시 Cloud Shell이 클래식 버전으로 전환되었는지 확인하세요.</font>**

4.  Cloud Shell 창에 다음 명령어를 입력하여 이 실습에 필요한 코드 파일이 포함된 GitHub 리포지토리를 복제(clone)합니다. (명령어를 직접 입력하거나, 클립보드에 복사한 후 명령줄에서 마우스 오른쪽 버튼을 클릭하여 일반 텍스트로 붙여넣으세요):

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **팁**: Cloud Shell에 명령어를 입력하다 보면 출력이 화면을 가득 채울 수 있습니다. 이때 `cls` 명령어를 입력하면 화면을 깨끗하게 정리하여 각 작업에 더 집중할 수 있습니다.

5.  리포지토리 복제가 완료되면, 애플리케이션 코드 파일이 있는 폴더로 이동합니다.

    ```
   cd mslearn-ai-info/Labfiles/content-app
   ls -a -l
    ```

    이 폴더에는 두 개의 스캔된 명함 이미지 파일과 앱을 빌드하는 데 필요한 Python 코드 파일들이 포함되어 있습니다.

6.  Cloud Shell 명령줄 창에서 다음 명령어를 실행하여 사용할 라이브러리들을 설치합니다.

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt
    ```

7.  제공된 설정 파일을 편집하기 위해 다음 명령어를 입력합니다.

    ```
   code .env
    ```

    이 명령을 실행하면 코드 편집기에서 `.env` 파일이 열립니다. 이 파일은 API 키와 같은 민감한 정보를 코드와 분리하여 안전하게 관리하는 데 사용됩니다.

8.  코드 파일에서 **YOUR_ENDPOINT**와 **YOUR_KEY** 자리 표시자를 Azure Portal에서 복사한 Azure AI 서비스의 엔드포인트(endpoint)와 키(key) 값 중 하나로 각각 교체합니다. 그리고 **ANALYZER_NAME**이 `business-card-analyzer`로 설정되어 있는지 확인합니다.
9.  자리 표시자를 모두 교체한 후, 코드 편집기 내에서 **CTRL+S**를 눌러 변경 사항을 저장하고, **CTRL+Q**를 눌러 코드 편집기를 닫습니다. Cloud Shell 명령줄은 계속 열어 둡니다.

    > **팁**: 이제 Cloud Shell 창을 최대화하여 작업하는 것이 편리할 수 있습니다.

10. Cloud Shell 명령줄에서 다음 명령어를 입력하여 제공된 **biz-card.json** JSON 파일을 확인합니다.

    ```
   cat biz-card.json
    ```

    Cloud Shell 창을 스크롤하여 파일 내용을 살펴보세요. 이 JSON 파일은 명함에서 어떤 정보를 추출할지를 정의하는 분석기 스키마(schema)입니다. 즉, 우리가 만들 분석기의 '설계도'와 같습니다.

11. 분석기용 JSON 파일을 확인했다면, 다음 명령어를 입력하여 제공된 **create-analyzer.py** Python 코드 파일을 편집합니다.

    ```
   code create-analyzer.py
    ```

    코드 편집기에서 Python 코드 파일이 열립니다.

12. 코드를 검토해 보세요. 이 코드는 다음과 같은 작업을 수행하도록 구성되어 있습니다.
    -   **biz-card.json** 파일에서 분석기 스키마를 로드합니다.
    -   환경 설정 파일(`.env`)에서 엔드포인트, 키, 분석기 이름을 가져옵니다.
    -   **create_analyzer**라는 함수를 호출합니다. 현재 이 함수는 구현되어 있지 않습니다.

13. **create_analyzer** 함수 내에서 **`# Create a Content Understanding analyzer`** 주석을 찾은 다음, 그 아래에 다음 코드를 추가합니다. (올바른 들여쓰기를 유지하도록 주의하세요):

    ```python
   # Create a Content Understanding analyzer
   print (f"Creating {analyzer}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # initiate the analyzer creation operation
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"}

   url = f"{endpoint}/contentunderstanding/analyzers/{analyzer}?api-version={CU_VERSION}"

   # Delete the analyzer if it already exists
   response = requests.delete(url, headers=headers)
   print(response.status_code)
   time.sleep(1)

   # Now create it
   response = requests.put(url, headers=headers, data=(schema))
   print(response.status_code)

   # Get the response and extract the callback URL
   callback_url = response.headers["Operation-Location"]

   # Check the status of the operation
   time.sleep(1)
   result_response = requests.get(callback_url, headers=headers)

   # Keep polling until the operation is no longer running
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(callback_url, headers=headers)
        status = result_response.json().get("status")

   result = result_response.json().get("status")
   print(result)
   if result == "Succeeded":
        print(f"Analyzer '{analyzer}' created successfully.")
   else:
        print("Analyzer creation failed.")
        print(result_response.json())
    ```

14. 추가한 코드를 검토하며 각 부분이 어떤 역할을 하는지 이해해 봅시다.
    -   REST 요청에 필요한 헤더(headers)를 생성합니다. 여기에는 인증을 위한 구독 키가 포함됩니다.
    -   HTTP *DELETE* 요청을 보내 만약 동일한 이름의 분석기가 이미 존재한다면 삭제합니다. 이는 스크립트를 여러 번 실행해도 오류가 발생하지 않도록 합니다.
    -   HTTP *PUT* 요청을 보내 분석기 생성을 시작합니다.
    -   응답을 확인하여 *Operation-Location* 콜백(callback) URL을 추출합니다. 분석기 생성은 즉시 완료되지 않는 비동기 작업이므로, 이 URL을 통해 진행 상태를 확인할 수 있습니다.
    -   콜백 URL로 HTTP *GET* 요청을 반복적으로 보내 작업 상태가 더 이상 'Running'이 아닐 때까지 확인합니다(이를 '폴링(polling)'이라고 합니다).
    -   최종적으로 작업의 성공 또는 실패 여부를 사용자에게 알려줍니다.

    > **참고**: 코드에는 서비스의 요청 속도 제한(request rate limit)을 초과하지 않도록 의도적인 시간 지연(`time.sleep`)이 포함되어 있습니다.

15. **CTRL+S**를 눌러 코드 변경 사항을 저장합니다. 코드에 오류가 있을 경우 수정을 위해 코드 편집기 창은 열어두세요. 창 크기를 조절하여 명령줄 창을 명확하게 볼 수 있도록 합니다.
16. Cloud Shell 명령줄 창에서 다음 명령어를 입력하여 Python 코드를 실행합니다.

    ```
   python create-analyzer.py
    ```

17. 프로그램의 출력 결과를 확인합니다. 분석기가 성공적으로 생성되었다는 메시지가 나타나야 합니다.

## REST API를 사용하여 콘텐츠 분석하기

이제 분석기를 생성했으니, 클라이언트 애플리케이션에서 Content Understanding REST API를 통해 이 분석기를 사용하여 콘텐츠를 분석할 수 있습니다.

1.  Cloud Shell 명령줄에서 다음 명령어를 입력하여 제공된 **read-card.py** Python 코드 파일을 편집합니다.

    ```
   code read-card.py
    ```

    코드 편집기에서 Python 코드 파일이 열립니다.

2.  코드를 검토해 보세요. 이 코드는 다음과 같은 작업을 수행하도록 구성되어 있습니다.
    -   분석할 이미지 파일을 식별하며, 기본값은 **biz-card-1.png**입니다.
    -   프로젝트에서 Azure AI 서비스 리소스의 엔드포인트와 키를 가져옵니다. (현재 Cloud Shell 세션의 Azure 자격 증명을 사용하여 인증합니다.)
    -   **analyze_card**라는 함수를 호출합니다. 현재 이 함수는 구현되어 있지 않습니다.

3.  **analyze_card** 함수 내에서 **`# Use Content Understanding to analyze the image`** 주석을 찾은 다음, 그 아래에 다음 코드를 추가합니다. (올바른 들여쓰기를 유지하도록 주의하세요):

    ```python
   # Use Content Understanding to analyze the image
   print (f"Analyzing {image_file}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # Read the image data
   with open(image_file, "rb") as file:
        image_data = file.read()
    
   ## Use a POST request to submit the image data to the analyzer
   print("Submitting request...")
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/octet-stream"}
   url = f'{endpoint}/contentunderstanding/analyzers/{analyzer}:analyze?api-version={CU_VERSION}'
   response = requests.post(url, headers=headers, data=image_data)

   # Get the response and extract the ID assigned to the analysis operation
   print(response.status_code)
   response_json = response.json()
   id_value = response_json.get("id")

   # Use a GET request to check the status of the analysis operation
   print ('Getting results...')
   time.sleep(1)
   result_url = f'{endpoint}/contentunderstanding/analyzerResults/{id_value}?api-version={CU_VERSION}'
   result_response = requests.get(result_url, headers=headers)
   print(result_response.status_code)

   # Keep polling until the analysis is complete
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(result_url, headers=headers)
        status = result_response.json().get("status")

   # Process the analysis results
   if status == "Succeeded":
        print("Analysis succeeded:\n")
        result_json = result_response.json()
        output_file = "results.json"
        with open(output_file, "w") as json_file:
            json.dump(result_json, json_file, indent=4)
            print(f"Response saved in {output_file}\n")

        # Iterate through the fields and extract the names and type-specific values
        contents = result_json["result"]["contents"]
        for content in contents:
            if "fields" in content:
                fields = content["fields"]
                for field_name, field_data in fields.items():
                    if field_data['type'] == "string":
                        print(f"{field_name}: {field_data['valueString']}")
                    elif field_data['type'] == "number":
                        print(f"{field_name}: {field_data['valueNumber']}")
                    elif field_data['type'] == "integer":
                        print(f"{field_name}: {field_data['valueInteger']}")
                    elif field_data['type'] == "date":
                        print(f"{field_name}: {field_data['valueDate']}")
                    elif field_data['type'] == "time":
                        print(f"{field_name}: {field_data['valueTime']}")
                    elif field_data['type'] == "array":
                        print(f"{field_name}: {field_data['valueArray']}")
    ```

4.  추가한 코드를 다시 한번 검토해 봅시다.
    -   이미지 파일의 내용을 읽습니다.
    -   사용할 Content Understanding REST API의 버전을 설정합니다.
    -   HTTP *POST* 요청을 Content Understanding 엔드포인트로 보내 이미지를 분석하도록 지시합니다.
    -   작업 응답을 확인하여 분석 작업에 할당된 ID를 추출합니다.
    -   Content Understanding 엔드포인트로 HTTP *GET* 요청을 반복적으로 보내 작업 상태가 더 이상 'Running'이 아닐 때까지 확인합니다.
    -   작업이 성공하면, JSON 응답을 파일에 저장한 다음, JSON을 파싱하여 각 타입별 필드에 대해 추출된 값을 화면에 표시합니다.

    > **참고**: 우리가 만든 간단한 명함 스키마에서는 모든 필드가 문자열(string) 타입입니다. 하지만 이 코드는 더 복잡한 스키마에서 숫자, 날짜 등 다양한 타입의 값을 추출하기 위해 각 필드의 타입을 확인해야 할 필요성을 보여줍니다.

5.  **CTRL+S**를 눌러 코드 변경 사항을 저장합니다. 코드에 오류가 있을 경우 수정을 위해 코드 편집기 창은 열어두세요. 창 크기를 조절하여 명령줄 창을 명확하게 볼 수 있도록 합니다.
6.  Cloud Shell 명령줄 창에서 다음 명령어를 입력하여 Python 코드를 실행합니다.

    ```
   python read-card.py biz-card-1.png
    ```

7.  프로그램의 출력 결과를 확인합니다. 다음 명함 이미지의 필드 값들이 추출되어 표시되어야 합니다.

    ![A business card for Roberto Tamburello, an Adventure Works Cycles employee.](./media/biz-card-1.png)

8.  다음 명령어를 사용하여 다른 명함 이미지로 프로그램을 실행해 봅니다.

    ```
   python read-card.py biz-card-2.png
    ```

9.  결과를 확인합니다. 이번에는 아래 명함 이미지에 있는 값들이 반영되어야 합니다.

    ![A business card for Mary Duartes, an Contoso employee.](./media/biz-card-2.png)

10. Cloud Shell 명령줄 창에서 다음 명령어를 사용하여 서비스로부터 반환된 전체 JSON 응답을 확인해 봅니다.

    ```
   cat results.json
    ```

    창을 스크롤하여 전체 JSON 구조와 내용을 살펴보세요. 추출된 값뿐만 아니라 신뢰도 점수, 경계 상자 좌표 등 훨씬 더 풍부한 정보가 포함되어 있음을 알 수 있습니다.

## 정리

Content Understanding 서비스를 사용한 작업이 모두 끝났다면, 불필요한 Azure 비용이 발생하지 않도록 이 실습에서 생성한 리소스들을 삭제해야 합니다.

1.  Azure Portal에서 이 실습 중에 생성한 리소스들을 삭제합니다. (주로 리소스 그룹을 삭제하면 그 안의 모든 리소스가 함께 삭제되어 편리합니다.)
