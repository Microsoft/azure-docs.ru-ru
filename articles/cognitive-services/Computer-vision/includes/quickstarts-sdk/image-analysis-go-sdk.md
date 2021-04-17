---
title: Краткое руководство. Клиентская библиотека службы "Анализ изображений" для Go
titleSuffix: Azure Cognitive Services
description: В этом кратком руководстве описано, как приступить к работе с клиентской библиотекой API "Анализ изображений" для Go.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: include
ms.date: 12/15/2020
ms.author: pafarley
ms.openlocfilehash: 780d0db40651255adbd2dd70336f5696ae99862e
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106287439"
---
<a name="HOLTop"></a>

Используйте клиентскую библиотеку анализа изображений для анализа изображений для поиска тегов, описания текста, лиц, содержимого для взрослых и многого другого.

[Справочная документация](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/computervision) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices/v2.1/computervision) | [Пакет](https://github.com/Azure/azure-sdk-for-go).

## <a name="prerequisites"></a>Предварительные требования

* подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Последняя версия [Go](https://golang.org/dl/).
* Получив подписку Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="создайте ресурс Компьютерного зрения"  target="_blank">create a Computer Vision resource </a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
    * Для подключения приложения к API Компьютерного зрения потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

## <a name="setting-up"></a>Настройка

### <a name="create-a-go-project-directory"></a>Создание каталога проекта Go

В окне консоли (cmd, PowerShell, Терминал или Bash) создайте новую рабочую область для проекта Go с именем `my-app`, и перейдите в нее.

```
mkdir -p my-app/{src, bin, pkg}  
cd my-app
```

Рабочая область будет содержать три папки:

* каталог **src** будет содержать исходный код и пакеты. Все пакеты, установленные с помощью команды `go get`, попадут в этот каталог;
* каталог **pkg** будет содержать скомпилированные объекты пакета Go. Все эти файлы имеют расширение `.a`;
* каталог **bin** будет содержать двоичные исполняемые файлы, создаваемые при запуске `go install`.

> [!TIP]
> Дополнительные сведения о структуре рабочей области Go см. в [документации по языку Go](https://golang.org/doc/code.html#Workspaces). В этом руководством содержатся сведения о настройке `$GOPATH` и `$GOROOT`.

### <a name="install-the-client-library-for-go"></a>Установка клиентской библиотеки для Go

Далее, установите клиентскую библиотеку для Go:

```bash
go get -u https://github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices/v2.1/computervision
```

Или, если вы используете dep, выполните в репозитории такую команду:

```bash
dep ensure -add https://github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices/v2.1/computervision
```

### <a name="create-a-go-application"></a>Создание приложения Go. 

Затем создайте файл в каталоге **src** с именем `sample-app.go`.

```bash
cd src
touch sample-app.go
```

Откройте `sample-app.go` в любой среде разработки или текстовом редакторе. Затем добавьте пакет и импортируйте следующие библиотеки.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_imports)]

Кроме того, объявите контекст в корне сценария. Этот объект потребуется для выполнения большинства вызовов функции Анализа изображений.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_context)]

Далее вы начнете добавлять код для выполнения различных операций службы "Компьютерное зрение".

> [!div class="nextstepaction"]
> [Мной настроен клиент](?success=set-up-client#object-model) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Go&Section=set-up-client)

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы обрабатывают некоторые основные функции пакета SDK для Анализа изображений Go.

|Имя|Описание|
|---|---|
| [BaseClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/computervision#BaseClient) | Этот класс необходим для всех функций службы "Компьютерное зрение", как, например, анализ изображений и чтение текста. Вы создаете его экземпляр с информацией о подписке и используете его для выполнения большинства операций с образами.|
|[ImageAnalysis](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/computervision#ImageAnalysis)| Этот тип содержит результаты вызова функции **AnalyzeImage**. Для каждой из функций, связанных с категорией, существуют похожие типы.|
|[VisualFeatureTypes](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/computervision#VisualFeatureTypes)| Этот тип определяет различные виды анализа изображений, которые можно выполнить в стандартной операции анализа. Набор значений VisualFeatureTypes указывается в зависимости от потребностей. |

## <a name="code-examples"></a>Примеры кода

В следующих фрагментах кода показано выполнение следующих действий с помощью клиентской библиотеки Анализа изображений для Go:

* [аутентификация клиента](#authenticate-the-client);
* [Анализ изображения](#analyze-an-image)

## <a name="authenticate-the-client"></a>Аутентификация клиента

> [!NOTE]
> В этом шаге предполагается, что вы уже [создали переменные среды](../../../cognitive-services-apis-create-account.md#configure-an-environment-variable-for-authentication) для ключа и конечной точки службы "Компьютерного зрения" с именами `COMPUTER_VISION_SUBSCRIPTION_KEY` и `COMPUTER_VISION_ENDPOINT` соответственно.

Создайте функцию `main` и добавьте в нее следующий код, чтобы создать экземпляр клиента с конечной точкой и ключом.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_client)]

> [!div class="nextstepaction"]
> [Мной аутентифицирован клиент](?success=authenticate-client#analyze-an-image) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Go&Section=authenticate-client)

## <a name="analyze-an-image"></a>Анализ изображения

Следующий код использует клиентский объект для анализа удаленного образа и вывода результатов на консоль. Вы можете получить текстовое описание, категоризацию, список тегов, обнаруженные объекты, торговые марки, лица, флажки содержимого для взрослых, основные цвета и тип изображения.

### <a name="set-up-test-image"></a>Настройка тестового изображения

Сначала сохраните ссылку на URL-адрес образа, который необходимо проанализировать. Вставьте ее в функцию `main`.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_analyze_url)]

> [!TIP]
> Можно также проанализировать локальный образ. Изучите информацию о методах класса [BaseClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/computervision#BaseClient), например о **DescribeImageInStream**. Либо просмотрите пример кода на [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/go/ComputerVision/ComputerVisionQuickstart.go) для сценариев, включающих использование локальных изображений.

### <a name="specify-visual-features"></a>Указание визуальных компонентов

Следующая функция вызывает извлечение различных визуальных компонентов из образца изображения. Эти функции будут определены в следующих разделах.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_analyze)]

### <a name="get-image-description"></a>Получение описания изображения

Следующая функция получает список созданных заголовков для изображения. Дополнительные сведения об описании изображения см. в разделе [Описание изображений](../../concept-describing-images.md).

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_analyze_describe)]

### <a name="get-image-category"></a>Получение категории изображения

Следующая функция получает обнаруженную категорию изображения. Дополнительные сведения см. в разделе [Классификация изображений](../../concept-categorizing-images.md).

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_analyze_categorize)]

