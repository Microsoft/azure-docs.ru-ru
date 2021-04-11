---
title: Краткое руководство. REST API Распознавателя документов
description: Узнайте, как использовать REST API Распознавателя документов для создания приложения для обработки форм, которое извлекает из пользовательских документов пары "ключ-значение" и табличные данные.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 03/15/2021
ms.author: lajanuar
ms.openlocfilehash: e0769f77bffaab0e04b492b19a5ea131cec64f06
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105105356"
---
<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD024 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD034 -->
> [!NOTE]
> В этом руководстве для выполнения вызовов REST API используется cURL. Также [на GitHub доступен пример кода](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/python/FormRecognizer/rest), демонстрирующий, как вызывать интерфейсы REST API с помощью Python.

## <a name="prerequisites"></a>Предварительные требования

* Установленная программа [cURL](https://curl.haxx.se/windows/).
* [PowerShell версии 6.0 и выше](/powershell/scripting/install/installing-powershell-core-on-windows) или аналогичное приложение командной строки.
* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Набор данных для обучения в хранилище BLOB-объектов Azure. Советы и варианты для объединения данных для обучения см. в статье о [создании обучающего набора данных для пользовательской модели](../../build-training-data-set.md). При работе с этим кратким руководством можно использовать файлы в папке **Train** из [примера набора данных](https://go.microsoft.com/fwlink/?linkid=2090451) (скачайте и разархивируйте файл *sample_data.zip*).
* Получив подписку Azure, перейдите к <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="Создание ресурса Распознавателя документов"  target="_blank">созданию ресурса Распознавателя документов </a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
  * Для подключения приложения к API Распознавателя документов потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
  * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.
* URL-адрес изображения квитанции. Вы можете использовать [пример изображения](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-REST-api-samples/master/curl/form-recognizer/contoso-allinone.jpg) при изучении этого краткого руководства.
* URL-адрес изображения визитной карточки. Вы можете использовать [пример изображения](https://raw.githubusercontent.com/Azure/azure-sdk-for-python/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms/business_cards/business-card-english.jpg) при изучении этого краткого руководства.
* URL-адрес изображения счета-фактуры. Для работы с этим кратким руководством вы можете использовать [пример документа](https://raw.githubusercontent.com/Azure/azure-sdk-for-python/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms/forms/Invoice_1.pdf).

## <a name="analyze-layout"></a>Анализ макета

С помощью Распознавателя документов можно анализировать и извлекать таблицы, метки выделения, текст и структуры в документах без необходимости обучать модель. Дополнительные сведения об извлечении макетов см. в статье [Служба макета Распознавателя документов](../../concept-layout.md). Перед выполнением команды внесите следующие изменения:

1. Замените `{Endpoint}` конечной точкой, полученной из подписки Распознавателя документов.
1. Замените `{subscription key}` ключом подписки, скопированным на предыдущем шаге.
1. Замените `\"{your-document-url}` одним из следующих примеров URL-адресов.

### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/v2-1)

```bash
curl -v -i POST "https://{Endpoint}/formrecognizer/v2.1-preview.3/layout/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{'source': '{your-document-url}'}"
```

### <a name="v20"></a>[Версия 2.0](#tab/v2-0)

```bash
curl -v -i POST "https://{Endpoint}/formrecognizer/v2.0/layout/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{'source': '{your-document-url}'}"
```

---

Вы получите ответ `202 (Success)`, содержащий заголовок **Operation-Location**. Значение этого заголовка содержит идентификатор операции, который можно использовать для запроса состояния асинхронной операции и получения результатов. В следующем примере строка после `analyzeResults/` является идентификатором операции.

```console
https://cognitiveservice/formrecognizer/v2/layout/analyzeResults/54f0b076-4e38-43e5-81bd-b85b8835fdfb
```

### <a name="get-layout-results"></a>Получение результатов макета

После вызова API **[анализа макета](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/AnalyzeLayoutAsync)** вызовите **[API получения результатов анализа макета](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/GetAnalyzeLayoutResult)** , чтобы получить состояние операции и извлеченных данных. Перед выполнением команды внесите следующие изменения:

1. Замените `{Endpoint}` конечной точкой, полученной из подписки Распознавателя документов.
1. Замените `{subscription key}` ключом подписки, скопированным на предыдущем шаге.
1. Замените `{resultId}` идентификатором операции из предыдущего шага.
<!-- markdownlint-disable MD024 -->

### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/v2-1)

```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.1-preview.3/layout/analyzeResults/{resultId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="v20"></a>[Версия 2.0](#tab/v2-0)

```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.0/layout/analyzeResults/{resultId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

---

### <a name="examine-the-results"></a>Просмотр результатов

Вы получите ответ `200 (success)` со следующими выходными данными JSON.

Ознакомьтесь со следующим изображением накладной и его соответствующими выходными данными в формате JSON.
* Узел `"readResults"` содержит каждую строку текста с соответствующим расположением ограничивающего прямоугольника на странице. 
* Узел `"selectionMarks"` (версии 2.1) отображает каждую отметку выделения (флажок или переключатель) и состояние "выбрано" или "не выбрано". 
* В разделе `"pageResults"` содержатся извлеченные таблицы. Для каждой таблицы извлекаются текст, строка и индекс столбца, объединение строки и столбца, ограничивающий прямоугольник и т. д.

:::image type="content" source="../../media/contoso-invoice.png" alt-text="Пример документа с таблицей":::

### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/v2-1)

Эти выходные данные сокращены для простоты. См. [полный пример выходных данных на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/sample-layout-output.json).

```json
{
    "status": "succeeded",
    "createdDateTime": "2020-08-20T20:40:50Z",
    "lastUpdatedDateTime": "2020-08-20T20:40:55Z",
    "analyzeResult": {
        "version": "2.1.0",
        "readResults": [
            {
                "page": 1,
                "angle": 0,
                "width": 8.5,
                "height": 11,
                "unit": "inch",
                "lines": [
                    {
                        "boundingBox": [
                            0.5826,
                            0.4411,
                            2.3387,
                            0.4411,
                            2.3387,
                            0.7969,
                            0.5826,
                            0.7969
                        ],
                        "text": "Contoso, Ltd.",
                        "words": [
                            {
                                "boundingBox": [
                                    0.5826,
                                    0.4411,
                                    1.744,
                                    0.4411,
                                    1.744,
                                    0.7969,
                                    0.5826,
                                    0.7969
                                ],
                                "text": "Contoso,",
                                "confidence": 1
                            },
                            {
                                "boundingBox": [
                                    1.8448,
                                    0.4446,
                                    2.3387,
                                    0.4446,
                                    2.3387,
                                    0.7631,
                                    1.8448,
                                    0.7631
                                ],
                                "text": "Ltd.",
                                "confidence": 1
                            }
                        ]
                    },
                    ...
                        ]
                    }
                ],
                "selectionMarks": [
                    {
                        "boundingBox": [
                            3.9737,
                            3.7475,
                            4.1693,
                            3.7475,
                            4.1693,
                            3.9428,
                            3.9737,
                            3.9428
                        ],
                        "confidence": 0.989,
                        "state": "selected"
                    },
                    ...
                ]
            }
        ],
        "pageResults": [
            {
                "page": 1,
                "tables": [
                    {
                        "rows": 5,
                        "columns": 5,
                        "cells": [
                            {
                                "rowIndex": 0,
                                "columnIndex": 0,
                                "text": "Training Date",
                                "boundingBox": [
                                    0.5133,
                                    4.2167,
                                    1.7567,
                                    4.2167,
                                    1.7567,
                                    4.4492,
                                    0.5133,
                                    4.4492
                                ],
                                "elements": [
                                    "#/readResults/0/lines/12/words/0",
                                    "#/readResults/0/lines/12/words/1"
                                ]
                            },
                            ...
                        ]
                    },
                    ...
                ]
            }
        ]
    }
}
```

### <a name="v20"></a>[Версия 2.0](#tab/v2-0)

Эти выходные данные сокращены для простоты. См. [полный пример выходных данных на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/sample-layout-output.json).

```json
{
    "status": "succeeded",
    "createdDateTime": "2020-08-20T20:36:52Z",
    "lastUpdatedDateTime": "2020-08-20T20:36:58Z",
    "analyzeResult": {
        "version": "2.0.0",
        "readResults": [
            {
                "page": 1,
                "language": "en",
                "angle": 0,
                "width": 8.5,
                "height": 11,
                "unit": "inch",
                "lines": [
                    {
                        "boundingBox": [
                            0.5826,
                            0.4411,
                            2.3387,
                            0.4411,
                            2.3387,
                            0.7969,
                            0.5826,
                            0.7969
                        ],
                        "text": "Contoso, Ltd.",
                        "words": [
                            {
                                "boundingBox": [
                                    0.5826,
                                    0.4411,
                                    1.744,
                                    0.4411,
                                    1.744,
                                    0.7969,
                                    0.5826,
                                    0.7969
                                ],
                                "text": "Contoso,",
                                "confidence": 1
                            },
                            {
                                "boundingBox": [
                                    1.8448,
                                    0.4446,
                                    2.3387,
                                    0.4446,
                                    2.3387,
                                    0.7631,
                                    1.8448,
                                    0.7631
                                ],
                                "text": "Ltd.",
                                "confidence": 1
                            }
                        ]
                    },
                    ...
                ]
            }
        ],
        "pageResults": [
            {
                "page": 1,
                "tables": [
                    {
                        "rows": 5,
                        "columns": 5,
                        "cells": [
                            {
                                "rowIndex": 0,
                                "columnIndex": 0,
                                "text": "Training Date",
                                "boundingBox": [
                                    0.5133,
                                    4.2167,
                                    1.7567,
                                    4.2167,
                                    1.7567,
                                    4.4492,
                                    0.5133,
                                    4.4492
                                ],
                                "elements": [
                                    "#/readResults/0/lines/14/words/0",
                                    "#/readResults/0/lines/14/words/1"
                                ]
                            },
                            ...
                        ]
                    },
                    ...
                ]
            }
        ]
    }
}
```

---

## <a name="analyze-invoices"></a>Анализ счетов

### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/v2-1)

Чтобы начать анализ счета, используйте следующую команду cURL. Дополнительные сведения об анализе счета см. в статье [Служба макета Распознавателя документов](../../concept-invoices.md). Перед выполнением команды внесите следующие изменения:

1. Замените `{Endpoint}` конечной точкой, полученной из подписки Распознавателя документов.
1. Замените `{your invoice URL}` на URL-адрес документа счета-фактуры.
1. Замените `{subscription key}` ключом своей подписки.

```bash
curl -v -i POST "https://{Endpoint}/formrecognizer/v2.1-preview.3/prebuilt/invoice/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key:  {subscription key}" --data-ascii "{'source': '{your invoice URL}'}"
```

Вы получите ответ `202 (Success)`, содержащий заголовок **Operation-Location**. Значение этого заголовка содержит идентификатор операции, который можно использовать для запроса состояния асинхронной операции и получения результатов.

```console
https://cognitiveservice/formrecognizer/v2.1-preview.3/prebuilt/invoice/analyzeResults/54f0b076-4e38-43e5-81bd-b85b8835fdfb
```

### <a name="get-invoice-results"></a>Получение результатов по счету-фактуре

После вызова API **[анализа счета](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/5ed8c9843c2794cbb1a96291)** вызовите API **[получения результатов анализа счета](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/5ed8c9acb78c40a2533aee83)** , чтобы узнать о состоянии операции и извлеченных данных. Перед выполнением команды внесите следующие изменения:

1. Замените `{Endpoint}` конечной точкой, полученной из ключа подписки Распознавателя документов (см. ресурс Распознавателя документов на вкладке **Обзор**).
1. Замените `{resultId}` идентификатором операции из предыдущего шага.
1. Замените `{subscription key}` ключом своей подписки.

```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.1-preview.3/prebuilt/invoice/analyzeResults/{resultId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="examine-the-response"></a>Изучите ответ.

Вы получите ответ `200 (Success)` со следующими выходными данными JSON. 
* Поле `"readResults"` содержит каждую строку текста, извлеченную из счета.
* `"pageResults"` включает таблицы и метки выбора, извлеченные из счета.
* Поле `"documentResults"` содержит сведения о ключе и значении для наиболее релевантных частей счета.

Ознакомьтесь со следующим счетом-фактурой и его соответствующими выходными данными в формате JSON. 

* [Пример счета](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/tree/master/curl/form-recognizer/sample-invoice.pdf)

Это содержимое JSON сокращено для удобства чтения. См. [полный пример выходных данных на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/sample-invoice-output.json).

```json
{
    "status": "succeeded",
    "createdDateTime": "2020-11-06T23:32:11Z",
    "lastUpdatedDateTime": "2020-11-06T23:32:20Z",
    "analyzeResult": {
        "version": "2.1.0",
        "readResults": [{
            "page": 1,
            "angle": 0,
            "width": 8.5,
            "height": 11,
            "unit": "inch"
        }],
        "pageResults": [{
            "page": 1,
            "tables": [{
                "rows": 3,
                "columns": 4,
                "cells": [{
                    "rowIndex": 0,
                    "columnIndex": 0,
                    "text": "QUANTITY",
                    "boundingBox": [0.4953,
                    5.7306,
                    1.8097,
                    5.7306,
                    1.7942,
                    6.0122,
                    0.4953,
                    6.0122]
                },
                {
                    "rowIndex": 0,
                    "columnIndex": 1,
                    "text": "DESCRIPTION",
                    "boundingBox": [1.8097,
                    5.7306,
                    5.7529,
                    5.7306,
                    5.7452,
                    6.0122,
                    1.7942,
                    6.0122]
                },
                ...
                ],
                "boundingBox": [0.4794,
                5.7132,
                8.0158,
                5.714,
                8.0118,
                6.5627,
                0.4757,
                6.5619]
            },
            {
                "rows": 2,
                "columns": 6,
                "cells": [{
                    "rowIndex": 0,
                    "columnIndex": 0,
                    "text": "SALESPERSON",
                    "boundingBox": [0.4979,
                    4.963,
                    1.8051,
                    4.963,
                    1.7975,
                    5.2398,
                    0.5056,
                    5.2398]
                },
                {
                    "rowIndex": 0,
                    "columnIndex": 1,
                    "text": "P.O. NUMBER",
                    "boundingBox": [1.8051,
                    4.963,
                    3.3047,
                    4.963,
                    3.3124,
                    5.2398,
                    1.7975,
                    5.2398]
                },
                ...
                ],
                "boundingBox": [0.4976,
                4.961,
                7.9959,
                4.9606,
                7.9959,
                5.5204,
                0.4972,
                5.5209]
            }]
        }],
        "documentResults": [{
            "docType": "prebuilt:invoice",
            "pageRange": [1,
            1],
            "fields": {
                "AmountDue": {
                    "type": "number",
                    "valueNumber": 610,
                    "text": "$610.00",
                    "boundingBox": [7.3809,
                    7.8153,
                    7.9167,
                    7.8153,
                    7.9167,
                    7.9591,
                    7.3809,
                    7.9591],
                    "page": 1,
                    "confidence": 0.875
                },
                "BillingAddress": {
                    "type": "string",
                    "valueString": "123 Bill St, Redmond WA, 98052",
                    "text": "123 Bill St, Redmond WA, 98052",
                    "boundingBox": [0.594,
                    4.3724,
                    2.0125,
                    4.3724,
                    2.0125,
                    4.7125,
                    0.594,
                    4.7125],
                    "page": 1,
                    "confidence": 0.997
                },
                "BillingAddressRecipient": {
                    "type": "string",
                    "valueString": "Microsoft Finance",
                    "text": "Microsoft Finance",
                    "boundingBox": [0.594,
                    4.1684,
                    1.7907,
                    4.1684,
                    1.7907,
                    4.2837,
                    0.594,
                    4.2837],
                    "page": 1,
                    "confidence": 0.998
                },
                ...                
            }
        }]
    }
}
```

### <a name="v20"></a>[Версия 2.0](#tab/v2-0)

> [!IMPORTANT]
> Эта функция недоступна в выбранной версии API.

---

## <a name="train-a-custom-model"></a>Обучение пользовательской модели

Чтобы обучить пользовательскую модель, вам потребуется набор обучающих данных в хранилище BLOB-объектов Azure. У вас должны быть как минимум пять заполненных форм (PDF-документы и (или) изображения) одного типа и структуры. Советы и варианты для объединения данных для обучения см. в статье [Build a training data set for a custom model](../../build-training-data-set.md) (Создание обучающего набора данных для пользовательской модели).

Обучение без помеченных данных является стандартной и более простой операцией. Кроме того, можно заранее пометить некоторые или все обучающие данные вручную. Это более сложный процесс, но его результатом является лучшая обученная модель.

> [!NOTE]
> Также можно обучить модель с графическим пользовательским интерфейсом, такой как [пример средства разметки Распознавателя документов](../../quickstarts/label-tool.md).


### <a name="train-a-model-without-labels"></a>Обучение моделей без меток

Чтобы обучить модель Распознавателя документов с помощью документов в контейнере больших двоичных объектов Azure, вызовите API-интерфейс **[обучения пользовательской модели](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/TrainCustomModelAsync)** , выполнив команду cURL, показанную ниже. Перед выполнением команды внесите следующие изменения:

1. Замените `{Endpoint}` конечной точкой, полученной из подписки Распознавателя документов.
1. Замените `{subscription key}` ключом подписки, скопированным на предыдущем шаге.
1. Замените `{SAS URL}` подписанным URL-адресом контейнера хранилища BLOB-объектов Azure. [!INCLUDE [get SAS URL](../sas-instructions.md)]

   :::image type="content" source="../../media/quickstarts/get-sas-url.png" alt-text="Получение подписанного URL-адреса":::

### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/v2-1)

```bash
curl -i -X POST "https://{Endpoint}/formrecognizer/v2.1-preview.3/custom/models" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ 'source': '{SAS URL}'}"
```

### <a name="v20"></a>[Версия 2.0](#tab/v2-0)

```bash
curl -i -X POST "https://{Endpoint}/formrecognizer/v2.0/custom/models" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ 'source': '{SAS URL}'}"
```

---

Вы получите ответ `201 (Success)` с заголовком **Location**. Значение этого заголовка — это идентификатор новой обучаемой модели.

### <a name="train-a-model-with-labels"></a>Обучение моделей с метками

Для обучения с метками в контейнере хранилища BLOB-объектов вместе с документами, используемыми для обучения, должны находиться особые файлы с информацией о метках (`\<filename\>.pdf.labels.json`). Эти файлы можно создать в пользовательском интерфейсе [средства разметки Распознавателя документов](../../quickstarts/label-tool.md). После этого можно вызвать API **[обучения настраиваемой модели](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/TrainCustomModelAsync)** , указав для параметра `"useLabelFile"` значение `true` в тексте JSON.

Перед выполнением команды внесите следующие изменения:

1. Замените `{Endpoint}` конечной точкой, полученной из подписки Распознавателя документов.
1. Замените `{subscription key}` ключом подписки, скопированным на предыдущем шаге.
1. Замените `{SAS URL}` подписанным URL-адресом контейнера хранилища BLOB-объектов Azure. [!INCLUDE [get SAS URL](../sas-instructions.md)]

   :::image type="content" source="../../media/quickstarts/get-sas-url.png" alt-text="Получение подписанного URL-адреса":::

### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/v2-1)

```bash
curl -i -X POST "https://{Endpoint}/formrecognizer/v2.1-preview.3/custom/models" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ 'source': '{SAS URL}', 'useLabelFile':true}"
```

### <a name="v20"></a>[Версия 2.0](#tab/v2-0)

```bash
curl -i -X POST "https://{Endpoint}/formrecognizer/v2.0/custom/models" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ 'source': '{SAS URL}', 'useLabelFile':true }"
```

---

Вы получите ответ `201 (Success)` с заголовком **Location**. Значение этого заголовка — это идентификатор новой обучаемой модели.

### <a name="get-training-results"></a>Получение результатов обучения

После начала операции обучения используется новая операция, **[Get Custom Model](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/GetCustomModel)** (Получение пользовательской модели), для проверки состояния обучения. Передайте идентификатор модели в этот вызов API, чтобы проверить состояние обучения:

1. Замените `{Endpoint}` конечной точкой, полученной из ключа подписки Распознавателя документов
1. Замените `{subscription key}` ключом своей подписки.
1. Замените `{model ID}` идентификатором модели, полученным на предыдущем шаге.

### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/v2-1)

```bash
curl -X GET "https://{Endpoint}/formrecognizer/v2.1-preview.3/custom/models/{model ID}" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="v20"></a>[Версия 2.0](#tab/v2-0)

```bash
curl -X GET "https://{Endpoint}/formrecognizer/v2.0/custom/models/{model ID}" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

---

Вы получите ответ `200 (Success)` с текстом JSON в следующем формате. Обратите внимание на поле `"status"`. После завершения обучения оно будет иметь значение `"ready"`. Если обучение модели не завершено, необходимо повторить запрос к службе, повторно выполнив команду. Мы рекомендуем установить между вызовами интервал в одну секунду или более.

Поле `"modelId"` содержит идентификатор обучаемой модели. Он потребуется для выполнения следующего шага.

```json
{
  "modelInfo":{
    "status":"ready",
    "createdDateTime":"2019-10-08T10:20:31.957784",
    "lastUpdatedDateTime":"2019-10-08T14:20:41+00:00",
    "modelId":"1cfb372bab404ba3aa59481ab2c63da5"
  },
  "trainResult":{
    "trainingDocuments":[
      {
        "documentName":"invoices\\Invoice_1.pdf",
        "pages":1,
        "errors":[

        ],
        "status":"succeeded"
      },
      {
        "documentName":"invoices\\Invoice_2.pdf",
        "pages":1,
        "errors":[

        ],
        "status":"succeeded"
      },
      {
        "documentName":"invoices\\Invoice_3.pdf",
        "pages":1,
        "errors":[

        ],
        "status":"succeeded"
      },
      {
        "documentName":"invoices\\Invoice_4.pdf",
        "pages":1,
        "errors":[

        ],
        "status":"succeeded"
      },
      {
        "documentName":"invoices\\Invoice_5.pdf",
        "pages":1,
        "errors":[

        ],
        "status":"succeeded"
      }
    ],
    "errors":[

    ]
  },
  "keys":{
    "0":[
      "Address:",
      "Invoice For:",
      "Microsoft",
      "Page"
    ]
  }
}
```

## <a name="analyze-forms-with-a-custom-model"></a>анализ документов с помощью пользовательской модели;

Теперь с помощью новой обученной модели нужно проанализировать документ и извлечь из него пары "ключ — значение" и таблицы. Вызовите API **[анализа формы](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/AnalyzeWithCustomForm)** , выполнив следующую команду. Перед выполнением команды внесите следующие изменения:

1. Замените `{Endpoint}` конечной точкой, которую вы получили из своего ключа подписки Распознавателя документов (см. ресурс Распознавателя документов на вкладке **Обзор**).
1. Замените `{model ID}` идентификатором модели, полученным в предыдущем разделе.
1. Замените `{SAS URL}` URL-адресом SAS для файла в службе хранилища Azure. Выполните действия, описанные в разделе "Обучение", но вместо получения URL-адреса SAS для всего контейнера больших двоичных объектов, получите его для определенного файла, который нужно проанализировать.
1. Замените `{subscription key}` ключом своей подписки.

### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/v2-1)

```bash
curl -v "https://{Endpoint}/formrecognizer/v2.1-preview.3/custom/models/{model ID}/analyze?includeTextDetails=true" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" -d "{ 'source': '{SAS URL}' } "
```

### <a name="v20"></a>[Версия 2.0](#tab/v2-0)

```bash
curl -v "https://{Endpoint}/formrecognizer/v2.0/custom/models/{model ID}/analyze?includeTextDetails=true" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" -d "{ 'source': '{SAS URL}' } "
```

---

Вы получите ответ `202 (Success)`, содержащий заголовок **Operation-Location**. Значение этого заголовка содержит идентификатор результатов, используемый для отслеживания результатов операции анализа. Сохраните этот идентификатор результатов для следующего шага.

### <a name="get-the-analyze-results"></a>Получение результатов анализа

Вызовите функцию API **[получения результата анализа формы](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/GetAnalyzeFormResult)** , чтобы запросить результаты операции анализа.

1. Замените `{Endpoint}` конечной точкой, которую вы получили из своего ключа подписки Распознавателя документов (см. ресурс Распознавателя документов на вкладке **Обзор**).
1. Замените `{result ID}` идентификатором, полученным в предыдущем разделе.
1. Замените `{subscription key}` ключом своей подписки.

### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/v2-1)

```bash
curl -X GET "https://{Endpoint}/formrecognizer/v2.1-preview.3/custom/models/{model ID}/analyzeResults/{result ID}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="v20"></a>[Версия 2.0](#tab/v2-0)

```bash
curl -X GET "https://{Endpoint}/formrecognizer/v2.0/custom/models/{model ID}/analyzeResults/{result ID}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

---

Вы получите ответ `200 (Success)` с текстом JSON в следующем формате. Выходные данные сокращены для простоты. Обратите внимание на поле `"status"` внизу. По завершении операции анализа оно будет иметь значение `"succeeded"`. Если операция анализа не завершена, необходимо повторить запрос к службе, повторно выполнив команду. Мы рекомендуем установить между вызовами интервал в одну секунду или более.

В пользовательских моделях, обученных без меток, пары "ключ-значение" и таблицы находятся в узле `"pageResults"` выходных данных JSON. В пользовательских моделях, обученных с метками, пары "ключ-значение" находится в узле `"documentResults"`. Если вы также указали извлечение простого текста с помощью параметра URL-адреса *includeTextDetails*, то на узле `"readResults"` будет отображаться содержимое и позиции всего текста в документе.

Выходные данные JSON сокращены для простоты. См. [полный пример выходных данных на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/analyze-result-invoice-6.pdf.json).

### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/v2-1)

```JSON
{
  "status": "succeeded",
  "createdDateTime": "2020-08-21T01:13:28Z",
  "lastUpdatedDateTime": "2020-08-21T01:13:42Z",
  "analyzeResult": {
    "version": "2.1.0",
    "readResults": [
      {
        "page": 1,
        "angle": 0,
        "width": 8.5,
        "height": 11,
        "unit": "inch",
        "lines": [
          {
            "text": "Project Statement",
            "boundingBox": [
              5.0444,
              0.3613,
              8.0917,
              0.3613,
              8.0917,
              0.6718,
              5.0444,
              0.6718
            ],
            "words": [
              {
                "text": "Project",
                "boundingBox": [
                  5.0444,
                  0.3587,
                  6.2264,
                  0.3587,
                  6.2264,
                  0.708,
                  5.0444,
                  0.708
                ]
              },
              {
                "text": "Statement",
                "boundingBox": [
                  6.3361,
                  0.3635,
                  8.0917,
                  0.3635,
                  8.0917,
                  0.6396,
                  6.3361,
                  0.6396
                ]
              }
            ]
          }, 
          ...
        ] 
      }
    ],
    "pageResults": [
      {
        "page": 1,
        "keyValuePairs": [
          {
            "key": {
              "text": "Date:",
              "boundingBox": [
                6.9833,
                1.0615,
                7.3333,
                1.0615,
                7.3333,
                1.1649,
                6.9833,
                1.1649
              ],
              "elements": [
                "#/readResults/0/lines/2/words/0"
              ]
            },
            "value": {
              "text": "9/10/2020",
              "boundingBox": [
                7.3833,
                1.0802,
                7.925,
                1.0802,
                7.925,
                1.174,
                7.3833,
                1.174
              ],
              "elements": [
                "#/readResults/0/lines/3/words/0"
              ]
            },
            "confidence": 1
          },
          ...
        ], 
        "tables": [
          {
            "rows": 5,
            "columns": 5,
            "cells": [
              {
                "text": "Training Date",
                "rowIndex": 0,
                "columnIndex": 0,
                "boundingBox": [
                  0.6944,
                  4.2779,
                  1.5625,
                  4.2779,
                  1.5625,
                  4.4005,
                  0.6944,
                  4.4005
                ],
                "confidence": 1,
                "rowSpan": 1,
                "columnSpan": 1,
                "elements": [
                  "#/readResults/0/lines/15/words/0",
                  "#/readResults/0/lines/15/words/1"
                ],
                "isHeader": true,
                "isFooter": false
              },
              ...
            ]
          }
        ], 
        "clusterId": 0
      }
    ], 
    "documentResults": [],
    "errors": []
  }
}
```

### <a name="v20"></a>[Версия 2.0](#tab/v2-0)

```JSON
{
  "status": "succeeded",
  "createdDateTime": "2020-08-21T00:46:25Z",
  "lastUpdatedDateTime": "2020-08-21T00:46:32Z",
  "analyzeResult": {
    "version": "2.0.0",
    "readResults": [
      {
        "page": 1,
        "angle": 0,
        "width": 8.5,
        "height": 11,
        "unit": "inch",
        "lines": [
          {
            "text": "Project Statement",
            "boundingBox": [
              5.0153,
              0.275,
              8.0944,
              0.275,
              8.0944,
              0.7125,
              5.0153,
              0.7125
            ],
            "words": [
              {
                "text": "Project",
                "boundingBox": [
                  5.0153,
                  0.275,
                  6.2278,
                  0.275,
                  6.2278,
                  0.7125,
                  5.0153,
                  0.7125
                ]
              },
              {
                "text": "Statement",
                "boundingBox": [
                  6.3292,
                  0.275,
                  8.0944,
                  0.275,
                  8.0944,
                  0.7125,
                  6.3292,
                  0.7125
                ]
              }
            ]
          }, 
        ...
        ]
      }
    ],
    "pageResults": [
      {
        "page": 1,
        "keyValuePairs": [
          {
            "key": {
              "text": "Date:",
              "boundingBox": [
                6.9722,
                1.0264,
                7.3417,
                1.0264,
                7.3417,
                1.1931,
                6.9722,
                1.1931
              ],
              "elements": [
                "#/readResults/0/lines/2/words/0"
              ]
            },
            "confidence": 1
          },
         ...
        ],
        "tables": [
          {
            "rows": 4,
            "columns": 5,
            "cells": [
              {
                "text": "Training Date",
                "rowIndex": 0,
                "columnIndex": 0,
                "boundingBox": [
                  0.6931,
                  4.2444,
                  1.5681,
                  4.2444,
                  1.5681,
                  4.4125,
                  0.6931,
                  4.4125
                ],
                "confidence": 1,
                "rowSpan": 1,
                "columnSpan": 1,
                "elements": [
                  "#/readResults/0/lines/15/words/0",
                  "#/readResults/0/lines/15/words/1"
                ],
                "isHeader": true,
                "isFooter": false
              },
              ...
            ]
          }
        ], 
        "clusterId": 0
      }
    ],
    "documentResults": [],
    "errors": []
  }
}
```  

---

### <a name="improve-results"></a>Улучшение результатов

[!INCLUDE [improve results](../improve-results-unlabeled.md)]

## <a name="analyze-receipts"></a>Анализ квитанций

В этом разделе объясняется, как с помощью предварительно обученной модели можно анализировать используемые в США квитанции и извлекать из них содержимое стандартных полей. Дополнительные сведения об анализе квитанций см. в статье [Готовая модель получения Распознавателя форм](../../concept-receipts.md). Для начала анализа квитанции запустите API **[анализа](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/AnalyzeReceiptAsync)** , используя приведенную ниже команду cURL. Перед выполнением команды внесите следующие изменения:

1. Замените `{Endpoint}` конечной точкой, полученной из подписки Распознавателя документов.
1. Замените `{your receipt URL}` на URL-адрес изображения квитанции.
1. Замените `{subscription key>` ключом подписки, скопированным на предыдущем шаге.

### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/v2-1)

```bash
curl -i -X POST "https://{Endpoint}/formrecognizer/v2.1-preview.3/prebuilt/receipt/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ 'source': '{your receipt URL}'}"
```

### <a name="v20"></a>[Версия 2.0](#tab/v2-0)

```bash
curl -i -X POST "https://{Endpoint}/formrecognizer/v2.0/prebuilt/receipt/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ 'source': '{your receipt URL}'}"
```

---

Вы получите ответ `202 (Success)`, содержащий заголовок **Operation-Location**. Значение этого заголовка содержит идентификатор операции, который можно использовать для запроса состояния асинхронной операции и получения результатов. В следующем примере строка после `operations/` является идентификатором операции.

```console
https://cognitiveservice/formrecognizer/v2.0/prebuilt/receipt/operations/54f0b076-4e38-43e5-81bd-b85b8835fdfb
```

### <a name="get-the-receipt-results"></a>Получение результатов анализа данных квитанции

После вызова API **анализа квитанции** вызовите API **[получения результатов анализа квитанции](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/GetAnalyzeReceiptResult)** , чтобы узнать о состоянии операции и извлеченных данных. Перед выполнением команды внесите следующие изменения:

1. Замените `{Endpoint}` конечной точкой, полученной из ключа подписки Распознавателя документов (см. ресурс Распознавателя документов на вкладке **Обзор**).
1. Замените `{operationId}` идентификатором операции из предыдущего шага.
1. Замените `{subscription key}` ключом своей подписки.

### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/v2-1)

```bash
curl -X GET "https://{Endpoint}/formrecognizer/v2.1-preview.3/prebuilt/receipt/analyzeResults/{operationId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="v20"></a>[Версия 2.0](#tab/v2-0)

```bash
curl -X GET "https://{Endpoint}/formrecognizer/v2.0/prebuilt/receipt/analyzeResults/{operationId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

---

### <a name="examine-the-response"></a>Изучите ответ.

Вы получите ответ `200 (Success)` со следующими выходными данными JSON. В первом поле, `"status"`, указывается состояние операции. Если операция не завершена, значение поля `"status"` будет `"running"` или `"notStarted"`, и вам снова понадобится вызвать API вручную или с помощью сценария. Мы рекомендуем установить между вызовами интервал в одну секунду или более.

Узел `"readResults"` содержит весь распознанный текст (если для необязательного параметра *includeTextDetails* вы задали значение `true`). Текст упорядочивается по страницам, затем по строкам, а затем по отдельным словам. Узел `"documentResults"` содержит обнаруженные моделью значения, зависящие от квитанции. Здесь вы найдете полезные пары "ключ — значение", такие как налог, итог, адрес продавца и т. д.

Ознакомьтесь со следующим изображением квитанции и его соответствующими выходными данными в формате JSON.

![Квитанция из магазина Contoso](../../media/contoso-allinone.jpg)

Эти выходные данные сокращены для удобства чтения. См. [полный пример выходных данных на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/receipt-result.json).

```json
{
  "status":"succeeded",
  "createdDateTime":"2019-12-17T04:11:24Z",
  "lastUpdatedDateTime":"2019-12-17T04:11:32Z",
  "analyzeResult":{
    "version":"2.1.0",
    "readResults":[
      {
        "page":1,
        "angle":0.6893,
        "width":1688,
        "height":3000,
        "unit":"pixel",
        "language":"en",
        "lines":[
          {
            "text":"Contoso",
            "boundingBox":[
              635,
              510,
              1086,
              461,
              1098,
              558,
              643,
              604
            ],
            "words":[
              {
                "text":"Contoso",
                "boundingBox":[
                  639,
                  510,
                  1087,
                  461,
                  1098,
                  551,
                  646,
                  604
                ],
                "confidence":0.955
              }
            ]
          },
          ...
        ]
      }
    ],
    "documentResults":[
      {
        "docType":"prebuilt:receipt",
        "pageRange":[
          1,
          1
        ],
        "fields":{
          "ReceiptType":{
            "type":"string",
            "valueString":"Itemized",
            "confidence":0.692
          },
          "MerchantName":{
            "type":"string",
            "valueString":"Contoso Contoso",
            "text":"Contoso Contoso",
            "boundingBox":[
              378.2,
              292.4,
              1117.7,
              468.3,
              1035.7,
              812.7,
              296.3,
              636.8
            ],
            "page":1,
            "confidence":0.613,
            "elements":[
              "#/readResults/0/lines/0/words/0",
              "#/readResults/0/lines/1/words/0"
            ]
          },
          "MerchantAddress":{
            "type":"string",
            "valueString":"123 Main Street Redmond, WA 98052",
            "text":"123 Main Street Redmond, WA 98052",
            "boundingBox":[
              302,
              675.8,
              848.1,
              793.7,
              809.9,
              970.4,
              263.9,
              852.5
            ],
            "page":1,
            "confidence":0.99,
            "elements":[
              "#/readResults/0/lines/2/words/0",
              "#/readResults/0/lines/2/words/1",
              "#/readResults/0/lines/2/words/2",
              "#/readResults/0/lines/3/words/0",
              "#/readResults/0/lines/3/words/1",
              "#/readResults/0/lines/3/words/2"
            ]
          },
          "MerchantPhoneNumber":{
            "type":"phoneNumber",
            "valuePhoneNumber":"+19876543210",
            "text":"987-654-3210",
            "boundingBox":[
              278,
              1004,
              656.3,
              1054.7,
              646.8,
              1125.3,
              268.5,
              1074.7
            ],
            "page":1,
            "confidence":0.99,
            "elements":[
              "#/readResults/0/lines/4/words/0"
            ]
          },
          "TransactionDate":{
            "type":"date",
            "valueDate":"2019-06-10",
            "text":"6/10/2019",
            "boundingBox":[
              265.1,
              1228.4,
              525,
              1247,
              518.9,
              1332.1,
              259,
              1313.5
            ],
            "page":1,
            "confidence":0.99,
            "elements":[
              "#/readResults/0/lines/5/words/0"
            ]
          },
          "TransactionTime":{
            "type":"time",
            "valueTime":"13:59:00",
            "text":"13:59",
            "boundingBox":[
              541,
              1248,
              677.3,
              1261.5,
              668.9,
              1346.5,
              532.6,
              1333
            ],
            "page":1,
            "confidence":0.977,
            "elements":[
              "#/readResults/0/lines/5/words/1"
            ]
          },
          "Items":{
            "type":"array",
            "valueArray":[
              {
                "type":"object",
                "valueObject":{
                  "Quantity":{
                    "type":"number",
                    "text":"1",
                    "boundingBox":[
                      245.1,
                      1581.5,
                      300.9,
                      1585.1,
                      295,
                      1676,
                      239.2,
                      1672.4
                    ],
                    "page":1,
                    "confidence":0.92,
                    "elements":[
                      "#/readResults/0/lines/7/words/0"
                    ]
                  },
                  "Name":{
                    "type":"string",
                    "valueString":"Cappuccino",
                    "text":"Cappuccino",
                    "boundingBox":[
                      322,
                      1586,
                      654.2,
                      1601.1,
                      650,
                      1693,
                      317.8,
                      1678
                    ],
                    "page":1,
                    "confidence":0.923,
                    "elements":[
                      "#/readResults/0/lines/7/words/1"
                    ]
                  },
                  "TotalPrice":{
                    "type":"number",
                    "valueNumber":2.2,
                    "text":"$2.20",
                    "boundingBox":[
                      1107.7,
                      1584,
                      1263,
                      1574,
                      1268.3,
                      1656,
                      1113,
                      1666
                    ],
                    "page":1,
                    "confidence":0.918,
                    "elements":[
                      "#/readResults/0/lines/8/words/0"
                    ]
                  }
                }
              },
              ...
            ]
          },
          "Subtotal":{
            "type":"number",
            "valueNumber":11.7,
            "text":"11.70",
            "boundingBox":[
              1146,
              2221,
              1297.3,
              2223,
              1296,
              2319,
              1144.7,
              2317
            ],
            "page":1,
            "confidence":0.955,
            "elements":[
              "#/readResults/0/lines/13/words/1"
            ]
          },
          "Tax":{
            "type":"number",
            "valueNumber":1.17,
            "text":"1.17",
            "boundingBox":[
              1190,
              2359,
              1304,
              2359,
              1304,
              2456,
              1190,
              2456
            ],
            "page":1,
            "confidence":0.979,
            "elements":[
              "#/readResults/0/lines/15/words/1"
            ]
          },
          "Tip":{
            "type":"number",
            "valueNumber":1.63,
            "text":"1.63",
            "boundingBox":[
              1094,
              2479,
              1267.7,
              2485,
              1264,
              2591,
              1090.3,
              2585
            ],
            "page":1,
            "confidence":0.941,
            "elements":[
              "#/readResults/0/lines/17/words/1"
            ]
          },
          "Total":{
            "type":"number",
            "valueNumber":14.5,
            "text":"$14.50",
            "boundingBox":[
              1034.2,
              2617,
              1387.5,
              2638.2,
              1380,
              2763,
              1026.7,
              2741.8
            ],
            "page":1,
            "confidence":0.985,
            "elements":[
              "#/readResults/0/lines/19/words/0"
            ]
          }
        }
      }
    ]
  }
}
```

## <a name="analyze-business-cards"></a>Анализ визитных карточек

### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/v2-1)  

В этом разделе показано, как с помощью предварительно обученной модели анализировать визитные карточки на английском языке и извлекать из них содержимое стандартных полей. Дополнительные сведения об анализе визитных карточек см. в статье [Готовая модель для анализа визитных карточек с помощью Распознавателя форм](../../concept-business-cards.md). Для начала анализа визитной карточки запустите API **[анализа визитных карточек](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/AnalyzeBusinessCardAsync)** , используя приведенную ниже команду cURL. Перед выполнением команды внесите следующие изменения:

1. Замените `{Endpoint}` конечной точкой, полученной из подписки Распознавателя документов.
1. Замените `{your business card URL}` на URL-адрес изображения квитанции.
1. Замените `{subscription key}` ключом подписки, скопированным на предыдущем шаге.

```bash
curl -i -X POST "https://{Endpoint}/formrecognizer/v2.1-preview.3/prebuilt/businessCard/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ 'source': '{your receipt URL}'}"
```

Вы получите ответ `202 (Success)`, содержащий заголовок **Operation-Location**. Значение этого заголовка содержит идентификатор операции, который можно использовать для запроса состояния асинхронной операции и получения результатов.

```console
https://cognitiveservice/formrecognizer/v2.1-preview.3/prebuilt/businessCard/analyzeResults/54f0b076-4e38-43e5-81bd-b85b8835fdfb
```

### <a name="get-business-card-results"></a>Получение результатов анализа визитной карточки

После вызова API **анализа визитных карточек** вызовите API **[получения результатов анализа визитных карточек](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/GetAnalyzeBusinessCardResult)** , чтобы узнать о состоянии операции и извлеченных данных. Перед выполнением команды внесите следующие изменения:

1. Замените `{Endpoint}` конечной точкой, полученной из ключа подписки Распознавателя документов (см. ресурс Распознавателя документов на вкладке **Обзор**).
1. Замените `{resultId}` идентификатором операции из предыдущего шага.
1. Замените `{subscription key}` ключом своей подписки.

```bash
curl -v -X GET "https://westcentralus.api.cognitive.microsoft.com/formrecognizer/v2.1-preview.3/prebuilt/businessCard/analyzeResults/{resultId}"
-H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="examine-the-response"></a>Изучите ответ.

Вы получите ответ `200 (Success)` со следующими выходными данными JSON. 

Узел `"readResults"` содержит весь распознанный текст. Текст упорядочивается по страницам, затем по строкам, а затем по отдельным словам. Узел `"documentResults"` содержит обнаруженные моделью значения для конкретной визитной карточки. Здесь вы найдете полезные контактные данные, такие как имя компании, ФИО, номер телефона и т. д.

![Визитная карточка компании Contoso](../../media/business-card-english.jpg)

Эти выходные данные JSON сокращены для удобства чтения. См. [полный пример выходных данных на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/business-card-result.json).

```json
{
    "status": "succeeded",
    "createdDateTime": "2020-06-04T08:19:29Z",
    "lastUpdatedDateTime": "2020-06-04T08:19:35Z",
    "analyzeResult": {
        "version": "2.1.1",
        "readResults": [
            {
                "page": 1,
                "angle": -17.0956,
                "width": 4032,
                "height": 3024,
                "unit": "pixel"
            }
        ],
        "documentResults": [
            {
                "docType": "prebuilt:businesscard",
                "pageRange": [
                    1,
                    1
                ],
                "fields": {
                    "ContactNames": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "object",
                                "valueObject": {
                                    "FirstName": {
                                        "type": "string",
                                        "valueString": "Avery",
                                        "text": "Avery",
                                        "boundingBox": [
                                            703,
                                            1096,
                                            1134,
                                            989,
                                            1165,
                                            1109,
                                            733,
                                            1206
                                        ],
                                        "page": 1
                                },
                                "text": "Dr. Avery Smith",
                                "boundingBox": [
                                    419.3,
                                    1154.6,
                                    1589.6,
                                    877.9,
                                    1618.9,
                                    1001.7,
                                    448.6,
                                    1278.4
                                ],
                                "confidence": 0.993
                            }
                        ]
                    },
                    "Emails": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "avery.smith@contoso.com",
                                "text": "avery.smith@contoso.com",
                                "boundingBox": [
                                    2107,
                                    934,
                                    2917,
                                    696,
                                    2935,
                                    764,
                                    2126,
                                    995
                                ],
                                "page": 1,
                                "confidence": 0.99
                            }
                        ]
                    },
                    "Websites": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "https://www.contoso.com/",
                                "text": "https://www.contoso.com/",
                                "boundingBox": [
                                    2121,
                                    1002,
                                    2992,
                                    755,
                                    3014,
                                    826,
                                    2143,
                                    1077
                                ],
                                "page": 1,
                                "confidence": 0.995
                            }
                        ]
                    }
                }
            }
        ]
    }
}
```

Сценарий выведет ответы в консоль до завершения операции **анализа визитной карточки**.

### <a name="v20"></a>[Версия 2.0](#tab/v2-0)  

> [!IMPORTANT]
> Эта функция недоступна в выбранной версии API.

---

## <a name="manage-custom-models"></a>Управление пользовательскими моделями

### <a name="get-a-list-of-custom-models"></a>получение списка пользовательских моделей.

Используйте API **[перечисления пользовательских моделей](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/GetCustomModels)** в следующей команде, чтобы получить список всех пользовательских моделей, связанных с вашей подпиской.

1. Замените `{Endpoint}` конечной точкой, полученной из подписки Распознавателя документов.
1. Замените `{subscription key}` ключом подписки, скопированным на предыдущем шаге.

### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/v2-1)

```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.1-preview.3/custom/models?op=full"
-H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="v20"></a>[Версия 2.0](#tab/v2-0)

```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.0/custom/models?op=full"
-H "Ocp-Apim-Subscription-Key: {subscription key}"
```

---

Вы получите успешный ответ `200` с данными JSON, как в следующем примере. Элемент `"modelList"` содержит все созданные модели и сведения о них.

```json
{
  "summary": {
    "count": 0,
    "limit": 0,
    "lastUpdatedDateTime": "string"
  },
  "modelList": [
    {
      "modelId": "string",
      "status": "creating",
      "createdDateTime": "string",
      "lastUpdatedDateTime": "string"
    }
  ],
  "nextLink": "string"
}
```

### <a name="get-a-specific-model"></a>Получение определенной модели

Чтобы получить дополнительные сведения об определенной пользовательской модели, используйте API **[получения пользовательской модели](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/GetCustomModel)** в следующей команде.

1. Замените `{Endpoint}` конечной точкой, полученной из подписки Распознавателя документов.
1. Замените `{subscription key}` ключом подписки, скопированным на предыдущем шаге.
1. Замените `{modelId}` идентификатором настраиваемой модели, которую необходимо найти.

### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/v2-1)

```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.1-preview.3/custom/models/{modelId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="v20"></a>[Версия 2.0](#tab/v2-0)

```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.0/custom/models/{modelId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

---

Вы получите успешный ответ `200` с данными JSON, как в следующем примере.

```json
{
  "modelInfo": {
    "modelId": "string",
    "status": "creating",
    "createdDateTime": "string",
    "lastUpdatedDateTime": "string"
  },
  "keys": {
    "clusters": {}
  },
  "trainResult": {
    "trainingDocuments": [
      {
        "documentName": "string",
        "pages": 0,
        "errors": [
          "string"
        ],
        "status": "succeeded"
      }
    ],
    "fields": [
      {
        "fieldName": "string",
        "accuracy": 0.0
      }
    ],
    "averageModelAccuracy": 0.0,
    "errors": [
      {
        "message": "string"
      }
    ]
  }
}
```

### <a name="delete-a-model-from-the-resource-account"></a>Удаление модели из учетной записи ресурсов

Модель можно удалить из учетной записи ресурсов по ее идентификатору. Эта команда вызывает API **[удаления пользовательской модели](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/DeleteCustomModel)** , чтобы удалить модель, используемую, как описано в предыдущем разделе.

1. Замените `{Endpoint}` конечной точкой, полученной из подписки Распознавателя документов.
1. Замените `{subscription key}` ключом подписки, скопированным на предыдущем шаге.
1. Замените `{modelId}` идентификатором настраиваемой модели, которую необходимо найти.

### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/v2-1)

```bash
curl -v -X DELETE "https://{Endpoint}/formrecognizer/v2.1-preview.3/custom/models/{modelId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="v20"></a>[Версия 2.0](#tab/v2-0)

```bash
curl -v -X DELETE "https://{Endpoint}/formrecognizer/v2.0/custom/models/{modelId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

---

Вы получите ответ `204`, указывающий, что модель помечена для удаления. Артефакты модели будут удалены в течение 48 часов.

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве описано использование REST API Распознавателя документов для обучения модели и анализа документов различными способами. Для более подробного изучения API Распознавателя документов см. справочную документацию.

> [!div class="nextstepaction"]
> [Справочная документация по REST API](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/AnalyzeWithCustomForm)

* [Что такое Распознаватель документов?](../../overview.md)