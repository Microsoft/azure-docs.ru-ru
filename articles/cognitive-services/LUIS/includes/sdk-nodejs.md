---
title: включить файл
description: включить файл
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.date: 09/01/2020
ms.topic: include
ms.custom: include file, devx-track-js, cog-serv-seo-aug-2020
ms.openlocfilehash: f4b9c84480940889b0278129952bcf2918d9c835
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/28/2021
ms.locfileid: "98947496"
---
С помощью клиентских библиотек Распознавания речи (LUIS) для Node.js вы можете выполнить приведенные ниже задачи.

* Создание приложения
* Добавление намерения, сущности, по которой выполнено машинное обучение, с примером речевого фрагмента
* Обучение и публикация приложения.
* Среда выполнения прогнозирующих запросов

[Справочная документация](/javascript/api/@azure/cognitiveservices-luis-authoring/) |  [Разработка](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-luis-authoring) и [Прогнозирование](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-luis-runtime) Исходный код библиотеки | [Разработка](https://www.npmjs.com/package/@azure/cognitiveservices-luis-authoring) и [Прогнозирование](https://www.npmjs.com/package/@azure/cognitiveservices-luis-runtime) NPM | [Примеры](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/LUIS/sdk-3x/index.js)

## <a name="prerequisites"></a>Предварительные требования

* [Node.js](https://nodejs.org)
* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* Получив подписку Azure, [создайте ресурс LUIS](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesLUISAllInOne) на портале Azure, чтобы получить ключ и конечную точку. Дождитесь, пока закончится развертывание, и нажмите кнопку **Перейти к ресурсу**.
    * Для подключения приложения к LUIS для разработки потребуется ключ и конечная точка для [созданного](../luis-how-to-azure-subscription.md#create-luis-resources-in-the-azure-portal) ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве. Вы можете использовать ценовую категорию "Бесплатный" (`F0`), чтобы поработать со службой.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-javascript-application"></a>Создание приложения JavaScript

1. В окне консоли создайте каталог для приложения и перейдите в него.

    ```console
    mkdir quickstart-sdk && cd quickstart-sdk
    ```

1. Инициализируйте каталог как приложение JavaScript, создав файл `package.json`.

    ```console
    npm init -y
    ```

1. Создайте файл с именем `index.js` для своего кода JavaScript.

    ```console
    touch index.js
    ```

### <a name="install-the-npm-libraries"></a>Установка библиотек NPM

В каталоге приложения установите зависимости с помощью следующей команды, выполняемой построчно.

```console
npm install @azure/ms-rest-js
npm install @azure/cognitiveservices-luis-authoring
npm install @azure/cognitiveservices-luis-runtime
```

Файл `package.json` должен выглядеть следующим образом:

```json
{
  "name": "quickstart-sdk",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "@azure/cognitiveservices-luis-authoring": "^4.0.0-preview.3",
    "@azure/cognitiveservices-luis-runtime": "^5.0.0",
    "@azure/ms-rest-js": "^2.0.8"
  }
}
```

## <a name="authoring-object-model"></a>Разработка объектной модели

Клиент для разработки LUIS является объектом [LUISAuthoringClient](/javascript/api/@azure/cognitiveservices-luis-authoring/luisauthoringclient), который проходит проверку подлинности в Azure и содержит ваш ключ разработки.

## <a name="code-examples-for-authoring"></a>Примеры кода для разработки

После создания используйте клиент для доступа к функциям, включая:

* Приложения — [Добавить](/javascript/api/@azure/cognitiveservices-luis-authoring/apps#add-applicationcreateobject--msrest-requestoptionsbase-), [Удалить](/javascript/api/@azure/cognitiveservices-luis-authoring/apps#deletemethod-string--models-appsdeletemethodoptionalparams-), [Опубликовать](/javascript/api/@azure/cognitiveservices-luis-authoring/apps#publish-string--applicationpublishobject--msrest-requestoptionsbase-)
* Примеры речевых фрагментов — [добавление по пакетам](/javascript/api/@azure/cognitiveservices-luis-authoring/examples#batch-string--string--examplelabelobject----msrest-requestoptionsbase-), [удаление по идентификатору](/javascript/api/@azure/cognitiveservices-luis-authoring/examples#deletemethod-string--string--number--msrest-requestoptionsbase-).
* Компоненты — управление [списками фраз](/javascript/api/@azure/cognitiveservices-luis-authoring/features#addphraselist-string--string--phraselistcreateobject--msrest-requestoptionsbase-).
* Модель — управление [намерениями](/javascript/api/@azure/cognitiveservices-luis-authoring/model#addintent-string--string--modelcreateobject--msrest-requestoptionsbase-) и сущностями.
* Шаблоны — управление [шаблонами](/javascript/api/@azure/cognitiveservices-luis-authoring/pattern#addpattern-string--string--patternrulecreateobject--msrest-requestoptionsbase-).
* Обучение — [обучение](/javascript/api/@azure/cognitiveservices-luis-authoring/train#trainversion-string--string--msrest-requestoptionsbase-) приложения и опрос [состояния обучения](/javascript/api/@azure/cognitiveservices-luis-authoring/train#getstatus-string--string--msrest-requestoptionsbase-).
* [Версии](/javascript/api/@azure/cognitiveservices-luis-authoring/versions) — управление с помощью клонирования, экспорта и удаления.

## <a name="prediction-object-model"></a>Объектная модель прогнозирования

Клиент для разработки LUIS является объектом [LUISAuthoringClient](/javascript/api/@azure/cognitiveservices-luis-runtime/luisruntimeclient), который проходит проверку подлинности в Azure и содержит ваш ключ разработки.

## <a name="code-examples-for-prediction-runtime"></a>Примеры кода для среды выполнения прогнозирования

После создания используйте клиент для доступа к функциям, включая:

* [Прогнозирование](/javascript/api/@azure/cognitiveservices-luis-runtime/predictionoperations#getslotprediction-string--string--predictionrequest--models-predictiongetslotpredictionoptionalparams-) по слоту `staging` или `production`
* [Прогнозирование по версии](/javascript/api/@azure/cognitiveservices-luis-runtime/predictionoperations#getversionprediction-string--string--predictionrequest--models-predictiongetversionpredictionoptionalparams-)

[!INCLUDE [Bookmark links to same article](sdk-code-examples.md)]

## <a name="add-the-dependencies"></a>Добавление зависимостей

Откройте файл `index.js` в предпочитаемом редакторе или названной интегрированной среде разработки и добавьте следующие зависимости.

[!code-javascript[Add NPM libraries to code file](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=Dependencies)]

## <a name="add-boilerplate-code"></a>Добавление стандартного кода

1. Добавьте метод `quickstart` и его вызов. Этот метод охватывает большую часть оставшегося кода. Он вызывается в конце файла.

    ```javascript
    const quickstart = async () => {

        // add calls here


    }
    quickstart()
        .then(result => console.log("Done"))
        .catch(err => {
            console.log(`Error: ${err}`)
            })
    ```

1. Добавьте оставшийся код в метод quickstart, если не указано иное.

## <a name="create-variables-for-the-app"></a>Создание переменных для приложения

Создайте два набора переменных, один из которых вам нужно изменить, а второй оставить таким, как показано в примере кода. 

1. Создайте переменные для хранения ключа разработки и имен ресурсов.

    [!code-javascript[Variables you need to change](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=VariablesYouChange)]

1. Создайте переменные для хранения конечных точек, имени приложения, версии и имени намерения.

    [!code-javascript[Variables you don't need to change](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=VariablesYouDontNeedToChangeChange)]

## <a name="authenticate-the-client"></a>Аутентификация клиента

Теперь создайте объект [CognitiveServicesCredentials](/javascript/api/@azure/ms-rest-js/apikeycredentials) с помощью ключа и используйте его со своей конечной точкой, чтобы создать объект [LUISAuthoringClient](/javascript/api/@azure/cognitiveservices-luis-authoring/luisauthoringclient).

[!code-javascript[Authenticate the client](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=AuthoringCreateClient)]

## <a name="create-a-luis-app"></a>Создание приложения LUIS

Приложение LUIS содержит модель обработки естественного языка (NLP), в том числе намерения, сущности и примеры речевых фрагментов.

Создайте метод [AppsOperation](/javascript/api/@azure/cognitiveservices-luis-authoring/apps) добавления [объекта](/javascript/api/@azure/cognitiveservices-luis-authoring/apps), чтобы создать приложение. Название, язык и региональные параметры являются обязательными свойствами.

[!code-javascript[Create a LUIS app](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=AuthoringCreateApplication)]


## <a name="create-intent-for-the-app"></a>Создание намерения для приложения
Основным объектом в модели приложения LUIS является намерение. Намерение совпадает с группировкой _намерений_ пользовательских речевых фрагментов. Пользователь может задать вопрос или сделать инструкцию в поисках конкретного _намеренного_ ответа от бота (или другого клиентского приложения). Примерами намерений являются бронирование рейса, запрос о погоде в городе назначения и запрос контактной информации для обслуживания клиентов.

Используйте метод [model.add_intent](/javascript/api/@azure/cognitiveservices-luis-authoring/model#addintent-string--string--modelcreateobject--msrest-requestoptionsbase-) с именем уникального намерения, затем передайте идентификатор приложения, идентификатор версии и новое имя намерения.

Значение `intentName` жестко запрограммировано в `OrderPizzaIntent` как одна из переменных, описанных в разделе [Создание переменных для приложения](#create-variables-for-the-app).

[!code-javascript[Create intent for the app](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=AddIntent)]

## <a name="create-entities-for-the-app"></a>Создание сущностей для приложения

Хотя сущности не требуются, они встречаются в большинстве приложений. Сущность извлекает информацию из речевого фрагмента пользователя, необходимую для выполнения намерения пользователя. Существует несколько типов [готовых](/javascript/api/@azure/cognitiveservices-luis-authoring/model#addcustomprebuiltentity-string--string--prebuiltdomainmodelcreateobject--msrest-requestoptionsbase-) и пользовательских объектов, каждый из которых имеет собственные модели объекта преобразования данных (DTO).  Обычные готовые сущности, добавляемые в ваше приложение, включают в себя [number](../luis-reference-prebuilt-number.md), [datetimeV2](../luis-reference-prebuilt-datetimev2.md), [geographyV2](../luis-reference-prebuilt-geographyv2.md) и [ordinal](../luis-reference-prebuilt-ordinal.md).

Важно знать, что объекты не отмечены намерением. Они могут и обычно относятся ко многим намерениям. Только примеры пользовательских речевых фрагментов помечены для определенного единственного намерения.

Методы создания сущностей являются частью класса [Model](/javascript/api/@azure/cognitiveservices-luis-authoring/model). Каждый тип сущности имеет собственную модель объекта преобразования данных (DTO).

Код создания сущности создает сущность машинного обучения с вложенными сущностями и функциями, применяемыми к вложенным сущностям `Quantity`.

:::image type="content" source="../media/quickstart-sdk/machine-learned-entity.png" alt-text="Снимок экрана: часть страницы портала с созданной сущностью машинного обучения с вложенными сущностями и функциями, применяемыми к вложенным сущностям Quantity.":::

[!code-javascript[Create entities for the app](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=AuthoringAddEntities)]

Поместите следующий метод над методом `quickstart`, чтобы определить идентификатор вложенной сущности Quantity для назначения ей функций.

[!code-javascript[Find subentity id](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=AuthoringSortModelObject)]

## <a name="add-example-utterance-to-intent"></a>Добавление примера высказывания в намерение

Чтобы определить намерение высказывания и извлечь сущности, приложению нужны примеры речевых фрагментов. Примеры должны быть нацелены на конкретное единственное намерение и должны отмечать все пользовательские объекты. Готовые объекты не нужно отмечать.

Добавьте примеры речевых фрагментов, создав список объектов [ExampleLabelObject](/javascript/api/@azure/cognitiveservices-luis-authoring/examplelabelobject), по одному объекту для каждого примера высказывания. Каждый пример должен пометить все сущности словарем пар "имя — значение" имени и значения сущности. Значение сущности должно быть точно таким, как оно указано в тексте примера высказывания.

:::image type="content" source="../media/quickstart-sdk/labeled-example-machine-learned-entity.png" alt-text="Снимок экрана: часть страницы с помеченным на портале примером речевого фрагмента.":::

Вызовите [examples.add](/javascript/api/@azure/cognitiveservices-luis-authoring/examples) с идентификатором приложения, идентификатором версии и примером.

[!code-javascript[Add example utterance to intent](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=AuthoringAddLabeledExamples)]

## <a name="train-the-app"></a>Обучение приложения

После создания модели приложение LUIS необходимо обучить этой версии модели. Обученную модель можно использовать в [контейнере](../luis-container-howto.md) или [опубликовать](../luis-how-to-publish-app.md) в промежуточных или рабочих слотах.

Методу [train.trainVersion](/javascript/api/@azure/cognitiveservices-luis-authoring/train#trainversion-string--string--msrest-requestoptionsbase-) требуется идентификатор приложения и идентификатор версии.

Очень маленькая модель, такая как в этом кратком руководстве, будет обучаться очень быстро. Обучение приложений рабочего уровня должно включать в себя вызов опроса с помощью метода [get_status](/javascript/api/@azure/cognitiveservices-luis-authoring/train#getstatus-string--string--msrest-requestoptionsbase-), чтобы определить, прошло ли обучение успешно. Ответ представляет собой список объектов [ModelTrainingInfo](/javascript/api/@azure/cognitiveservices-luis-authoring/modeltraininginfo) с отдельным состоянием для каждого объекта. Все объекты должны быть успешно обучены, чтобы обучение считалось завершенным.

[!code-javascript[Train the app](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=TrainAppVersion)]

## <a name="publish-app-to-production-slot"></a>Публикация приложения в рабочий слот

Опубликуйте приложение LUIS с помощью метода [app.publish](/javascript/api/@azure/cognitiveservices-luis-authoring/apps#publish-string--applicationpublishobject--msrest-requestoptionsbase-). При этом публикуется текущая обученная версия в указанный слот в конечной точке. Ваше клиентское приложение использует эту конечную точку для отправки пользовательских высказываний для прогнозирования намерения и извлечения сущности.

[!code-javascript[Publish app to production slot](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=PublishVersion)]


## <a name="authenticate-the-prediction-runtime-client"></a>Аутентификация клиента среды выполнения прогнозирования

Используйте объект msRest.ApiKeyCredentials с ключом и примените его со своей конечной точкой, чтобы создать объект [LUIS.LUISRuntimeClient](/javascript/api/@azure/cognitiveservices-luis-runtime/luisruntimeclient).

[!INCLUDE [Caution about using authoring key](caution-authoring-key.md)]

[!code-javascript [Authenticate the prediction runtime client](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=PredictionCreateClient)]

## <a name="get-prediction-from-runtime"></a>Получение прогноза из среды выполнения

Добавьте следующий код, который создает запрос к среде выполнения прогнозирования. Речевой фрагмент пользователя является частью объекта [predictionRequest](/javascript/api/@azure/cognitiveservices-luis-runtime/predictionrequest).

Метод **[luisRuntimeClient.prediction.getSlotPrediction](/javascript/api/@azure/cognitiveservices-luis-runtime/predictionoperations#getslotprediction-string--string--predictionrequest--models-predictiongetslotpredictionoptionalparams-)** принимает для выполнения запроса несколько обязательных параметров, в том числе идентификатор приложения, имя слота и объект запроса прогнозирования. Есть еще несколько необязательных параметров, например для режимов подробного протоколирования, отображения всех намерений и ведения журнала.

[!code-javascript [Get prediction from runtime](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=QueryPredictionEndpoint)]

[!INCLUDE [Prediction JSON response](sdk-json.md)]

## <a name="run-the-application"></a>Выполнение приложения

Запустите приложение, выполнив команду `node index.js` для файла quickstart.

```console
node index.js
```
