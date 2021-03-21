---
title: Визитные карточки — распознаватель форм
titleSuffix: Azure Cognitive Services
description: Узнайте о концепциях, связанных с анализом визитных карточек, с помощью API-интерфейса распознавателя форм и ограничений.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 03/15/2021
ms.author: lajanuar
ms.openlocfilehash: 5211c1263af599eb5fd09ad276545c725ce5c867
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103467008"
---
# <a name="form-recognizer-prebuilt-business-cards-model"></a>Модель предварительно подготовленных визитных карточек распознавателя форм 

Средство распознавания форм Azure может анализировать и извлекать контактные данные из визитных карточек с помощью готовой модели бизнес-карт. Он сочетает мощные возможности оптического распознавания символов (OCR) с нашей бизнес-картой, которая позволяет извлекать основные сведения из визитных карточек на английском языке. Он извлекает личные контактные данные, название компании, должность и многое другое. Предварительно разработанный API-интерфейс для визитных карточек общедоступен в формате предварительного просмотра распознавателя версии 2.1. 

## <a name="what-does-the-business-card-service-do"></a>Что делает служба "Визитная карта"?

Предварительно созданный API-интерфейс визитной карточки извлекает ключевые поля из визитных карточек и возвращает их в организованном ответе JSON.

![Изображение, которое является элементом Contoso, из выходных данных ФОТТ + JSON](./media/business-card-example.jpg)



### <a name="fields-extracted"></a>Извлеченные поля:

|Имя| Тип | Описание | Текст | 
|:-----|:----|:----|:----|
| ContactNames | массив объектов | Имя контакта, извлеченное из визитной карточки | [{"FirstName": "John", "LastName": "Петров"}] |
| FirstName | строка | Первый (заданный) имя контакта | "John" | 
| LastName | строка | Фамилия (фамилия) контакта |     "Doe" | 
| компанинамес | Массив строк | Название компании, извлеченное из визитной карточки | [Contoso] | 
| Departments | Массив строк | Отдел или организация контакта | ["R&D"] | 
| жобтитлес | Массив строк | Название контакта в указанной должности | ["Программный инженер"] | 
| Адреса электронной почты | Массив строк | Контактная электронная почта, извлеченная из визитной карточки | ["johndoe@contoso.com"] | 
| веб-сайты; | Массив строк | Веб-сайт извлечен из визитной карточки | ["https://www.contoso.com"] | 
| Адреса | Массив строк | Адрес, извлеченный из визитной карточки | ["123. Главная улица, Редмонд, WA 98052"] | 
| мобилефонес | массив телефонных номеров | Номер мобильного телефона, извлеченный из визитной карточки | ["+ 19876543210"] |
| Факсов | массив телефонных номеров | Номер телефона факса, извлеченный из визитной карточки | ["+ 19876543211"] |
| воркфонес | массив телефонных номеров | Номер рабочего телефона, извлеченный из визитной карточки | ["+ 19876543231"] |
| осерфонес     | массив телефонных номеров | Другой номер телефона, извлеченный с визитной карточки | ["+ 19876543233"] |


API визитной карточки также может вернуть весь распознанный текст из визитной карточки. Эти выходные данные OCR включаются в ответ JSON.  

### <a name="input-requirements"></a>Требования к входным данным 

[!INCLUDE [input requirements](./includes/input-requirements-receipts.md)]

## <a name="the-analyze-business-card-operation"></a>Операция анализа бизнес-карты

В качестве входных данных [анализ визитной карты](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/AnalyzeBusinessCardAsync) принимает изображение или PDF-файл визитной карточки и извлекает интересующие его значения. Вызов возвращает поле заголовка ответа с именем `Operation-Location` . `Operation-Location`Значение — это URL-адрес, СОДЕРЖАЩИЙ идентификатор результата, который будет использоваться на следующем шаге.

|Заголовок ответа| URL-адрес результата |
|:-----|:----|
|Operation-Location | `https://cognitiveservice/formrecognizer/v2.1-preview.3/prebuilt/businessCard/analyzeResults/49a36324-fc4b-4387-aa06-090cfbf0064f` |

## <a name="the-get-analyze-business-card-result-operation"></a>Операция получения результата анализа бизнес-карты

Второй шаг — вызов операции [получения результата для анализа бизнес-карты](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/GetAnalyzeBusinessCardResult) . Эта операция принимает в качестве входных данных идентификатор результата, созданный операцией анализ бизнес-карты. Он возвращает ответ JSON, содержащий поле **состояния** со следующими возможными значениями. Эта операция вызывается итеративно, пока она не вернется со значением, которое было **выполнено** . Используйте интервал от 3 до 5 секунд, чтобы избежать превышения скорости запросов в секунду (RPS).

|Поле| Тип | Возможные значения |
|:-----|:----:|:----|
|status | строка | notStarted: операция анализа не началась.<br /><br />выполняется: выполняется операция анализа.<br /><br />Сбой: не удалось выполнить операцию анализа.<br /><br />успешно: операция анализа успешно выполнена.|

Если в поле **состояние** указано значение **успешно** , в ответе JSON будут включены сведения об этом и необязательных результатах распознавания, если они запрошены. Результат «основные сведения о бизнес-карте» организован в виде словаря значений именованных полей, где каждое значение содержит извлеченный текст, нормализованное значение, ограничивающую рамку, достоверность и соответствующие элементы слова. Результат распознавания текста организован в виде иерархии строк и слов с текстом, ограничивающим прямоугольником и сведениями о достоверности.

![Пример выходных данных визитной карточки](./media/business-card-results.png)

### <a name="sample-json-output"></a>Пример выходных данных JSON

