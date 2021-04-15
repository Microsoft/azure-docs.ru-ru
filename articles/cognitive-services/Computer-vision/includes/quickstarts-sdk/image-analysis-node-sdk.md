---
title: Краткое руководство. Клиентская библиотека службы "Анализ изображений" для Node.js
description: В этом кратком руководстве описано, как приступить к работе с клиентской библиотекой API "Анализ изображений" для Node.js.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: include
ms.date: 12/15/2020
ms.author: pafarley
ms.custom: devx-track-js
ms.openlocfilehash: 80c08eb20a734c16543f2767d18d86b260f17eee
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106287427"
---
<a name="HOLTop"></a>

Используйте клиентскую библиотеку анализа изображений для анализа изображений для поиска тегов, описания текста, лиц, содержимого для взрослых и многого другого.

[Справочная документация](/javascript/api/@azure/cognitiveservices-computervision/) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-computervision) | [Пакет (npm)](https://www.npmjs.com/package/@azure/cognitiveservices-computervision) | [Примеры](https://azure.microsoft.com/resources/samples/?service=cognitive-services&term=vision&sort=0)

## <a name="prerequisites"></a>Предварительные требования

* подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Текущая версия [Node.js](https://nodejs.org/)
* Получив подписку Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="создайте ресурс Компьютерного зрения"  target="_blank">create a Computer Vision resource </a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
    * Для подключения приложения к API Компьютерного зрения потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
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

Установите `ms-rest-azure` и пакет NPM `@azure/cognitiveservices-computervision`:

```console
npm install @azure/cognitiveservices-computervision
```

Установите также модуль async:

```console
npm install async
```

Файл `package.json` этого приложения будет дополнен зависимостями.

Создайте новый файл, *index. js* и откройте его в текстовом редакторе. Добавьте в файл следующие операторы импорта.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_imports)]

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/ComputerVision/ComputerVisionQuickstart.js), где размещены примеры кода для этого краткого руководства.

Создайте переменные для конечной точки Azure и ключа ресурса.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_vars)]

> [!IMPORTANT]
> Перейдите на портал Azure. Если ресурс Компьютерного зрения, созданный с учетом **предварительных требований**, успешно развернут, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключ и конечная точка располагаются на странице **ключа и конечной точки** ресурса в разделе **управления ресурсами**. 
>
> Не забудьте удалить ключ из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Дополнительные сведения см. в статье о [безопасности в Cognitive Services](../../../cognitive-services-security.md).

> [!div class="nextstepaction"]
> [Мной настроен клиент](?success=set-up-client#object-model) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Javascript&Section=set-up-client)

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы обрабатывают некоторые основные функции пакета SDK для Анализа изображений Node.js.

|Имя|Описание|
|---|---|
| [ComputerVisionClient](/javascript/api/@azure/cognitiveservices-computervision/computervisionclient) | Этот класс требуется для всех функций Компьютерного зрения. Вы создаете его экземпляр с информацией о подписке и используете его для выполнения большинства операций с образами.|
|[VisualFeatureTypes](/javascript/api/@azure/cognitiveservices-computervision/visualfeaturetypes)| Это перечисление определяет различные типы анализа изображений, которые можно выполнить в стандартной операции анализа. Набор значений **VisualFeatureTypes** указывается в зависимости от потребностей. |

## <a name="code-examples"></a>Примеры кода

В следующих фрагментах кода показано выполнение следующих действий с помощью клиентской библиотеки Анализа изображений для Node.js:

* [аутентификация клиента](#authenticate-the-client);
* [Анализ изображения](#analyze-an-image)

## <a name="authenticate-the-client"></a>Аутентификация клиента


Создайте экземпляр клиента с конечной точкой и ключом. Создайте объект [ApiKeyCredentials](/python/api/msrest/msrest.authentication.apikeycredentials) с помощью своего ключа и конечной точкой для создания объекта [ComputerVisionClient](/javascript/api/@azure/cognitiveservices-computervision/computervisionclient).

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_client)]

Затем определите функцию `computerVision` и объявите асинхронный ряд с первичной функцией и функцией обратного вызова. Вы добавите код краткого руководства в основную функцию и вызовите `computerVision` в нижней части скрипта. Остальной код в этом кратком руководстве необходимо скопировать в функцию `computerVision`.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_functiondef_begin)]

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_functiondef_end)]

> [!div class="nextstepaction"]
> [Мной аутентифицирован клиент](?success=authenticate-client#analyze-an-image) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Javascript&Section=authenticate-client)

## <a name="analyze-an-image"></a>Анализ изображения

Код в этом разделе анализирует удаленные образы для извлечения различных визуальных компонентов. Эти операции можно выполнить как часть метода **analyzeImage** клиентского объекта, или же их можно вызвать с помощью отдельных методов. Дополнительные сведения см. в [справочной документации](/javascript/api/@azure/cognitiveservices-computervision/).

> [!NOTE]
> Можно также проанализировать локальный образ. См. подробные сведения о методах [ComputerVisionClient](/javascript/api/@azure/cognitiveservices-computervision/computervisionclient), например **analyzeImageInStream**. Либо просмотрите пример кода на [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/ComputerVision/ComputerVisionQuickstart.js) ля сценариев, включающих использование локальных изображений.

### <a name="get-image-description"></a>Получение описания изображения

Следующий код получает список созданных заголовков для изображения. Дополнительные сведения см. в статье [Описание изображений на понятном для пользователя языке](../../concept-describing-images.md).

Сначала определите URL-адрес изображения для анализа:

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_describe_image)]

