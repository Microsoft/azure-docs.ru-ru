---
title: Краткое руководство по использованию клиентской библиотеки Content Moderator для Java
titleSuffix: Azure Cognitive Services
description: В этом кратком руководстве показано, как начать работу с клиентской библиотекой Azure Content Moderator для Java. Встройте в свое приложение программное обеспечение для фильтрации содержимого, чтобы обеспечить соответствие нормативным требованиям и настроить надлежащую среду для пользователей.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: include
ms.date: 10/16/2020
ms.custom: devx-track-java, cog-serv-seo-aug-2020
ms.author: pafarley
ms.openlocfilehash: d1574d9728036bac393cbecb37e2097be6ef90e7
ms.sourcegitcommit: bb330af42e70e8419996d3cba4acff49d398b399
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105105820"
---
Начните работу с клиентской библиотекой Azure Content Moderator для Java. Выполните приведенные здесь действия, чтобы установить пакет Maven и протестировать пример кода для выполнения базовых задач. 

Content Moderator — это служба ИИ, позволяющая управлять содержимым, которое может носить оскорбительный характер или представлять опасность, либо нежелательным содержимым иного рода. Используйте службу модерации содержимого на основе ИИ, чтобы проверить текст, изображения и видео, а также автоматически применить флаги содержимого. Встройте в свое приложение программное обеспечение для фильтрации содержимого, чтобы обеспечить соответствие нормативным требованиям и настроить надлежащую среду для пользователей.

Клиентскую библиотеку Content Moderator для Java можно использовать для такой задачи:

* Модерация текста
* Модерация изображений

[Справочная документация](/java/api/overview/azure/cognitiveservices/client/contentmoderator) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cognitiveservices/ms-azure-cs-contentmoderator) |[Артефакт (Maven)](https://mvnrepository.com/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-contentmoderator) | [Примеры](/samples/browse/?products=azure&term=content-moderator)

## <a name="prerequisites"></a>Предварительные требования

* подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Текущая версия [пакета средств разработки Java (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html).
* [Средство сборки Gradle](https://gradle.org/install/) или другой диспетчер зависимостей.
* Получив подписку Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesContentModerator"  title="Создание ресурса Content Moderator"  target="_blank">создайте ресурс Content Moderator </a> на портале Azure, чтобы получить ключ и конечную точку. Дождитесь, пока закончится развертывание, и нажмите кнопку **Перейти к ресурсу**.
    * Для подключения приложения к Content Moderator потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
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

## <a name="install-the-client-library"></a>Установка клиентской библиотеки

Найдите файл *build.gradle.kts* и откройте его в предпочитаемой интегрированной среде разработки или текстовом редакторе. Затем скопируйте и вставьте в файл приведенную ниже конфигурацию сборки. Эта конфигурация определяет проект как приложение Java, точкой входа которого является класс **ContentModeratorQuickstart**. Она импортирует клиентскую библиотеку для Content Moderator, а также пакет SDK для GSON для сериализации JSON.

```kotlin
plugins {
    java
    application
}

application{ 
    mainClassName = "ContentModeratorQuickstart"
}

repositories{
    mavenCentral()
}

dependencies{
    compile(group = "com.microsoft.azure.cognitiveservices", name = "azure-cognitiveservices-contentmoderator", version = "1.0.2-beta")
    compile(group = "com.google.code.gson", name = "gson", version = "2.8.5")
}
```

### <a name="create-a-java-file"></a>Создание файла Java


В рабочей папке выполните следующую команду, чтобы создать исходную папку проекта.

```console
mkdir -p src/main/java
```

Перейдите в новую папку и создайте файл с именем *ContentModeratorQuickstart.java*. Откройте его в предпочитаемом редакторе или интегрированной среде разработки и добавьте следующие операторы `import`:

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imports)]

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java), где размещены примеры кода для этого краткого руководства.

В классе приложения **ContentModeratorQuickstart** создайте переменные для ключа и конечной точки вашего ресурса.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_creds)]

> [!IMPORTANT]
> Перейдите на портал Azure. Если ресурс Content Moderator, созданный в соответствии с указаниями в разделе **Предварительные требования**, успешно развернут, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключ и конечная точка располагаются на странице **ключа и конечной точки** ресурса в разделе **управления ресурсами**. 
>
> Не забудьте удалить ключ из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Дополнительные сведения см. в статье о [безопасности в Cognitive Services](../../../cognitive-services-security.md).

В методе **main** приложения добавьте вызовы методов, используемых в этом кратком руководстве. Они будут определены позже.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_maincalls)]