Ответ на операцию "получить анализ результатов бизнес-карты" будет структурированным представлением визитной карточки со всей извлеченной информацией.  См. здесь [Пример файла визитной карточки](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/business-card-english.jpg) и образец выходных данных [визитной карточки](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/business-card-result.json).

См. Следующий пример успешного ответа JSON:
* Узел `"readResults"` содержит весь распознанный текст. Текст упорядочивается по страницам, затем по строкам, а затем по отдельным словам. 
* Узел `"documentResults"` содержит обнаруженные моделью значения для конкретной визитной карточки. Здесь вы найдете полезные контактные данные, такие как имя, фамилия, название организации и многое другое.

```json
{
    "status": "succeeded",
    "createdDateTime": "2020-08-20T17:41:19Z",
    "lastUpdatedDateTime": "2020-08-20T17:41:24Z",
    "analyzeResult": {
        "version": "2.1.0",
        "readResults": [
            {
                "page": 1,
                "angle": -17.0956,
                "width": 4032,
                "height": 3024,
                "unit": "pixel",
                   "lines": 
                             {
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
                        "words": [
                            {
                                "text": "Dr.",
                                "boundingBox": [
                                    419,
                            ]
    
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
                                    "LastName": {
                                        "type": "string",
                                        "valueString": "Smith",
                                        "text": "Smith",
                                        "boundingBox": [
                                            1186,
                                            976,
                                            1585,
                                            879,
                                            1618,
                                            998,
                                            1218,
                                            1096
                                        ],
                                        "page": 1
                                    }
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
                                "confidence": 0.97
                            }
                        ]
                    },
                    "JobTitles": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "Senior Researcher",
                                "text": "Senior Researcher",
                                "boundingBox": [
                                    451.8,
                                    1301.9,
                                    1313.5,
                                    1099.9,
                                    1333.8,
                                    1186.7,
                                    472.2,
                                    1388.7
                                ],
                                "page": 1,
                                "confidence": 0.99
                            }
                        ]
                    },
                    "Departments": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "Cloud & Al Department",
                                "text": "Cloud & Al Department",
                                "boundingBox": [
                                    480.1,
                                    1403.3,
                                    1590.5,
                                    1129.6,
                                    1612.6,
                                    1219.6,
                                    502.3,
                                    1493.3
                                ],
                                "page": 1,
                                "confidence": 0.99
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
                    },
                    "MobilePhones": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "phoneNumber",
                                "text": "+44 (0) 7911 123456",
                                "boundingBox": [
                                    2434.9,
                                    1033.3,
                                    3072,
                                    836,
                                    3096.2,
                                    914.3,
                                    2459.1,
                                    1111.6
                                ],
                                "page": 1,
                                "confidence": 0.99
                            }
                        ]
                    },
                    "OtherPhones": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "phoneNumber",
                                "text": "+44 (0) 20 9876 5432",
                                "boundingBox": [
                                    2473.2,
                                    1115.4,
                                    3139.2,
                                    907.7,
                                    3163.2,
                                    984.7,
                                    2497.2,
                                    1192.4
                                ],
                                "page": 1,
                                "confidence": 0.99
                            }
                        ]
                    },
                    "Faxes": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "phoneNumber",
                                "text": "+44 (0) 20 6789 2345",
                                "boundingBox": [
                                    2525,
                                    1185.4,
                                    3192.4,
                                    977.9,
                                    3217.9,
                                    1060,
                                    2550.5,
                                    1267.5
                                ],
                                "page": 1,
                                "confidence": 0.99
                            }
                        ]
                    },
                    "Addresses": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "2 Kingdom Street Paddington, London, W2 6BD",
                                "text": "2 Kingdom Street Paddington, London, W2 6BD",
                                "boundingBox": [
                                    1230,
                                    2138,
                                    2535.2,
                                    1678.6,
                                    2614.2,
                                    1903.1,
                                    1309,
                                    2362.5
                                ],
                                "page": 1,
                                "confidence": 0.977
                            }
                        ]
                    },
                    "CompanyNames": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "Contoso",
                                "text": "Contoso",
                                "boundingBox": [
                                    1152,
                                    1916,
                                    2293,
                                    1552,
                                    2358,
                                    1733,
                                    1219,
                                    2105
                                ],
                                "page": 1,
                                "confidence": 0.97
                            }
                        ]
                    }
                }
            }
        ]
    }
}
```

Следуйте инструкциям [краткого руководства,](./QuickStarts/client-library.md) чтобы реализовать извлечение данных из визитных карточек с помощью Python и REST API.

## <a name="customer-scenarios"></a>Пользовательские сценарии  

Данные, извлеченные с помощью API визитной карточки, можно использовать для выполнения различных задач. Извлечение этой контактной информации автоматически экономит время для пользователей в клиентских ролях. Ниже приведено несколько примеров того, что наши клиенты выполнили с помощью API визитной карточки:

* Извлеките контактные данные из визитных карточек и быстро создавайте телефонные контакты. 
* Интегрируйтесь с CRM, чтобы автоматически создавать контакт с помощью образов визитных карточек. 
* Следите за интересами к продажам.  
* Извлеките контактную информацию из существующих образов визитных карточек. 

API-интерфейс визитной карточки также включает [функцию обработки визитных карт в построителе искусственного интеллекта](/ai-builder/prebuilt-business-card).

## <a name="next-steps"></a>Следующие шаги

- Следуйте инструкциям [краткого руководства](./quickstarts/client-library.md) , чтобы приступить к распознаванию визитных карточек.

## <a name="see-also"></a>См. также

* [Что такое Распознаватель документов?](./overview.md)
* [Справочная документация по REST API](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/AnalyzeBusinessCardAsync)
