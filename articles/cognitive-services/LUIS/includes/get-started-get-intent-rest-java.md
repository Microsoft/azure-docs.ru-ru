---
title: Получение сведений о намерении с помощью вызова REST и Java
titleSuffix: Azure Cognitive Services
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: include
ms.date: 06/03/2020
ms.openlocfilehash: 785d0afa666534c0b6dc0a8d93efbef4a6d1d9f5
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "95098307"
---
[Справочная документация](https://westeurope.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c08) | [Пример](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/LUIS/java-predict-with-rest/Predict.java)

## <a name="prerequisites"></a>Предварительные требования

* [JDK SE](/azure/developer/java/fundamentals/java-jdk-long-term-support) (комплект разработчика Java, выпуск "Стандартный");
* [Visual Studio Code](https://code.visualstudio.com/) или привычный вам редактор кода;

## <a name="create-pizza-app"></a>Создание приложения Pizza

[!INCLUDE [Create pizza app](get-started-get-intent-create-pizza-app.md)]

## <a name="get-intent-programmatically"></a>Получение намерения программным способом

Используйте Java, чтобы запросить [конечную точку прогноза](https://aka.ms/luis-apim-v3-prediction) и получить результат прогноза.

1. Создайте новую папку для размещения проекта Java, например `java-predict-with-rest`.

1. Создайте подкаталог с именем `lib` и скопируйте в его подкаталог `lib` следующие библиотеки Java.

    * [commons-logging-1.2.jar](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-language-understanding/master/documentation-samples/quickstarts/analyze-text/java/lib/commons-logging-1.2.jar);
    * [httpclient-4.5.3.jar](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-language-understanding/master/documentation-samples/quickstarts/analyze-text/java/lib/httpclient-4.5.3.jar);
    * [httpcore-4.4.6.jar](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-language-understanding/master/documentation-samples/quickstarts/analyze-text/java/lib/httpcore-4.4.6.jar).

1. Скопируйте следующий код для создания класса в файл с именем `Predict.java`:

    [!code-java[Code snippet](~/cognitive-services-quickstart-code/java/LUIS/java-predict-with-rest/Predict.java)]

1. Замените значения, начинающиеся с `YOUR-`, собственными значениями.

    |Сведения|Назначение|
    |--|--|
    |`YOUR-APP-ID`|Идентификатор приложения. Находится на портале LUIS на странице "Application Settings" (Параметры приложения) для приложения.
    |`YOUR-PREDICTION-KEY`|Ваш ключ прогнозирования длиной в 32 символа. Находится на портале LUIS на странице "Azure Resources" (Ресурсы Azure) для приложения.
    |`YOUR-PREDICTION-ENDPOINT`| Конечная точка URL-адреса прогнозирования. Находится на портале LUIS на странице "Azure Resources" (Ресурсы Azure) для приложения.<br>Например, `https://westus.api.cognitive.microsoft.com/`.|

1. Скомпилируйте программу Java из командной строки.

    * В Windows выполните следующую команду: `javac -cp ";lib/*" Predict.java`.
    * В macOS и Linux выполните следующую команду: `javac -cp ":lib/*" Predict.java`.

1. Запустите программу Java из командной строки:

    * В Windows выполните следующую команду: `java -cp ";lib/*" Predict`.
    * В macOS и Linux выполните следующую команду: `java -cp ":lib/*" Predict`.

1. Проверьте ответ прогноза, который возвращается в формате JSON:

    ```json
    {"query":"I want two large pepperoni pizzas on thin crust please","prediction":{"topIntent":"ModifyOrder","intents":{"ModifyOrder":{"score":1.0},"None":{"score":8.55E-09},"Greetings":{"score":1.82222226E-09},"CancelOrder":{"score":1.47272727E-09},"Confirmation":{"score":9.8125E-10}},"entities":{"Order":[{"FullPizzaWithModifiers":[{"PizzaType":["pepperoni pizzas"],"Size":[["Large"]],"Quantity":[2],"Crust":[["Thin"]],"$instance":{"PizzaType":[{"type":"PizzaType","text":"pepperoni pizzas","startIndex":17,"length":16,"score":0.9978157,"modelTypeId":1,"modelType":"Entity Extractor","recognitionSources":["model"]}],"Size":[{"type":"SizeList","text":"large","startIndex":11,"length":5,"score":0.9984481,"modelTypeId":1,"modelType":"Entity Extractor","recognitionSources":["model"]}],"Quantity":[{"type":"builtin.number","text":"two","startIndex":7,"length":3,"score":0.999770939,"modelTypeId":1,"modelType":"Entity Extractor","recognitionSources":["model"]}],"Crust":[{"type":"CrustList","text":"thin crust","startIndex":37,"length":10,"score":0.933985531,"modelTypeId":1,"modelType":"Entity Extractor","recognitionSources":["model"]}]}}],"$instance":{"FullPizzaWithModifiers":[{"type":"FullPizzaWithModifiers","text":"two large pepperoni pizzas on thin crust","startIndex":7,"length":40,"score":0.90681237,"modelTypeId":1,"modelType":"Entity Extractor","recognitionSources":["model"]}]}}],"ToppingList":[["Pepperoni"]],"$instance":{"Order":[{"type":"Order","text":"two large pepperoni pizzas on thin crust","startIndex":7,"length":40,"score":0.9047088,"modelTypeId":1,"modelType":"Entity Extractor","recognitionSources":["model"]}],"ToppingList":[{"type":"ToppingList","text":"pepperoni","startIndex":17,"length":9,"modelTypeId":5,"modelType":"List Entity Extractor","recognitionSources":["model"]}]}}}}
    ```

    Ответ JSON, отформатированный для удобочитаемости:

    ```JSON
    {
      "query": "I want two large pepperoni pizzas on thin crust please",
      "prediction": {
        "topIntent": "ModifyOrder",
        "intents": {
          "ModifyOrder": {
            "score": 1
          },
          "None": {
            "score": 8.55e-9
          },
          "Greetings": {
            "score": 1.82222226e-9
          },
          "CancelOrder": {
            "score": 1.47272727e-9
          },
          "Confirmation": {
            "score": 9.8125e-10
          }
        },
        "entities": {
          "Order": [
            {
              "FullPizzaWithModifiers": [
                {
                  "PizzaType": [
                    "pepperoni pizzas"
                  ],
                  "Size": [
                    [
                      "Large"
                    ]
                  ],
                  "Quantity": [
                    2
                  ],
                  "Crust": [
                    [
                      "Thin"
                    ]
                  ],
                  "$instance": {
                    "PizzaType": [
                      {
                        "type": "PizzaType",
                        "text": "pepperoni pizzas",
                        "startIndex": 17,
                        "length": 16,
                        "score": 0.9978157,
                        "modelTypeId": 1,
                        "modelType": "Entity Extractor",
                        "recognitionSources": [
                          "model"
                        ]
                      }
                    ],
                    "Size": [
                      {
                        "type": "SizeList",
                        "text": "large",
                        "startIndex": 11,
                        "length": 5,
                        "score": 0.9984481,
                        "modelTypeId": 1,
                        "modelType": "Entity Extractor",
                        "recognitionSources": [
                          "model"
                        ]
                      }
                    ],
                    "Quantity": [
                      {
                        "type": "builtin.number",
                        "text": "two",
                        "startIndex": 7,
                        "length": 3,
                        "score": 0.999770939,
                        "modelTypeId": 1,
                        "modelType": "Entity Extractor",
                        "recognitionSources": [
                          "model"
                        ]
                      }
                    ],
                    "Crust": [
                      {
                        "type": "CrustList",
                        "text": "thin crust",
                        "startIndex": 37,
                        "length": 10,
                        "score": 0.933985531,
                        "modelTypeId": 1,
                        "modelType": "Entity Extractor",
                        "recognitionSources": [
                          "model"
                        ]
                      }
                    ]
                  }
                }
              ],
              "$instance": {
                "FullPizzaWithModifiers": [
                  {
                    "type": "FullPizzaWithModifiers",
                    "text": "two large pepperoni pizzas on thin crust",
                    "startIndex": 7,
                    "length": 40,
                    "score": 0.90681237,
                    "modelTypeId": 1,
                    "modelType": "Entity Extractor",
                    "recognitionSources": [
                      "model"
                    ]
                  }
                ]
              }
            }
          ],
          "ToppingList": [
            [
              "Pepperoni"
            ]
          ],
          "$instance": {
            "Order": [
              {
                "type": "Order",
                "text": "two large pepperoni pizzas on thin crust",
                "startIndex": 7,
                "length": 40,
                "score": 0.9047088,
                "modelTypeId": 1,
                "modelType": "Entity Extractor",
                "recognitionSources": [
                  "model"
                ]
              }
            ],
            "ToppingList": [
              {
                "type": "ToppingList",
                "text": "pepperoni",
                "startIndex": 17,
                "length": 9,
                "modelTypeId": 5,
                "modelType": "List Entity Extractor",
                "recognitionSources": [
                  "model"
                ]
              }
            ]
          }
        }
      }
    }
    ```

## <a name="clean-up-resources"></a>Очистка ресурсов

Завершив работу с этим кратким руководством, удалите папку проекта из файловой системы.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Добавление высказываний и обучение на Java](../get-started-get-model-rest-apis.md)