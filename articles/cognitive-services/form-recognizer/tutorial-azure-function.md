---
title: Учебник. Использование Функции Azure для обработки сохраненных документов
titleSuffix: Azure Cognitive Services
description: В этом учебнике показано, как использовать функцию Azure для активации обработки документов, отправленных в контейнер хранилища BLOB-объектов Azure.
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: tutorial
ms.date: 03/19/2021
ms.author: lajanuar
ms.openlocfilehash: bf455d9401593b5c09fa295e492368a2a5bee240
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105048698"
---
# <a name="tutorial-use-an-azure-function-to-process-stored-documents"></a>Учебник. Использование Функции Azure для обработки сохраненных документов

Распознаватель документов можно использовать как часть автоматизированного конвейера обработки данных, созданного с помощью Функций Azure. В этом учебнике показано, как использовать функцию Azure для обработки документов, отправленных в контейнер хранилища BLOB-объектов Azure. Этот рабочий процесс извлекает данные таблицы из сохраненных документов с помощью службы макета Распознавателя документов и сохраняет данные таблицы в CSV-файле в Azure. Затем данные можно отобразить с помощью Microsoft Power BI (не рассматривается здесь).

> [!div class="mx-imgBorder"]
> ![Схема рабочего процесса службы Azure](./media/tutorial-azure-function/workflow-diagram.png)

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создание учетной записи хранения Azure
> * Создание проекта Функций Azure
> * Извлечение данных макета из отправленных форм.
> * Отправка данных макета в службу хранилища Azure.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="Создайте ресурс Распознавателя документов"  target="_blank">создание ресурса Распознавателя документов <span class="docon docon-navigate-external x-hidden-focus"></span></a> на портале Azure, чтобы получить ключ и конечную точку Распознавателя документов. После развертывания щелкните **Перейти к ресурсам**.
  * Для подключения приложения к API "Распознаватель документов" потребуется ключ и конечная точка созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
  * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.
