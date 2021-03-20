---
title: Импорт речевых фрагментов с использованием Node.js в LUIS
titleSuffix: Azure Cognitive Services
description: Сведения о создании приложения LUIS программным образом из существующих данных в формате CSV с помощью API разработки LUIS.
services: cognitive-services
manager: nitinme
ms.custom: seodec18, devx-track-js
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 09/05/2019
ms.openlocfilehash: 58eb92f4d0bc3de4671ca2ece14a178a876e4a6b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91541053"
---
# <a name="build-a-luis-app-programmatically-using-nodejs"></a>Создание приложения LUIS программным способом с помощью Node.js

Служба LUIS предоставляет программный API, выполняющий те же задачи, что и веб-сайт [LUIS](luis-reference-regions.md). Это позволит сэкономить время при уже имеющихся данных, поэтому создавать приложение LUIS программным способом можно быстрее, чем путем ввода информации вручную.

[!INCLUDE [Waiting for LUIS portal refresh](./includes/wait-v3-upgrade.md)]

## <a name="prerequisites"></a>Предварительные условия

* Войдите на веб-сайт [LUIS](luis-reference-regions.md) и в параметрах учетной записи найдите [ключ разработки](luis-how-to-azure-subscription.md#authoring-key). Этот ключ используется для вызова API разработки.
* Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/), прежде чем начинать работу.
* Эта статья начинается с CSV-файла для гипотетических файлов журналов запросов пользователей. Его можно скачать [здесь](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/IoT.csv).
* Установите Node.js последней версии с NPM. Скачать эту версию можно [здесь](https://nodejs.org/en/download/).
* **(Рекомендуется)** Visual Studio Code для применения функции IntelliSense и отладки. Скачать Visual Studio Code бесплатно можно [здесь](https://code.visualstudio.com/).

Весь код в этой статье доступен в [репозитории Azure-samples распознавание речи GitHub](https://github.com/Azure-Samples/cognitive-services-language-understanding/tree/master/examples/build-app-programmatically-csv).

## <a name="map-preexisting-data-to-intents-and-entities"></a>Сопоставление существующих данных с намерениями и сущностями
Даже если у вас есть система, которая была создана без учета LUIS, и если она содержит текстовые данные, сопоставляемые с разными задачами, которые хотят выполнять пользователи, вы можете осуществлять сопоставления из существующих категорий пользовательского ввода с намерениями в LUIS. Если вы можете определить важные слова или фразы в речи пользователя, эти слова можно сопоставлять с сущностями.

Откройте файл [`IoT.csv`](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/IoT.csv). Он содержит журнал пользовательских запросов к вымышленной службе по домашней автоматике, в том числе способ их классификации, что именно сказал пользователь и некоторые столбцы с извлеченной из них полезной информацией.

![CSV-файл уже имеющихся данных](./media/luis-tutorial-node-import-utterances-csv/csv.png)

Вы видите, что столбец **RequestType** может быть намерениями, а в столбце **Request** показан пример высказывания. Другие поля могут быть сущностями, если они встречаются в высказывании. Поскольку здесь есть намерения, сущности и примеры высказываний, вы выполнили требования к примеру простого приложения.

## <a name="steps-to-generate-a-luis-app-from-non-luis-data"></a>Действия по созданию приложения LUIS из данных, отличных от LUIS
Чтобы создать приложение LUIS из CSV-файла, сделайте следующее.

* Проанализируйте данные из CSV-файла:
    * Выполните преобразование в формат, который можно передать в LUIS с помощью API разработки.
    * Из проанализированных данных соберите сведения о намерениях и сущностях.
* Выполните вызовы API разработки:
    * Создайте приложение.
    * Добавьте намерения и сущности, собранные из проанализированных данных.
    * После создания приложения LUIS можно добавить примеры высказываний из проанализированных данных.

Этот поток программы можно увидеть в последней части файла `index.js`. Скопируйте или [скачайте](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/index.js) этот код и сохраните его в `index.js`.

   [!code-javascript[Node.js code for calling the steps to build a LUIS app](~/samples-luis/examples/build-app-programmatically-csv/index.js)]


## <a name="parse-the-csv"></a>Анализ CSV-файла

Записи столбцов, содержащие высказывания в CSV-файле, необходимо анализировать в формате JSON, понятном для LUIS. Этот формат JSON должен содержать поле `intentName`, которое определяет намерение высказывания. Он также должен содержать поле `entityLabels`, которое может быть пустым в случае отсутствия записей в высказывании.

Например, запись "Turn on the lights" соответствует этому JSON:

```json
        {
            "text": "Turn on the lights",
            "intentName": "TurnOn",
            "entityLabels": [
                {
                    "entityName": "Operation",
                    "startCharIndex": 5,
                    "endCharIndex": 6
                },
                {
                    "entityName": "Device",
                    "startCharIndex": 12,
                    "endCharIndex": 17
                }
            ]
        }
```

В этом примере `intentName` поступает из запроса пользователя под заголовком столбца **Request** в CSV-файле, а `entityName` поступает из других столбцов с ключевой информацией. Например, если имеется запись для **Operation** или **Device** и эта строка также появляется в фактическом запросе, ее можно пометить как сущность. Этот процесс анализа показан в следующем коде. Вы можете скопировать или [скачать](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_parse.js) его и сохранить в `_parse.js`.

   [!code-javascript[Node.js code for parsing a CSV file to extract intents, entities, and labeled utterances](~/samples-luis/examples/build-app-programmatically-csv/_parse.js)]



## <a name="create-the-luis-app"></a>Создание приложения LUIS
После анализа данных в JSON добавьте их в приложение LUIS. Следующий код создает приложение LUIS. Скопируйте или [скачайте](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_create.js) его и сохраните в `_create.js`.

   [!code-javascript[Node.js code for creating a LUIS app](~/samples-luis/examples/build-app-programmatically-csv/_create.js)]


## <a name="add-intents"></a>Добавление намерений
В созданное приложение необходимо добавить намерения. Следующий код создает приложение LUIS. Скопируйте или [скачайте](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_intents.js) его и сохраните в `_intents.js`.

   [!code-javascript[Node.js code for creating a series of intents](~/samples-luis/examples/build-app-programmatically-csv/_intents.js)]


## <a name="add-entities"></a>Добавление сущностей
Следующий код добавляет сущности в приложение LUIS. Скопируйте или [скачайте](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_entities.js) его и сохраните в `_entities.js`.

   [!code-javascript[Node.js code for creating entities](~/samples-luis/examples/build-app-programmatically-csv/_entities.js)]



## <a name="add-utterances"></a>Добавление высказываний
После определения сущностей и намерений в приложении LUIS можно добавить высказывания. В следующем коде используется API [Utterances_AddBatch](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c09), позволяющий добавлять до 100 высказываний одновременно.  Скопируйте или [скачайте](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_upload.js) его и сохраните в `_upload.js`.

   [!code-javascript[Node.js code for adding utterances](~/samples-luis/examples/build-app-programmatically-csv/_upload.js)]


## <a name="run-the-code"></a>Выполнение кода


### <a name="install-nodejs-dependencies"></a>Установка зависимостей Node.js
Установите зависимости Node.js из NPM в терминале или командной строке.

```console
> npm install
```

### <a name="change-configuration-settings"></a>Изменение параметров конфигурации
Чтобы использовать это приложение, необходимо изменить значения в файле index.js на свой ключ конечной точки и указать имя приложения. Можно также задать язык и региональные параметры приложения или изменить номер версии.

Откройте файл index.js и измените эти значения в его верхней части.


```javascript
// Change these values
const LUIS_programmaticKey = "YOUR_AUTHORING_KEY";
const LUIS_appName = "Sample App";
const LUIS_appCulture = "en-us";
const LUIS_versionId = "0.1";
```

### <a name="run-the-script"></a>Выполнение скрипта
Запустите сценарий из терминала или командной строки с помощью Node.js.

```console
> node index.js
```

или диспетчер конфигурации служб

```console
> npm start
```

### <a name="application-progress"></a>Ход выполнения приложения
Во время работы приложения в командной строке отображается ход его выполнения. В выходных данных командной строки отображается формат ответов из LUIS.

```console
> node index.js
intents: ["TurnOn","TurnOff","Dim","Other"]
entities: ["Operation","Device","Room"]
parse done
JSON file should contain utterances. Next step is to create an app with the intents and entities it found.
Called createApp, created app with ID 314b306c-0033-4e09-92ab-94fe5ed158a2
Called addIntents for intent named TurnOn.
Called addIntents for intent named TurnOff.
Called addIntents for intent named Dim.
Called addIntents for intent named Other.
Results of all calls to addIntent = [{"response":"e7eaf224-8c61-44ed-a6b0-2ab4dc56f1d0"},{"response":"a8a17efd-f01c-488d-ad44-a31a818cf7d7"},{"response":"bc7c32fc-14a0-4b72-bad4-d345d807f965"},{"response":"727a8d73-cd3b-4096-bc8d-d7cfba12eb44"}]
called addEntity for entity named Operation.
called addEntity for entity named Device.
called addEntity for entity named Room.
Results of all calls to addEntity= [{"response":"6a7e914f-911d-4c6c-a5bc-377afdce4390"},{"response":"56c35237-593d-47f6-9d01-2912fa488760"},{"response":"f1dd440c-2ce3-4a20-a817-a57273f169f3"}]
retrying add examples...

Results of add utterances = [{"response":[{"value":{"UtteranceText":"turn on the lights","ExampleId":-67649},"hasError":false},{"value":{"UtteranceText":"turn the heat on","ExampleId":-69067},"hasError":false},{"value":{"UtteranceText":"switch on the kitchen fan","ExampleId":-3395901},"hasError":false},{"value":{"UtteranceText":"turn off bedroom lights","ExampleId":-85402},"hasError":false},{"value":{"UtteranceText":"turn off air conditioning","ExampleId":-8991572},"hasError":false},{"value":{"UtteranceText":"kill the lights","ExampleId":-70124},"hasError":false},{"value":{"UtteranceText":"dim the lights","ExampleId":-174358},"hasError":false},{"value":{"UtteranceText":"hi how are you","ExampleId":-143722},"hasError":false},{"value":{"UtteranceText":"answer the phone","ExampleId":-69939},"hasError":false},{"value":{"UtteranceText":"are you there","ExampleId":-149588},"hasError":false},{"value":{"UtteranceText":"help","ExampleId":-81949},"hasError":false},{"value":{"UtteranceText":"testing the circuit","ExampleId":-11548708},"hasError":false}]}]
upload done
```




## <a name="open-the-luis-app"></a>Открытие приложения LUIS
После выполнения скрипта вы сможете войти в [LUIS](luis-reference-regions.md) и увидеть созданное приложение LUIS в разделе **Мои приложения**. Вы также увидите высказывания, добавленные в намерения **TurnOn**, **TurnOff** и **None**.

![Намерение TurnOn](./media/luis-tutorial-node-import-utterances-csv/imported-utterances-661.png)


## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Проверьте и обучите свое приложение на веб-сайте LUIS](luis-interactive-test.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

В этом примере приложения используются следующие API LUIS:
- [создать приложение](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c36)
- [Добавление целей](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c0c)
- [Добавление сущностей](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c0e)
- [Добавить фразы продолжительностью](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c09)
