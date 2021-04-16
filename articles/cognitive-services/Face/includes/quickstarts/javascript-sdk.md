---
title: Краткое руководство по использованию клиентских библиотек Распознавания лиц Bing для JavaScript
description: Клиентская библиотека Распознавания лиц для JavaScript позволяет обнаруживать лица, находить похожие лица (поиск лиц по изображению), идентифицировать лица (поиск с распознаванием лиц) и переносить ваши данные о лицах.
services: cognitive-services
author: v-jaswel
manager: chrhoder
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: include
ms.date: 11/05/2020
ms.author: v-jawe
ms.openlocfilehash: 8f968572a357bb3c98d9c3133a7ec0a0a94dbf93
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105958033"
---
## <a name="quickstart-face-client-library-for-javascript"></a>Краткое руководство. Клиентская библиотека Распознавания лиц для JavaScript

Узнайте, как приступить к распознаванию лиц с помощью клиентской библиотеки Распознавания лиц для JavaScript. Выполните приведенные здесь действия, чтобы установить пакет и протестировать пример кода для выполнения базовых задач. В службе "Распознавание лиц" доступны передовые алгоритмы обнаружения и распознавания лиц на изображениях.

Клиентская библиотека Распознавания лиц для JavaScript позволяет выполнять такие задачи:

* [обнаружение лиц на изображении](#detect-faces-in-an-image);
* [поиск похожих лиц](#find-similar-faces);
* [Создание PersonGroup](#create-a-persongroup)
* [опознание лица](#identify-a-face);

[Справочная документация](/javascript/api/@azure/cognitiveservices-face/) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-face) | [Пакет (npm)](https://www.npmjs.com/package/@azure/cognitiveservices-face) | [Примеры](/samples/browse/?products=azure&term=face&languages=javascript)

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Последняя версия [Node.js](https://nodejs.org/en/).
* Получив подписку Azure, [создайте ресурс Распознавания лиц](https://portal.azure.com/#create/Microsoft.CognitiveServicesFace) на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
    * Для подключения приложения к API Распознавания лиц потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-nodejs-application"></a>создание приложения Node.js;

В окне консоли (например, cmd, PowerShell или Bash) создайте новый каталог для приложения и перейдите в него. 

```console
mkdir myapp && cd myapp
```

Выполните команду `npm init`, чтобы создать приложение узла с помощью файла `package.json`. 

```console
npm init
```

### <a name="install-the-client-library"></a>Установка клиентской библиотеки 

Установите пакеты NPM: `ms-rest-azure` и `azure-cognitiveservices-face`.

```console
npm install @azure/cognitiveservices-face @azure/ms-rest-js
```

Файл `package.json` этого приложения будет дополнен зависимостями.

Создайте файл с именем `index.js` и импортируйте следующие библиотеки:

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](), где размещены примеры кода для этого краткого руководства.

```javascript
const msRest = require("@azure/ms-rest-js");
const Face = require("@azure/cognitiveservices-face");
const uuid = require("uuid/v4");
```

Создайте переменные для конечной точки Azure и ключа ресурса. 

> [!IMPORTANT]
> Перейдите на портал Azure. Если ресурс "Распознавание лиц", созданный в соответствии с указаниями в разделе **Предварительные требования**, успешно развернут, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключ и конечная точка располагаются на странице **ключа и конечной точки** ресурса в разделе **управления ресурсами**. 
>
> Не забудьте удалить ключ из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Дополнительные сведения см. в статье о [безопасности в Cognitive Services](../../../cognitive-services-security.md).

```javascript
key = "<paste-your-face-key-here>"
endpoint = "<paste-your-face-endpoint-here>"
```

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы воплощают некоторые основные функции клиентской библиотеки API "Распознавание лиц" для .NET.

|Имя|Описание|
|---|---|
|[FaceClient](/javascript/api/@azure/cognitiveservices-face/faceclient) | Этот класс реализует авторизацию для использования Распознавания лиц и требуется для реализации всех ее функций. Вы создаете его экземпляр с информацией о подписке и используете его для создания экземпляров других классов. |
|[Распознавание лиц](/javascript/api/@azure/cognitiveservices-face/face)|Этот класс обрабатывает основные задачи по обнаружению и распознаванию лиц. |
|[DetectedFace](/javascript/api/@azure/cognitiveservices-face/detectedface)|Этот класс представляет все данные об отдельном лице, обнаруженном на изображении. Его можно использовать для получения подробных сведений о лице.|
|[FaceList](/javascript/api/@azure/cognitiveservices-face/facelist)|Этот класс управляет хранимыми в облаке конструкциями **FaceList**, которые включают систематизированную коллекцию лиц. |
|[PersonGroupPerson](/javascript/api/@azure/cognitiveservices-face/persongroupperson)| Этот класс управляет хранимыми в облаке конструкциями **Person**, в которых хранится коллекция лиц одного человека.|
|[PersonGroup](/javascript/api/@azure/cognitiveservices-face/persongroup)| Этот класс управляет хранимыми в облаке конструкциями **PersonGroup**, в которых хранится систематизированная коллекция объектов **Person**. |

## <a name="code-examples"></a>Примеры кода

В приведенных ниже фрагментах кода показано, как выполнять следующие задачи с клиентской библиотекой Распознавания лиц для .NET:

* [аутентификация клиента](#authenticate-the-client);
* [Определение лиц на изображении](#detect-faces-in-an-image)
* [поиск похожих лиц](#find-similar-faces);
* [Создание PersonGroup](#create-a-persongroup)
* [опознание лица](#identify-a-face);

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/Face/sdk_quickstart.js), где размещены примеры кода для этого краткого руководства.

## <a name="authenticate-the-client"></a>Аутентификация клиента

Создайте экземпляр клиента с конечной точкой и ключом. Создайте объект **[ApiKeyCredentials](/javascript/api/@azure/ms-rest-js/apikeycredentials)** с помощью ключа и используйте его со своей конечной точкой, чтобы создать объект **[FaceClient](/javascript/api/@azure/cognitiveservices-face/faceclient)** .

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="credentials":::

## <a name="declare-global-values-and-helper-function"></a>Объявление глобальных значений и вспомогательной функции

Приведенные ниже глобальные значения требуются для некоторых операций Распознавания лиц, которые будут добавлены позже.

Этот URL-адрес указывает на папку с примерами изображений. Значение UUID будет использоваться как имя и идентификатор для создаваемого объекта PersonGroup.

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="globals":::

Чтобы дождаться завершения обучения PersonGroup, будет использоваться приведенная ниже функция.

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="helpers":::

## <a name="detect-faces-in-an-image"></a>Определение лиц на изображении

### <a name="get-detected-face-objects"></a>Получение обнаруженных объектов лиц

Создайте новый метод для обнаружения лиц. Метод `DetectFaceExtract` обрабатывает три изображения, размещенные по указанному URL-адресу, и создает список объектов **[DetectedFace](/javascript/api/@azure/cognitiveservices-face/detectedface)** в памяти программы. В списке значений **[FaceAttributeType](/javascript/api/@azure/cognitiveservices-face/faceattributetype)** указано, какие признаки следует извлечь. 

Затем метод `DetectFaceExtract` анализирует и выводит данные атрибутов для каждого обнаруженного лица. Каждый атрибут должен быть указан отдельно в исходном вызове API обнаружения лиц (в списке **[FaceAttributeType](/javascript/api/@azure/cognitiveservices-face/faceattributetype)** ). Следующий код обрабатывает каждый атрибут, но вам, скорее всего, потребуются лишь один-два из них.

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="detect":::

> [!TIP]
> Обнаружение лиц можно также выполнить, используя локальное изображение. Изучите методы класса [Face](/javascript/api/@azure/cognitiveservices-face/face), например [DetectWithStreamAsync](/javascript/api/@azure/cognitiveservices-face/face#detectWithStream_msRest_HttpRequestBody__FaceDetectWithStreamOptionalParams__ServiceCallback_DetectedFace____).

## <a name="find-similar-faces"></a>Поиск похожих лиц

Следующий код принимает одно обнаруженное лицо (источник) и выполняет поиск совпадений (поиск лиц по изображению) в коллекции других лиц (цель). При обнаружении совпадения в консоль выводится идентификатор лица, с которым найдено совпадение.

### <a name="detect-faces-for-comparison"></a>Обнаружение лиц для сравнения

Во-первых, определите второй метод для обнаружения лиц. Прежде чем сравнивать лица, их нужно обнаружить на изображениях, и этот метод обнаружения оптимизирован для операций сравнения. Он не извлекает подробные атрибуты лица, как в предыдущем разделе, и использует другую модель распознавания.

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="recognize":::

### <a name="find-matches"></a>Поиск совпадений

Следующий метод обнаруживает лица в наборе целевых изображений и в одном исходном изображении. Затем он сравнивает их и находит все целевые изображения, аналогичные исходному изображению. Наконец, он выводит в консоль сведения о совпадениях.

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="find_similar":::

## <a name="identify-a-face"></a>Опознание лица

Операция [Identify](/javascript/api/@azure/cognitiveservices-face/face#identify_string____FaceIdentifyOptionalParams__ServiceCallback_IdentifyResult____) принимает изображение человека или нескольких людей и пытается опознать каждое лицо на этом изображении (поиск с распознаванием лиц). Он сравнивает каждое обнаруженное лицо с [PersonGroup](/javascript/api/@azure/cognitiveservices-face/persongroup), которая является базой данных объектов [Person](/javascript/api/@azure/cognitiveservices-face/person) с известными характеристиками лиц. Чтобы выполнить операцию Identify, сначала необходимо создать и обучить [PersonGroup](/javascript/api/@azure/cognitiveservices-face/persongroup).

### <a name="add-faces-to-persongroup"></a>Добавление лиц в PersonGroup

Создайте приведенную ниже функцию, чтобы добавить лица в [PersonGroup](/javascript/api/@azure/cognitiveservices-face/persongroup).

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="add_faces":::

### <a name="wait-for-training-of-persongroup"></a>Ожидание обучения PersonGroup

Создайте приведенную ниже вспомогательную функцию, чтобы дождаться завершения обучения **PersonGroup**.

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="wait_for_training":::

### <a name="create-a-persongroup"></a>Создание PersonGroup

Приведенный ниже код:
- создает [PersonGroup](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-face/persongroup);
- добавляет лица в **PersonGroup** посредством вызова функции `AddFacesToPersonGroup`, которая была определена ранее;
- обучает **PersonGroup**;
- определяет лица в группе **PersonGroup**.

Эта группа **PersonGroup** и связанные с ней объекты **Person** теперь готовы к использованию в операциях проверки, определения или группирования.

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="identify":::

> [!TIP]
> Объект **PersonGroup** можно также создать, используя локальные изображения. Изучите методы класса [IPersonGroupPerson](/javascript/api/@azure/cognitiveservices-face/persongroupperson), например [AddFaceFromStream](/javascript/api/@azure/cognitiveservices-face/persongroupperson#addFaceFromStream_string__string__msRest_HttpRequestBody__Models_PersonGroupPersonAddFaceFromStreamOptionalParams_).

## <a name="main"></a>Главная ветвь

Наконец, создайте функцию `main` и вызовите ее.

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="main":::

## <a name="run-the-application"></a>Выполнение приложения

Запустите приложение, выполнив команду `node` для файла quickstart.

```console
node index.js
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку Cognitive Services, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней.

* [Портал](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководство вы узнали, как с помощью клиентской библиотеки Распознавания лиц для JavaScript выполнять базовые задачи по распознаванию лиц. Далее ознакомьтесь со справочной документацией, чтобы узнать больше о библиотеке.

> [!div class="nextstepaction"]
> [Справочник по API Распознавания лиц (JavaScript)](/javascript/api/@azure/cognitiveservices-face/)

* [Что представляет собой служба "Распознавание лиц"?](../../overview.md)
* Исходный код для этого шаблона можно найти на портале [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/Face/sdk_quickstart.js).