* Локальный PDF-документ для анализа. Вы можете скачать этот [образец документа](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/sample-layout.pdf) для использования.
* Установленный [Python 3.8.x](https://www.python.org/downloads/).
* Установленный [Обозреватель службы хранилища Azure](https://azure.microsoft.com/features/storage-explorer/).
* Установленный набор [Azure Functions Core Tools](../../azure-functions/functions-run-local.md?tabs=windows%2ccsharp%2cbash#install-the-azure-functions-core-tools).
* Visual Studio Code со следующими установленными расширениями:
  * [Расширение "Функции Azure"](/azure/developer/python/tutorial-vs-code-serverless-python-01#visual-studio-code-python-and-the-azure-functions-extension).
  * [Расширение Python](https://code.visualstudio.com/docs/python/python-tutorial#_install-visual-studio-code-and-the-python-extension)

## <a name="create-an-azure-storage-account"></a>Создание учетной записи хранения Azure

[Создайте учетную запись хранения Azure](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM) на портале Azure. В поле "Тип учетной записи" выберите **StorageV2**.

На левой панели выберите вкладку **CORS** и удалите имеющуюся политику CORS при наличии.

После развертывания создайте два пустых контейнера хранилища BLOB-объектов с именем **test** и **output**.

## <a name="create-an-azure-functions-project"></a>Создание проекта Функций Azure

Откройте Visual Studio Code. Если расширение "Функции Azure" установлено, в левой области навигации отобразится логотип Azure. Выберите ее. Создайте новый проект, а при появлении запроса создайте локальную папку **coa_new** для проекта.

![Кнопка функции создания VSCode](./media/tutorial-azure-function/vs-code-create-function.png)


Вам будет предложено настроить ряд параметров:
* В поле **Выберите язык** выберите Python.
* В поле **Выберите шаблон** выберите триггер для Хранилища BLOB-объектов Azure. Затем присвойте имя триггеру по умолчанию.
* В поле **Выберите параметр** выберите создание новых параметров локального приложения.
* Выберите подписку Azure, в которой создана учетная запись хранения. Затем необходимо ввести имя контейнера хранилища (в нашем случае это `test/{name}`).
* Откройте проект в текущем окне. 

![Пример запроса на создание VSCode](./media/tutorial-azure-function/vs-code-prompt.png)

После выполнения этих действий в VSCode будет добавлен новый проект Функции Azure с помощью скрипта Python *\_\_init\_\_.py*. Этот скрипт активируется при отправке файла в контейнер хранилища **test**, но без выполнения каких-либо действий.

## <a name="test-the-function"></a>Проверка функции

Нажмите клавишу F5, чтобы запустить базовую функцию. VSCode предложит выбрать учетную запись хранения для взаимодействия. Выберите созданную учетную запись хранения и продолжайте работу.

Откройте Обозреватель службы хранилища Azure и отправьте образец PDF-документа в контейнер **Test**. Затем проверьте терминал VSCode. Скрипт должен зарегистрировать, что он был активирован при отправке PDF-файла.

![Проверка терминала VSCode](./media/tutorial-azure-function/vs-code-terminal-test.png)


Прежде чем продолжить, завершите выполнение скрипта.

## <a name="add-form-processing-code"></a>Добавление кода обработки формы

Далее вы добавите свой собственный код в скрипт Python, чтобы вызвать службу "Распознаватель документов" и проанализировать отправленные документы с помощью [API макета](concept-layout.md) Распознавателя документов.

В VSCode перейдите к файлу функции *requirements.txt*. В нем указаны зависимости для скрипта. Добавьте следующие пакеты Python в файл:

```
cryptography
azure-functions
azure-storage-blob
azure-identity
requests
pandas
numpy
```

Затем откройте скрипт *\_\_init\_\_.py*. Добавьте следующие операторы `import` :

```Python
import logging
from azure.storage.blob import BlobServiceClient
import azure.functions as func
import json
import time
from requests import get, post
import os
from collections import OrderedDict
import numpy as np
import pandas as pd
```

Созданную функцию `main` можно оставить без изменений. Вы добавите пользовательский код в эту функцию.

```python
# This part is automatically generated
def main(myblob: func.InputStream):
    logging.info(f"Python blob trigger function processed blob \n"
    f"Name: {myblob.name}\n"
    f"Blob Size: {myblob.length} bytes")
```

В следующем блоке кода вызывается API [анализа макета](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/AnalyzeLayoutAsync) Распознавателя документов в отправленном документе. Заполните значения конечной точки и ключа. 


# <a name="version-20"></a>[Версия 2.0](#tab/2-0)

```Python
# This is the call to the Form Recognizer endpoint
    endpoint = r"Your Form Recognizer Endpoint"
    apim_key = "Your Form Recognizer Key"
    post_url = endpoint + "/formrecognizer/v2.0/Layout/analyze"
    source = myblob.read()

    headers = {
    # Request headers
    'Content-Type': 'application/pdf',
    'Ocp-Apim-Subscription-Key': apim_key,
        }

    text1=os.path.basename(myblob.name)
```

# <a name="version-21-preview"></a>[Предварительная версия 2.1](#tab/2-1)

```Python
# This is the call to the Form Recognizer endpoint
    endpoint = r"Your Form Recognizer Endpoint"
    apim_key = "Your Form Recognizer Key"
    post_url = endpoint + "/formrecognizer/v2.1-preview.3/Layout/analyze"
    source = myblob.read()

    headers = {
    # Request headers
    'Content-Type': 'application/pdf',
    'Ocp-Apim-Subscription-Key': apim_key,
        }

    text1=os.path.basename(myblob.name)
```
---

> [!IMPORTANT]
> Перейдите на портал Azure. Если ресурс Распознавателя документов, созданный в соответствии с указаниями в разделе **Предварительные требования**, успешно развернут, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключ и конечная точка располагаются на странице **ключа и конечной точки** ресурса в разделе **управления ресурсами**. 
>
> Не забудьте удалить ключ из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Дополнительные сведения см. в статье о [безопасности в Cognitive Services](../cognitive-services-security.md).

Далее добавьте код для запроса к службе и получения возвращаемых данных. 


```Python
resp = requests.post(url = post_url, data = source, headers = headers)
    if resp.status_code != 202:
        print("POST analyze failed:\n%s" % resp.text)
        quit()
    print("POST analyze succeeded:\n%s" % resp.headers)
    get_url = resp.headers["operation-location"]

    wait_sec = 25
    
    time.sleep(wait_sec)
    # The layout API is async therefore the wait statement
    
    resp =requests.get(url = get_url, headers = {"Ocp-Apim-Subscription-Key": apim_key})
    
    resp_json = json.loads(resp.text)
    
    
    status = resp_json["status"]
    
    
    if status == "succeeded":
        print("Layout Analysis succeeded:\n%s")
        results=resp_json
    else:
        print("GET Layout results failed:\n%s")
        quit()

    results=resp_json
```

Затем добавьте следующий код для подключения к контейнеру **output** службы хранилища Azure. Введите собственные значения для имени и ключа учетной записи хранения. Ключ можно получить на вкладке **Ключи доступа** ресурса хранилища на портале Azure.

```Python
# This is the connection to the blob storage, with the Azure Python SDK
    blob_service_client = BlobServiceClient.from_connection_string("DefaultEndpointsProtocol=https;AccountName="Storage Account Name";AccountKey="storage account key";EndpointSuffix=core.windows.net")
    container_client=blob_service_client.get_container_client("output")
```

Следующий код анализирует возвращенный ответ Распознавателя документов, конструирует CSV-файл и отправляет его в контейнер **output**. 


> [!IMPORTANT]
> Вам, скорее всего, потребуется изменить этот код в соответствии со структурой собственных документов форм.

```Python
# The code below is how I extract the json format into tabular data 
    # Please note that you need to adjust the code below to your form structure
    # It probably won't work out-of-box for your specific form
    pages = results["analyzeResult"]["pageResults"]

    def make_page(p):
        res=[]
        res_table=[]
        y=0
        page = pages[p]
        for tab in page["tables"]:
            for cell in tab["cells"]:
                res.append(cell)
                res_table.append(y)
            y=y+1
    
        res_table=pd.DataFrame(res_table)
        res=pd.DataFrame(res)
        res["table_num"]=res_table[0]
        h=res.drop(columns=["boundingBox","elements"])
        h.loc[:,"rownum"]=range(0,len(h))
        num_table=max(h["table_num"])
        return h, num_table, p

    h, num_table, p= make_page(0)   

    for k in range(num_table+1):
        new_table=h[h.table_num==k]
        new_table.loc[:,"rownum"]=range(0,len(new_table))
        row_table=pages[p]["tables"][k]["rows"]
        col_table=pages[p]["tables"][k]["columns"]
        b=np.zeros((row_table,col_table))
        b=pd.DataFrame(b)
        s=0
        for i,j in zip(new_table["rowIndex"],new_table["columnIndex"]):
            b.loc[i,j]=new_table.loc[new_table.loc[s,"rownum"],"text"]
            s=s+1

```

Наконец, последний блок кода отправляет извлеченную таблицу и текстовые данные в элемент хранилища BLOB-объектов.

```Python
    # Here is the upload to the blob storage
    tab1_csv=b.to_csv(header=False,index=False,mode='w')
    name1=(os.path.splitext(text1)[0]) +'.csv'
    container_client.upload_blob(name=name1,data=tab1_csv)
```

## <a name="run-the-function"></a>Выполнение функции

Нажмите клавишу F5, чтобы запустить функцию повторно. Используйте Обозреватель службы хранилища Azure для отправки образца формы PDF в контейнер хранилища **Test**. Это действие должно активировать скрипт, и в контейнере **output** должен отобразиться полученный CSV-файл (в виде таблицы).

Этот контейнер можно подключить к Power BI для создания полнофункциональных визуализаций содержащихся в нем данных.

## <a name="next-steps"></a>Дальнейшие действия

Из этого учебника вы узнали, как использовать Функцию Azure, написанную на языке Python, для автоматической обработки отправленных PDF-документов и вывода их содержимого в более понятном формате. Далее вы узнаете, как использовать Power BI для отображения данных.

> [!div class="nextstepaction"]
> [Microsoft Power BI](https://powerbi.microsoft.com/integrations/azure-table-storage/)

* [Что такое Распознаватель документов?](overview.md)
* Узнайте больше об [API макета](concept-layout.md)