---
title: Краткое руководство. Клиентская библиотека оптического распознавания символов для Java
description: В этом кратком руководстве описано, как приступить к работе с клиентской библиотекой оптического распознавания символов для Java.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: include
ms.date: 12/15/2020
ms.custom: devx-track-java
ms.author: pafarley
ms.openlocfilehash: 84ac8e8309d9f1d0536d0f7a16ab9cd9f3c10a2c
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106284815"
---
<a name="HOLTop"></a>

Используйте клиентскую библиотеку оптического распознавания символов для чтения печатных и рукописных текстов в изображениях.

[Справочная документация](/java/api/overview/azure/cognitiveservices/client/computervision) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cognitiveservices/ms-azure-cs-computervision) |[Артефакт (Maven)](https://search.maven.org/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-computervision) | [Примеры](https://azure.microsoft.com/resources/samples/?service=cognitive-services&term=vision&sort=0)

## <a name="prerequisites"></a>Предварительные требования

* подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Текущая версия [пакета средств разработки Java (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html).
* [Средство сборки Gradle](https://gradle.org/install/) или другой диспетчер зависимостей.
* Получив подписку Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="создайте ресурс Компьютерного зрения"  target="_blank">create a Computer Vision resource </a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
    * Для подключения приложения к API Компьютерного зрения потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-gradle-project"></a>Создание проекта Gradle

В окне консоли (например, cmd, PowerShell или Bash) создайте новый каталог для приложения и перейдите в него. 

```console
mkdir myapp && cd myapp
```

Выполните команду `gradle init` из рабочей папки. Эта команда создает необходимые файлы сборки для Gradle, включая *build.gradle.kts*, который используется во время выполнения для создания и настройки приложения.

```console
gradle init --type basic
```

Когда появится запрос на выбор **предметно-ориентированного языка**, выберите **Kotlin**.

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

В этом кратком руководстве используется диспетчер зависимостей Gradle. Клиентскую библиотеку и информацию для других диспетчеров зависимостей можно найти в [центральном репозитории Maven](https://search.maven.org/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-computervision).

Найдите файл *build.gradle.kts* и откройте его в предпочитаемой интегрированной среде разработки или текстовом редакторе. Затем скопируйте и вставьте в файл приведенную ниже конфигурацию сборки. Эта конфигурация определяет проект как приложение Java, точкой входа которого является класс **ComputerVisionQuickstarts**. Он импортирует библиотеку Компьютерного зрения.

```kotlin
plugins {
    java
    application
}
application { 
    mainClassName = "ComputerVisionQuickstarts"
}
repositories {
    mavenCentral()
}
dependencies {
    compile(group = "com.microsoft.azure.cognitiveservices", name = "azure-cognitiveservices-computervision", version = "1.0.4-beta")
}
```

### <a name="create-a-java-file"></a>Создание файла Java

В рабочей папке выполните следующую команду, чтобы создать исходную папку проекта.

```console
mkdir -p src/main/java
```

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java), где размещены примеры кода для этого краткого руководства.

Перейдите в новую папку и создайте файл с именем *ComputerVisionQuickstarts.java*. Откройте его в предпочитаемом редакторе или интегрированной среде разработки.

### <a name="find-the-subscription-key-and-endpoint"></a>Поиск ключа подписки и конечной точки

Перейдите на портал Azure. Если ресурс Компьютерного зрения, созданный с учетом **предварительных требований**, успешно развернут, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключ подписки и конечную точку можно найти на странице **Ключи и конечная точка** ресурса в разделе **Управление ресурсами**. 

Определите класс **ComputerVisionQuickstarts**. Создайте переменные для конечной точки и ключа подписки на службу "Компьютерное зрение". Вставьте ключ подписки и конечную точку в следующий код, где указано. Конечная точка службы "Компьютерное зрение" имеет такой формат: `https://<your_computer_vision_resource_name>.cognitiveservices.azure.com/`.

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_imports_and_vars)]

> [!IMPORTANT]
> Не забудьте удалить ключ подписки из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Например, [хранилище ключей Azure](../../../../key-vault/general/overview.md).

В методе **main** приложения добавьте вызовы методов, используемых в этом кратком руководстве. Они будут определены позже.

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_beginmain)]

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_authinmain)]

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_readinmain)]

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_endmain)]
> [!div class="nextstepaction"]
> [Мной настроен клиент](?success=set-up-client#object-model) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Java&Section=set-up-client)

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы обрабатывают некоторые основные функции пакета SDK для OCR Java.

|Имя|Описание|
|---|---|
| [ComputerVisionClient](/java/api/com.microsoft.azure.cognitiveservices.vision.computervision.computervisionclient) | Этот класс требуется для всех функций Компьютерного зрения. Вы создаете его экземпляр с информацией о подписке и используете его для создания экземпляров других классов.|

## <a name="code-examples"></a>Примеры кода

Эти фрагменты кода демонстрируют выполнение следующих действий с помощью клиентской библиотеки оптического распознавания символов для Java:

* [аутентификация клиента](#authenticate-the-client);
* [Чтение печатного и рукописного текста](#read-printed-and-handwritten-text).

## <a name="authenticate-the-client"></a>Аутентификация клиента

В новом методе создайте экземпляр объекта [ComputerVisionClient](/java/api/com.microsoft.azure.cognitiveservices.vision.computervision.computervisionclient) с использованием конечной точки и ключа.

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_auth)]

> [!div class="nextstepaction"]
> [Мной аутентифицирован клиент](?success=authenticate-client#read-printed-and-handwritten-text) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Java&Section=authenticate-client)



## <a name="read-printed-and-handwritten-text"></a>Чтение печатного и рукописного текста

Служба OCR может считывать видимый текст в образе и преобразовывать его в поток символов. В этом разделе определяется метод `ReadFromFile`, который получает путь к локальному файлу и выводит текст c изображения на консоль.

> [!TIP]
> Кроме того, вы можете считать текст на удаленном изображении, на которое ссылается URL-адрес. См. подробные сведения о методах [ComputerVision](/java/api/com.microsoft.azure.cognitiveservices.vision.computervision.computervision), например **read**. Пример кода для сценариев, в которых используются удаленные изображения, см. на [сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java).

### <a name="set-up-test-image"></a>Настройка тестового изображения

Создайте папку **resources/** в папке **src/main/** вашего проекта и добавьте изображение, на котором нужно прочесть текст. Вы можете скачать [пример изображения](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/cognitive-services/Computer-vision/Images/readsample.jpg) для использования.

Затем добавьте следующее определение метода в класс **ComputerVisionQuickstarts**. Измените значение `localFilePath` в соответствии с вашим файлом изображения. 

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_read_setup)]

### <a name="call-the-read-api"></a>Вызов API чтения

Затем добавьте следующий код, чтобы вызвать метод **readInStreamWithServiceResponseAsync** для данного изображения.

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_read_call)]

Следующий блок кода извлекает идентификатор операции из ответа на вызов Read. Он использует этот идентификатор со вспомогательным методом для вывода результатов чтения текста на консоль. 

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_read_response)]

Закройте блок try-catch и определение метода.

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_read_catch)]

### <a name="get-read-results"></a>Получение результатов чтения

Затем добавьте определение для вспомогательного метода. Этот метод использует идентификатор операции из предыдущего шага для запроса операции чтения и получения результатов распознавания текста, когда они доступны.

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_read_result_helper_call)]

Остальная часть метода анализирует результаты распознавания текста и выводит их на консоль.

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_read_result_helper_print)]

Наконец, добавьте другой вспомогательный метод, который использовался выше, который извлекает идентификатор операции из начального ответа.

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_opid_extract)]

> [!div class="nextstepaction"]
> [Мной считан текст](?success=read-printed-handwritten-text#run-the-application) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Java&Section=read-printed-handwritten-text)

## <a name="run-the-application"></a>Выполнение приложения

Чтобы создать приложение, выполните следующую команду:

```console
gradle build
```

Запустите приложение, выполнив команду `gradle run`:

```console
gradle run
```

> [!div class="nextstepaction"]
> [Мной запущено приложение](?success=run-the-application#clean-up-resources) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Java&Section=run-the-application)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку Cognitive Services, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней.

* [Портал](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

> [!div class="nextstepaction"]
> [Мной очищены ресурсы](?success=clean-up-resources#next-steps) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Java&Section=clean-up-resources)

## <a name="next-steps"></a>Дальнейшие действия

Изучив это краткое руководство, вы узнали, как с помощью библиотеки для OCR Java выполнять базовые задачи. Далее ознакомьтесь со справочной документацией, чтобы узнать больше о библиотеке.

> [!div class="nextstepaction"]
>[Справочник по пакету SDK для OCR (Java)](/java/api/overview/azure/cognitiveservices/client/computervision)


* [Общие сведения об OCR](../../overview-ocr.md)
* Исходный код для этого шаблона можно найти на портале [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java).