Затем добавьте следующий код, чтобы получить описание образа и вывести его на консоль.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_describe)]

### <a name="get-image-category"></a>Получение категории изображения

Следующий код получает обнаруженную категорию изображения. Дополнительные сведения см. в статье [Categorize images by subject matter](../../concept-categorizing-images.md) (Классификация изображений по темам).

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_categories)]

Определите вспомогательную функцию `formatCategories`:

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_categories_format)]

### <a name="get-image-tags"></a>Получение тегов изображения

Следующий код получает набор обнаруженных тегов изображения. Дополнительные сведения см. в статье о [тегах содержимого](../../concept-tagging-images.md).

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_tags)]

Определите вспомогательную функцию `formatTags`:

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_tagsformat)]

### <a name="detect-objects"></a>Обнаружение объектов

Следующий код обнаруживает общие объекты в образе и выводит их на консоль. Дополнительные сведения см. в статье [Обнаружение объектов](../../concept-object-detection.md).

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_objects)]

Определите вспомогательную функцию `formatRectObjects` для возврата верхней, левой, нижней и правой координат, а также значений ширины и высоты.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_objectformat)]

### <a name="detect-brands"></a>Обнаружение торговых марок

Следующий код обнаруживает фирменные торговые марки и логотипы в образе и выводит их на консоль. Дополнительные сведения см. в статье [Обнаружение торговых марок](../../concept-brand-detection.md).

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_brands)]

### <a name="detect-faces"></a>Распознавание лиц

Следующий код возвращает обнаруженные лица на изображении с их координатами прямоугольника и атрибутами выбора лиц. Дополнительные сведения см. в статье [Определение лиц с помощью компьютерного зрения](../../concept-detecting-faces.md).

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_faces)]

Определите вспомогательную функцию `formatRectFaces`:

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_formatfaces)]

### <a name="detect-adult-racy-or-gory-content"></a>Обнаружение содержимого для взрослых, непристойного или жуткого содержимого

Следующий код выводит сведения об обнаружении содержимого для взрослых на изображении. Дополнительные сведения см. в статье [Detect adult content](../../concept-detecting-adult-content.md) (Обнаружение содержимого для взрослых).

Укажите URL-адрес для изображения, которое необходимо использовать:

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_adult_image)]

Затем добавьте следующий код для обнаружения содержимого для взрослых и вывода результатов на консоль.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_adult)]

### <a name="get-image-color-scheme"></a>Получение цветовой схемы изображения

Следующий код выводит обнаруженные атрибуты цвета на изображении, например доминирующие и акцентные цвета. Дополнительные сведения см. в статье [Обнаружение цветовых схем на изображениях](../../concept-detecting-color-schemes.md).

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_colors)]

Определите вспомогательную функцию `printColorScheme` для вывода сведений о цветовой схеме на консоль.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_colors_print)]

### <a name="get-domain-specific-content"></a>Получение содержимого, связанного с определенной предметной областью

Анализ изображений может использовать специализированную модель для дальнейшего анализа изображений. Дополнительные сведения см. в статье [Обнаружение содержимого, связанного с определенными предметными областями](../../concept-detecting-domain-content.md).

Сначала определите URL-адрес изображения для анализа:

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_domain_image)]

Следующий код анализирует данные об обнаруженных достопримечательностях на изображении.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_landmarks)]

Определите вспомогательную функцию `formatRectDomain`, чтобы проанализировать данные расположения обнаруженных ориентиров.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_landmarks_rect)]

### <a name="get-the-image-type"></a>Получение сведений о типе изображения

Следующий код выводит информацию о типе изображения&mdash;, независимо от того, является ли это картинкой или рисунком.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_imagetype)]

Определите вспомогательную функцию `describeType`:

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_imagetype_describe)]

> [!div class="nextstepaction"]
> [Мной проанализировано изображение](?success=analyze-image#run-the-application) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Javascript&Section=analyze-image)



## <a name="run-the-application"></a>Выполнение приложения

Запустите приложение, выполнив команду `node` для файла quickstart.

```console
node index.js
```

> [!div class="nextstepaction"]
> [Мной запущено приложение](?success=run-the-application#clean-up-resources) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Javascript&Section=run-the-application)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку Cognitive Services, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней.

* [Портал](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

> [!div class="nextstepaction"]
> [Мной очищены ресурсы](?success=clean-up-resources#next-steps) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Javascript&Section=clean-up-resources)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
>[Справочные материалы по API "Анализ изображений" (Node.js)](/javascript/api/@azure/cognitiveservices-computervision/)


* [Общие сведения об Анализе изображений](../../overview-image-analysis.md)
* Исходный код для этого шаблона можно найти на портале [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/ComputerVision/ComputerVisionQuickstart.js).
