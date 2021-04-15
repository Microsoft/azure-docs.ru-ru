---
title: Краткое руководство. Клиентская библиотека управления Azure для Java
description: В этом кратком руководстве описано, как приступить к работе с клиентской библиотекой управления Azure для Java.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 3/22/2021
ms.author: pafarley
ms.openlocfilehash: 4c0d4dd1a834e42a75da5199b7aaed0e123f8e63
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104879785"
---
[Справочная документация](/java/api/com.microsoft.azure.management.cognitiveservices) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cognitiveservices/mgmt-v2017_04_18/src/main/java/com/microsoft/azure/management/cognitiveservices/v2017_04_18) | [Пакет (Maven)](https://mvnrepository.com/artifact/com.microsoft.azure/azure-mgmt-cognitiveservices).

## <a name="java-prerequisites"></a>Предварительные требования Java

* Действующая подписка Azure ([создайте бесплатную учетную запись](https://azure.microsoft.com/free/)).
* Текущая версия [пакета средств разработки Java (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html).
* [Средство сборки Gradle](https://gradle.org/install/) или другой диспетчер зависимостей.


[!INCLUDE [Create a service principal](./create-service-principal.md)]

[!INCLUDE [Create a resource group](./create-resource-group.md)]

## <a name="create-a-new-java-application"></a>Создание нового приложения Java

В окне консоли (например, cmd, PowerShell или Bash) создайте новый каталог для приложения и перейдите в него. 

```console
mkdir myapp && cd myapp
```

Выполните команду `gradle init` из рабочей папки. Эта команда создает необходимые файлы сборки для Gradle, включая *build.gradle.kts*, который используется во время выполнения для создания и настройки приложения.

```console
gradle init --type basic
```

Когда появится запрос на выбор **предметно-ориентированного языка**, выберите **Kotlin**.

Выполните следующие команды из рабочего каталога:

```console
mkdir -p src/main/java
```

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

В этом кратком руководстве используется диспетчер зависимостей Gradle. Клиентскую библиотеку и информацию для других диспетчеров зависимостей можно найти в [центральном репозитории Maven](https://mvnrepository.com/artifact/com.azure/azure-ai-formrecognizer).

В файле проекта *build.gradle.kts* включите клиентскую библиотеку в качестве оператора `implementation` наряду с требуемыми подключаемыми модулями и параметрами.

```kotlin
plugins {
    java
    application
}
application {
    mainClass.set("FormRecognizer")
}
repositories {
    mavenCentral()
}
dependencies {
    implementation(group = "com.microsoft.azure", name = "azure-mgmt-cognitiveservices", version = "1.10.0-beta")
}
```

### <a name="import-libraries"></a>Импорт библиотек

Перейдите к новой папке **src/main/java** и создайте файл с именем *Management.java*. Откройте его в предпочитаемом редакторе или интегрированной среде разработки и добавьте следующие операторы `import`:

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_imports)]

## <a name="authenticate-the-client"></a>Аутентификация клиента

Добавьте класс в файл *Management.java*, а затем добавьте в него следующие поля со значениями. Заполните эти значения, используя созданный субъект-службу и информацию учетной записи Azure.

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_constants)]

Затем в методе **main** используйте эти значения для создания объекта **CognitiveServicesManager**. Этот объект необходим для всех операций управления Azure.

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_auth)]

## <a name="call-management-methods"></a>Вызов методов управления

Добавьте следующий код в метод **Main**, чтобы получить список доступных ресурсов, создать пример ресурса, вывести список собственных ресурсов, а затем удалить пример ресурса. Эти методы будут определены позже.

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_calls)]

## <a name="create-a-cognitive-services-resource-java"></a>Создание ресурса Cognitive Services (Java)

Чтобы создать ресурс Cognitive Services и подписаться на него, используйте метод **create**. Этот метод добавляет новый оплачиваемый ресурс в передаваемую группу ресурсов. При создании ресурса необходимо знать, какой вид службы вы хотите использовать, а также выбрать ценовую категорию (или номер SKU) и расположение Azure. Следующий метод использует все эти аргументы и создает ресурс.

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_create)]

### <a name="choose-a-service-and-pricing-tier"></a>Выбор службы и ценовой категории

При создании ресурса необходимо знать, какой вид службы вы хотите использовать, а также выбрать [ценовую категорию](https://azure.microsoft.com/pricing/details/cognitive-services/) (или номер SKU). Вы будете использовать эту и другую информацию в качестве параметров при создании ресурса. Чтобы получить список доступных видов служб Cognitive Service, вызовите следующий метод:

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_list_avail)]

[!INCLUDE [cognitive-services-subscription-types](../../../../includes/cognitive-services-subscription-types.md)]

[!INCLUDE [SKUs and pricing](./sku-pricing.md)]

## <a name="view-your-resources"></a>Просмотр ресурсов

Чтобы просмотреть все ресурсы в учетной записи Azure (для всех групп ресурсов), используйте следующий метод:

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_list)]

## <a name="delete-a-resource"></a>Удаление ресурса

Следующий метод удаляет указанные ресурсы из заданной группы ресурсов.

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_delete)]

## <a name="see-also"></a>См. также раздел

* Сведения о безопасной работе с Cognitive Services см. в статье **[Проверка подлинности запросов к Azure Cognitive Services](../../authentication.md)** .
* См. статью **[Общие сведения об Azure Cognitive Services](../../what-are-cognitive-services.md)** , в которой можно увидеть список различных категорий в Cognitive Services.
* Список естественных языков, поддерживаемых Cognitive Services, см. в статье **[Поддержка естественного языка](../../language-support.md)** .
* Сведения об использовании Cognitive Services в локальной среде см. в статье **[Использование Cognitive Services в качестве контейнеров](../../cognitive-services-container-support.md)** .
* См. статью **[Планирование затрат и управление ими в Cognitive Services](../../plan-manage-costs.md)** , чтобы получить сведения о том, как оценивать затраты на использование Cognitive Services.
* Дополнительные сведения о пакете SDK для управления см. в **[справочной документации по пакету SDK для управления Azure](/java/api/com.microsoft.azure.management.cognitiveservices)** .
