---
title: Пример настраиваемого навыка (Python)
titleSuffix: Azure Cognitive Search
description: Для разработчиков на Python Узнайте о средствах и методах создания пользовательского навыка с помощью функций Azure и Visual Studio. Пользовательские навыки содержат определяемые пользователем модели или логику, которые можно добавить в конвейер индексирования на основе искусственного интеллекта в Azure Когнитивный поиск.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 01/15/2020
ms.custom: devx-track-python
ms.openlocfilehash: 40e20ad4bab0275b44cd868521c7dc70dec52567
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88936135"
---
# <a name="example-create-a-custom-skill-using-python"></a>Пример. Создание пользовательского навыка с помощью Python

В этом примере набора навыков Azure Когнитивный поиск вы узнаете, как создать пользовательский навык веб-API с помощью Python и Visual Studio Code. В примере используется [функция Azure](https://azure.microsoft.com/services/functions/) , которая реализует [Пользовательский интерфейс квалификации](cognitive-search-custom-skill-interface.md).

Пользовательский навык прост в проектировании (он объединяет две строки), чтобы вы могли сосредоточиться на средствах и технологиях, используемых для разработки пользовательских навыков в Python. После того как вы пройдете простой навык, вы сможете создать более сложные сценарии.

## <a name="prerequisites"></a>Предварительные требования

+ Изучите [Пользовательский интерфейс навыка](cognitive-search-custom-skill-interface.md) , чтобы получить представление о интерфейсе ввода-вывода, который должен быть реализован в пользовательском навыке.

+ Настройте среду. Мы простроили [это руководство](/azure/python/tutorial-vs-code-serverless-python-01) , чтобы настроить бессерверную функцию Azure с помощью расширений Visual Studio Code и Python. В этом учебнике описывается установка следующих средств и компонентов: 

  + [Python 3,75](https://www.python.org/downloads/release/python-375/)
  + [Visual Studio Code](https://code.visualstudio.com/)
  + [Расширение Python для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  + [Azure Functions Core Tools](../azure-functions/functions-run-local.md#v2)
  + [Расширение "Функции Azure" для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)

## <a name="create-an-azure-function"></a>Создание функции Azure

В этом примере используется функция Azure для демонстрации концепции размещения веб-API, но возможны и другие подходы. При условии соблюдения [требований к интерфейсу для когнитивного навыка](cognitive-search-custom-skill-interface.md) выбранный вами подход не имеет значения. Однако Функции Azure упрощают создание пользовательских навыков.

### <a name="create-a-function-app"></a>Создание приложения-функции

С помощью шаблона проекта Функции Azure в Visual Studio Code можно создать проект и затем опубликовать его в виде приложения-функции в Azure. Приложение-функция позволяет группировать функции в логические единицы для развертывания и совместного использования ресурсов, а также управления ими.

1. В Visual Studio Code нажмите клавишу F1, чтобы открыть палитру команд. В палитре команд найдите и щелкните `Azure Functions: Create new project...`.

1. Выберите расположение для рабочей области проекта и нажмите кнопку **Выбрать**.

    > [!NOTE]
    > Рассматриваемые в этой статье шаги выполняются вне рабочей области. По этой причине не следует выбирать папку проекта, которая является частью рабочей области.

1. Выберите язык для проекта приложения-функции. Для работы с этим руководством выберите **Python**.
1. Выберите версию Python (версия 3.7.5 поддерживается функциями Azure).
1. Выберите шаблон для первой функции проекта. Выберите **триггер HTTP** , чтобы создать функцию, активируемую HTTP, в новом приложении-функции.
1. Укажите имя функции. В этом случае давайте будем использовать **конкатенации** 
1. Выберите **функция** в качестве уровня авторизации. Это означает, что мы предоставляем [ключ функции](../azure-functions/functions-bindings-http-webhook-trigger.md#authorization-keys) для вызова КОНЕЧНОЙ точки HTTP функции. 
1. Выберите способ открытия проекта. Для этого шага выберите **Добавить в рабочую область** , чтобы создать приложение функции в текущей рабочей области.

Редактор Visual Studio Code создаст проект приложения-функции в новой рабочей области. Проект будет содержать файлы конфигурации [host.json](../azure-functions/functions-host-json.md) и [local.settings.json](../azure-functions/functions-run-local.md#local-settings-file), а также файлы проекта для указанного языка. 

Новая функция, активируемая HTTP, также создается в папке **конкатенации** проекта приложения-функции. Внутри него будет файл с именем « \_ \_ init__. Корректировка» с этим содержимым:

```py
import logging

import azure.functions as func


def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            name = req_body.get('name')

    if name:
        return func.HttpResponse(f"Hello {name}!")
    else:
        return func.HttpResponse(
             "Please pass a name on the query string or in the request body",
             status_code=400
        )

```

Теперь давайте изменим код для выполнения [пользовательского интерфейса навыков](cognitive-search-custom-skill-interface.md). Измените код, используя следующее содержимое:

```py
import logging
import azure.functions as func
import json

def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    try:
        body = json.dumps(req.get_json())
    except ValueError:
        return func.HttpResponse(
             "Invalid body",
             status_code=400
        )
    
    if body:
        result = compose_response(body)
        return func.HttpResponse(result, mimetype="application/json")
    else:
        return func.HttpResponse(
             "Invalid body",
             status_code=400
        )


def compose_response(json_data):
    values = json.loads(json_data)['values']
    
    # Prepare the Output before the loop
    results = {}
    results["values"] = []
    
    for value in values:
        output_record = transform_value(value)
        if output_record != None:
            results["values"].append(output_record)
    return json.dumps(results, ensure_ascii=False)

## Perform an operation on a record
def transform_value(value):
    try:
        recordId = value['recordId']
    except AssertionError  as error:
        return None

    # Validate the inputs
    try:         
        assert ('data' in value), "'data' field is required."
        data = value['data']        
        assert ('text1' in data), "'text1' field is required in 'data' object."
        assert ('text2' in data), "'text2' field is required in 'data' object."
    except AssertionError  as error:
        return (
            {
            "recordId": recordId,
            "errors": [ { "message": "Error:" + error.args[0] }   ]       
            })

    try:                
        concatenated_string = value['data']['text1'] + " " + value['data']['text2']  
        # Here you could do something more interesting with the inputs

    except:
        return (
            {
            "recordId": recordId,
            "errors": [ { "message": "Could not complete operation for record." }   ]       
            })

    return ({
            "recordId": recordId,
            "data": {
                "text": concatenated_string
                    }
            })
```

Метод **transform_value** выполняет операцию с одной записью. Вы можете изменить метод в соответствии с конкретными потребностями. Не забывайте выполнять необходимую проверку входных данных и возвращать любые ошибки и предупреждения, созданные, если операция не может быть выполнена для записи.

### <a name="debug-your-code-locally"></a>Отладка кода в локальной среде

Visual Studio Code упрощает отладку кода. Нажмите клавишу F5 или перейдите в меню **Отладка** и выберите **начать отладку**.

Можно задать любые точки останова в коде, нажав "F9" в интересующей строке.

После начала отладки функция будет выполняться локально. Чтобы отправить запрос в localhost, можно использовать такой инструмент, как POST или Fiddler. Обратите внимание на расположение локальной конечной точки в окне терминала. 

## <a name="publish-your-function"></a>Публикация функции

Когда поведение функции будет удовлетворено, его можно опубликовать.

1. В Visual Studio Code нажмите клавишу F1, чтобы открыть палитру команд. В палитре команд найдите и выберите **развернуть в приложение-функция...**. 

1. Выберите подписку Azure, в которой вы хотите развернуть приложение.

1. Выберите **+ создать новый приложение-функция в Azure** .

1. Введите глобально уникальное имя для приложения-функции.

1. Выберите версию Python (для этой функции работает Python 3.7. x).

1. Выберите расположение для нового ресурса (например, "Западная часть США 2").

На этом этапе в подписке Azure будут созданы необходимые ресурсы для размещения новой функции Azure в Azure. Дождитесь завершения развертывания. В окне вывода отобразится состояние процесса развертывания.

1. В [портал Azure](https://portal.azure.com)перейдите ко **всем ресурсам** и найдите функцию, опубликованную по имени. Если вы назвали его с помощью функции **объединения**, выберите ресурс.

1. Нажмите кнопку **</> получить URL-адрес функции** . Это позволит вам скопировать URL-адрес для вызова функции.

## <a name="test-the-function-in-azure"></a>Тестирование функции в Azure

Теперь, когда у вас есть ключ узла по умолчанию, протестируйте функцию следующим образом:

```http
POST [Function URL you copied above]
```

### <a name="request-body"></a>Текст запроса
```json
{
    "values": [
        {
            "recordId": "e1",
            "data":
            {
                "text1":  "Hello",
                "text2":  "World"
            }
        },
        {
            "recordId": "e2",
            "data": "This is an invalid input"
        }
    ]
}
```

Этот пример должен иметь тот же результат, который вы видели ранее при выполнении функции в локальной среде.

## <a name="connect-to-your-pipeline"></a>Подключение к конвейеру

Теперь, когда у вас есть новый пользовательский навык, можно добавить его в ваш набор навыков. В приведенном ниже примере показано, как вызвать навык для сцепления заголовка и автора документа в одно поле, которое мы вызываем merged_title_author. Замените `[your-function-url-here]` URL-адресом новой функции Azure.

```json
{
    "skills": [
      "[... your existing skills remain here]",  
      {
        "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
        "description": "Our new search custom skill",
        "uri": "https://[your-function-url-here]",        
          "context": "/document/merged_content/organizations/*",
          "inputs": [
            {
              "name": "text1",
              "source": "/document/metadata_title"
            },
            {
              "name": "text2",
              "source": "/document/metadata_author"
            },
          ],
          "outputs": [
            {
              "name": "text",
              "targetName": "merged_title_author"
            }
          ]
      }
  ]
}
```

## <a name="next-steps"></a>Дальнейшие действия
Поздравляем! Вы создали свой первый пользовательский навык. Теперь вы можете использовать ту же схему для добавления пользовательских функций. Чтобы получить дополнительные сведения, перейдите по следующим ссылкам.

+ [Power Skills: репозиторий пользовательских навыков](https://github.com/Azure-Samples/azure-search-power-skills)
+ [Добавление пользовательского навыка в конвейер обогащения искусственного интеллекта](cognitive-search-custom-skill-interface.md)
+ [Определение набора навыков](cognitive-search-defining-skillset.md)
+ [Создание набора навыков (REST)](/rest/api/searchservice/create-skillset)
+ [Сопоставление обогащенных полей](cognitive-search-output-field-mapping.md)