### <a name="get-image-tags"></a>Получение тегов изображения

Следующая функция получает набор обнаруженных тегов изображения. Дополнительные сведения см. в разделе [Теги содержимого](../../concept-tagging-images.md).

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_tags)]

### <a name="detect-objects"></a>Обнаружение объектов

Следующая функция обнаруживает общие объекты в образе и выводит их на консоль. Дополнительные сведения см. в разделе [Обнаружение объектов](../../concept-object-detection.md).

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_objects)]

### <a name="detect-brands"></a>Обнаружение торговых марок

Следующий код обнаруживает фирменные торговые марки и логотипы в образе и выводит их на консоль. Дополнительные сведения см. в разделе [Обнаружение торговых марок](../../concept-brand-detection.md).

Сначала объявите ссылку на новый образец изображения в функции `main`.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_brand_url)]

В следующем коде определяется функция обнаружения торговой марки.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_brands)]

### <a name="detect-faces"></a>Распознавание лиц

Следующая функция возвращает обнаруженные лица на изображении с их координатами прямоугольника и определенными атрибутами лиц. Дополнительные сведения см. в разделе [Распознавание лиц](../../concept-detecting-faces.md).

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_faces)]

### <a name="detect-adult-racy-or-gory-content"></a>Обнаружение содержимого для взрослых, непристойного или жуткого содержимого

Следующая функция выводит сведения об обнаружении содержимого для взрослых на изображении. Дополнительные сведения см. в статье [Обнаружение содержимого для взрослых](../../concept-detecting-adult-content.md).

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_adult)]

### <a name="get-image-color-scheme"></a>Получение цветовой схемы изображения

Следующая функция выводит обнаруженные атрибуты цвета на изображении, например доминирующие и акцентные цвета. Подробнее см. в разделе [Цветовые схемы](../../concept-detecting-color-schemes.md).

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_color)]

### <a name="get-domain-specific-content"></a>Получение содержимого, связанного с определенной предметной областью

Анализ изображений может использовать специализированные модели для дальнейшего анализа изображений. Дополнительные сведения см. в статье [Обнаружение содержимого, связанного с определенными предметными областями](../../concept-detecting-domain-content.md). 

Следующий код анализирует данные об обнаруженных известных лицах на изображении.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_celebs)]

Следующий код анализирует данные об обнаруженных достопримечательностях на изображении.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_landmarks)]

### <a name="get-the-image-type"></a>Получение сведений о типе изображения

Следующая функция выводит информацию о типе изображения&mdash;, независимо от того, является ли это картинкой или рисунком.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_type)]

> [!div class="nextstepaction"]
> [Мной проанализировано изображение](?success=analyze-image#run-the-application) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Go&Section=analyze-image)


## <a name="run-the-application"></a>Выполнение приложения

Запустите приложение из каталога приложения с помощью команды `go run`.

```bash
go run sample-app.go
```

> [!div class="nextstepaction"]
> [Мной запущено приложение](?success=run-the-application#clean-up-resources) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Go&Section=run-the-application)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку Cognitive Services, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней.

* [Портал](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

> [!div class="nextstepaction"]
> [Мной очищены ресурсы](?success=clean-up-resources#next-steps) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Go&Section=clean-up-resources)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Справочные материалы по API "Анализ изображений" (Go)](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/computervision)


* [Общие сведения об Анализе изображений](../../overview-image-analysis.md)
* Исходный код для этого шаблона можно найти на портале [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/go/ComputerVision/ComputerVisionQuickstart.go).