## <a name="object-model"></a>Объектная модель

Следующие классы обрабатывают некоторые основные функции клиентской библиотеки Content Moderator для Java.

|Имя|Описание|
|---|---|
|[ContentModeratorClient](/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.contentmoderatorclient)|Этот класс требуется для всех функций Content Moderator. Вы создаете его экземпляр с информацией о подписке и используете его для создания экземпляров других классов.|
|[ImageModeration](/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.imagemoderations)|Этот класс предоставляет функции для анализа изображений на наличие содержимого для взрослых, личной информации или лиц людей.|
|[TextModerations](/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.textmoderations)|Этот класс предоставляет функции для анализа текста с целью определения языка, наличия ненормативной лексики, ошибок и личной информации.|
|[Проверки](/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.reviews)|Этот класс предоставляет функциональные возможности API-интерфейсов проверки, в том числе методы создания заданий, настраиваемых рабочих процессов и пользовательских проверок.|


## <a name="code-examples"></a>Примеры кода

Эти фрагменты кода показывают, как выполнить следующие действия с помощью клиентской библиотеки Content Moderator для Java:

* [аутентификация клиента](#authenticate-the-client);
* [модерация текста](#moderate-text);
* [модерация изображений](#moderate-images);


## <a name="authenticate-the-client"></a>Аутентификация клиента

В методе `main` приложения создайте объект [ContentModeratorClient](/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.contentmoderatorclient), используя значение конечной точки подписки и ключа подписки.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_client)]



## <a name="moderate-text"></a>Модерация текста

### <a name="set-up-sample-text"></a>Настройка примера текста

В верхней части класса **ContentModeratorQuickstart** определите ссылку на локальный текстовый файл. Добавьте TXT-файл в каталог проекта и введите текст, который необходимо проанализировать.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_textmod_var)]

### <a name="analyze-text"></a>Анализ текста

Создайте метод, считывающий TXT-файл и вызывающий метод **screenText** в каждой строке.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_textmod)]

### <a name="print-text-moderation-results"></a>Печать результатов модерации текста

Добавьте следующий код, чтобы вывести результаты модерации в JSON-файл в каталоге проекта.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_textmod_print)]

Закройте операторы `try` и `catch` для завершения метода.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_textmod_catch)]


## <a name="moderate-images"></a>Модерация изображений

### <a name="set-up-sample-image"></a>Настройка примера изображения

В новом методе создайте объект **[BodyModelModel](/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.models.bodymodelmodel)** с заданной строкой URL-адреса, указывающей на изображение.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod)]


### <a name="define-helper-class"></a>Определение вспомогательного класса

Затем в файле *ContentModeratorQuickstart.java* добавьте следующее определение класса в класс **ContentModeratorQuickstart**. Этот внутренний класс используется в процессе модерации изображений.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_evaluationdata)]


### <a name="analyze-content"></a>Анализ содержимого
Эта строка кода проверяет изображение по указанному URL-адресу на предмет содержимого для взрослых и непристойного характера. Описание такого содержимого см. в концептуальном руководстве по модерации изображений.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_ar)]

### <a name="check-for-text"></a>Проверка на наличие текста
Эта строка кода проверяет изображение на наличие видимого текста.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_text)]

### <a name="check-for-faces"></a>Проверка на наличие лиц
Эта строка кода проверяет изображение на наличие видимых лиц людей.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_faces)]

Наконец, сохраните возвращенную информацию в списке `EvaluationData`.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_storedata)]

### <a name="print-results"></a>Вывод результатов

После цикла `while` добавьте следующий код, который выводит результаты на консоль и в выходной файл *src/main/resources/ModerationOutput.json*.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_printdata)]

Закройте оператор `try` и добавьте оператор `catch` для завершения метода.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_catch)]

## <a name="run-the-application"></a>Выполнение приложения

Чтобы создать приложение, выполните следующую команду:

```console
gradle build
```

Запустите приложение, выполнив команду `gradle run`:

```console
gradle run
```

Затем перейдите к файлу *src/main/resources/ModerationOutput.json* и просмотрите результаты модерации вашего содержимого.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку Cognitive Services, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней.

* [Портал](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы узнали, как использовать библиотеку Java для Content Moderator для выполнения задач модерации. Теперь узнайте больше о модерации изображений или другого мультимедиа, прочитав концептуальное руководство.

> [!div class="nextstepaction"]
> [Принципы модерации изображений](../../image-moderation-api.md)
