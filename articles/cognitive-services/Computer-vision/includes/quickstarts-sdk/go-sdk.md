---
title: Краткое руководство. Клиентская библиотека оптического распознавания символов для Go
titleSuffix: Azure Cognitive Services
description: В этом кратком руководстве описано, как приступить к работе с клиентской библиотекой оптического распознавания символов для Go.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: include
ms.date: 12/15/2020
ms.author: pafarley
ms.openlocfilehash: b935b8a467097eb60c095c5d317f13693ef68b62
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106284832"
---
<a name="HOLTop"></a>

Используйте клиентскую библиотеку OCR для чтения печатного и рукописного текста на изображениях.

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

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/go/ComputerVision/ComputerVisionQuickstart.go), где размещены примеры кода для этого краткого руководства.

Откройте `sample-app.go` в любой среде разработки или текстовом редакторе.

Объявите контекст в корне скрипта. Этот объект потребуется для выполнения большинства вызовов функции Компьютерного зрения.

### <a name="find-the-subscription-key-and-endpoint"></a>Поиск ключа подписки и конечной точки

Перейдите на портал Azure. Если ресурс Компьютерного зрения, созданный с учетом **предварительных требований**, успешно развернут, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключ подписки и конечную точку можно найти на странице **Ключи и конечная точка** ресурса в разделе **Управление ресурсами**. 

Создайте переменные для конечной точки и ключа подписки на службу "Компьютерное зрение". Вставьте ключ подписки и конечную точку в следующий код, где указано. Конечная точка службы "Компьютерное зрение" имеет такой формат: `https://<your_computer_vision_resource_name>.cognitiveservices.azure.com/`.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_imports_and_vars)]

> [!IMPORTANT]
> Не забудьте удалить ключ подписки из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Например, [хранилище ключей Azure](../../../../key-vault/general/overview.md).

Далее вы будете добавлять код для выполнения различных операций OCR.

> [!div class="nextstepaction"]
> [Мной настроен клиент](?success=set-up-client#object-model) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Go&Section=set-up-client)

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы обрабатывают некоторые основные функции пакета SDK OCR для Go.

|Имя|Описание|
|---|---|
| [BaseClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/computervision#BaseClient) | Этот класс необходим для всех функций службы "Компьютерное зрение", как, например, анализ изображений и чтение текста. Вы создаете его экземпляр с информацией о подписке и используете его для выполнения большинства операций с образами.|
|[ReadOperationResult](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/computervision#ReadOperationResult)| Этот тип содержит результаты операции пакетной службы чтения. |

## <a name="code-examples"></a>Примеры кода

В следующих фрагментах кода показано выполнение следующих действий с помощью клиентской библиотеки OCR для Go:

* [аутентификация клиента](#authenticate-the-client);
* [Чтение печатного и рукописного текста](#read-printed-and-handwritten-text).

## <a name="authenticate-the-client"></a>Аутентификация клиента

> [!NOTE]
> В этом шаге предполагается, что вы уже [создали переменные среды](../../../cognitive-services-apis-create-account.md#configure-an-environment-variable-for-authentication) для ключа и конечной точки службы "Компьютерного зрения" с именами `COMPUTER_VISION_SUBSCRIPTION_KEY` и `COMPUTER_VISION_ENDPOINT` соответственно.

Создайте функцию `main` и добавьте в нее следующий код, чтобы создать экземпляр клиента с конечной точкой и ключом.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_client)]

> [!div class="nextstepaction"]
> [Мной аутентифицирован клиент](?success=authenticate-client#read-printed-and-handwritten-text) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Go&Section=authenticate-client)



## <a name="read-printed-and-handwritten-text"></a>Чтение печатного и рукописного текста

Служба OCR может считывать видимый текст в образе и преобразовывать его в поток символов. Код в этом разделе определяет функцию `RecognizeTextReadAPIRemoteImage`, которая использует клиентский объект для обнаружения и извлечения печатного или рукописного текста в образе.

Добавьте образец ссылки на изображение и вызов функции в функцию `main`.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_readinmain)]

> [!TIP]
> Кроме того, можно извлечь текст из локального образа. Изучите информацию о методах класса [BaseClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/computervision#BaseClient), например о **BatchReadFileInStream**. Либо просмотрите пример кода на [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/go/ComputerVision/ComputerVisionQuickstart.go) для сценариев, включающих использование локальных изображений.

### <a name="call-the-read-api"></a>Вызов API чтения

Определите новую функцию для чтения текста, `RecognizeTextReadAPIRemoteImage`. Добавьте приведенный ниже код, который вызывает метод **BatchReadFile** для данного образа. Этот метод возвращает идентификатор операции и запускает асинхронный процесс чтения содержимого образа.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_read_call)]

### <a name="get-read-results"></a>Получение результатов чтения

Затем получите идентификатор операции, возвращенный из вызова **BatchReadFile**, и используйте его с методом **GetReadOperationResult** для запроса в службу для результатов операции. Следующий код проверяет операцию с интервалами в одну секунду, пока не будут возвращены результаты. После этого извлеченные текстовые данные выводятся на консоль.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_read_response)]

### <a name="display-read-results"></a>Отображение результатов чтения

Добавьте следующий код для анализа и отображения полученных текстовых данных и завершения определения функции.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_read_display)]

> [!div class="nextstepaction"]
> [Мной считан текст](?success=read-printed-handwritten-text#run-the-application) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Go&Section=read-printed-handwritten-text)

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
> [Справочник по API OCR (Go)](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/computervision)


* [Общие сведения об OCR](../../overview-ocr.md)
* Исходный код для этого шаблона можно найти на портале [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/go/ComputerVision/ComputerVisionQuickstart.go).
