## 전제 조건  
다음이 설치되어 있는지 확인하십시오:

Python 3.x  
필요한 Python 라이브러리:  
```bash
pip install azure-search-documents azure-core azure-identity openai python-dotenv
```
환경 설정  
.env 파일 구성  
.env 파일을 생성하고 다음 내용을 추가하십시오:  

```plaintext
AZURE_OPENAI_ENDPOINT=https://your-openai-endpoint.azure.com/
AZURE_OPENAI_API_KEY=your_openai_api_key
AZURE_OAI_MODEL=gpt-35-turbo-16k
AZURE_SEARCH_ENDPOINT=https://your-search-endpoint.search.windows.net
AZURE_SEARCH_API_KEY=your_search_api_key
AZURE_SEARCH_INDEX=your_search_index
```
위의 자리 표시자 값을 실제 Azure 리소스 정보로 교체하십시오.  

### 메인 함수
설명  
메인 함수는 환경을 초기화하고, Azure OpenAI 및 Azure Cognitive Search 클라이언트를 설정하며, 다양한 작업을 실행하기 위한 명령 인터페이스를 제공합니다.  

```python
import os
from dotenv import load_dotenv
import utils
from openai import AzureOpenAI
from azure.search.documents import SearchClient
from azure.core.credentials import AzureKeyCredential

def main():
    try:
        load_dotenv()
        utils.initLogFile()
        azure_oai_endpoint = os.getenv("AZURE_OPENAI_ENDPOINT")
        azure_oai_key = os.getenv("AZURE_OPENAI_API_KEY")
        azure_oai_model = os.getenv("AZURE_OAI_MODEL")
        
        # Define Azure OpenAI client
        aiClient = AzureOpenAI(
            api_key=azure_oai_key,
            api_version="2023-12-01-preview",
            azure_endpoint=azure_oai_endpoint
        )
        
        # Define Azure Search client
        search_service_endpoint = os.getenv("AZURE_SEARCH_ENDPOINT")
        search_api_key = os.getenv("AZURE_SEARCH_API_KEY")
        search_index_name = os.getenv("AZURE_SEARCH_INDEX")
        search_client = SearchClient(
            endpoint=search_service_endpoint,
            index_name=search_index_name,
            credential=AzureKeyCredential(search_api_key)
        )

        function_map = {
            "1": function1,
            "2": function2,
            "3": function3,
            "4": function4
        }

        while True:
            print('1: Validate PoC\n' +
                  '2: Company chatbot\n' +
                  '3: Developer tasks\n' +
                  '4: Use company data\n' +
                  '\'quit\' to exit the program\n')
            command = input('Enter a number:')
            if command.strip() in function_map:
                function_map[command](aiClient, azure_oai_model, search_client)
            elif command.strip().lower() == 'quit':
                print('Exiting program...')
                break
            else:
                print("Invalid input. Please enter number 1, 2, 3, or 4.")
    except Exception as ex:
        print(ex)

if __name__ == '__main__':
    main()
```
## 작업 1: PoC 검증
설명  
샘플 텍스트 프롬프트를 Azure OpenAI 모델에 보내고 응답을 받는 함수입니다.  

코드  
```python
def function1(aiClient, aiModel):
    inputText = utils.getPromptInput("Task 1: Validate PoC", "sample-text.txt")

    # Build messages to send to Azure OpenAI model
    messages = [
        {"role": "user", "content": inputText}
    ]

    # Define argument list
    apiParams = {
        "model": aiModel,
        "messages": messages,
        "max_tokens": 100,
        "temperature": 0.5
    }

    utils.writeLog("API Parameters:\n", apiParams)

    # Call chat completion connection
    response = aiClient.chat.completions.create(**apiParams)

    utils.writeLog("Response:\n", str(response))
    print("Response: " + response.choices[0].message.content + "\n")
    return response

```
## 작업 2: 회사 챗봇
설명  
질문에 대해 캐주얼한 톤으로 답변하고 각 응답에 특정 종료 메시지를 추가하는 회사 챗봇을 만드는 함수입니다.  

코드  
```python
def function2(aiClient, aiModel):
    inputText = utils.getPromptInput("Task 2: Company chatbot", "sample-text.txt")

    # Build messages to send to Azure OpenAI model
    messages = [
        {"role": "system", "content": "You are a helpful assistant who responds in a casual tone."},
        {"role": "user", "content": "Where can I find the company phone number?"},
        {"role": "assistant", "content": "You can find it on the footer of every page on our website. Hope that helps! Thanks for using Contoso, Ltd."},
        {"role": "user", "content": inputText}
    ]

    # Define argument list
    apiParams = {
        "model": aiModel,
        "messages": messages,
        "max_tokens": 1000,
        "temperature": 0.5
    }

    utils.writeLog("API Parameters:\n", apiParams)

    # Call chat completion connection
    response = aiClient.chat.completions.create(**apiParams)

    response_text = response.choices[0].message.content.strip()
    response_text += " Hope that helps! Thanks for using Contoso, Ltd."
    
    utils.writeLog("Response:\n", response_text)
    print("Response: " + response_text + "\n")
    return response_text
```

## 작업 3: 개발자 작업
설명  
레거시 코드에 주석을 추가하고 문서를 생성하며, fibonacci.py의 함수에 대한 단위 테스트를 생성하는 함수입니다.  

코드  
```python
def function3(aiClient, aiModel):
    # Get the task selection from the user
    print('Select task:\n1: Add comments and generate documentation for legacyCode.py\n2: Generate five unit tests for fibonacci.py')
    task = input('Enter the task number (1 or 2): ').strip()

    if task == '1':
        file_path = "C:\\files\\legacyCode.py"
        prompt_task = "Please add comments to the following legacy code and generate documentation for it."
    elif task == '2':
        file_path = "C:\\files\\fibonacci.py"
        prompt_task = "Please generate five unit tests for the following function in fibonacci.py."
    else:
        print("Invalid input. Please enter 1 or 2.")
        return

    with open(file_path, 'r') as file:
        file_content = file.read()

    inputText = f"{prompt_task}\n\n{file_content}"

    # Build messages to send to Azure OpenAI model
    messages = [
        {"role": "user", "content": inputText}
    ]

    # Define argument list
    apiParams = {
        "model": aiModel,
        "messages": messages,
        "max_tokens": 1000,
        "temperature": 0.5
    }

    utils.writeLog("API Parameters:\n", apiParams)

    # Call chat completion connection
    response = aiClient.chat.completions.create(**apiParams)

    response_text = response.choices[0].message.content.strip()
    
    utils.writeLog("Response:\n", response_text)
    print("Response: " + response_text + "\n")
    
    # Write the response to a new file
    output_file_path = file_path.replace(".py", "_output.py") if task == '1' else "C:\\files\\fibonacci_tests.py"
    with open(output_file_path, 'w') as output_file:
        output_file.write(response_text)
    
    print(f"Output written to: {output_file_path}")

    return response_text
```

## 작업 4: 회사 데이터 사용
설명
Azure Cognitive Search에 연결하여 색인된 회사 데이터를 키워드 검색하고, 검색 결과를 사용하여 고객 질문에 답변하는 함수입니다.

코드(To be updated)
