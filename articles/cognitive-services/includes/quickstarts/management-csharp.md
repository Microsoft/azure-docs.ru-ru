---
title: Краткое руководство. Клиентская библиотека управления Azure для .NET
description: В этом кратком руководстве описано, как приступить к работе с клиентской библиотекой управления Azure для .NET.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 3/22/2021
ms.author: pafarley
ms.openlocfilehash: 1ecfd58e961e70182bfc2e0cc8eaf77f5053eb77
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104879775"
---
[Справочная документация](/dotnet/api/overview/azure/cognitiveservices/management) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Microsoft.Azure.Management.CognitiveServices) | [Пакет (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.Management.CognitiveServices/) | [Примеры](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Microsoft.Azure.Management.CognitiveServices/tests)

## <a name="c-prerequisites"></a>Предварительные требования C#

* Действующая подписка Azure ([создайте бесплатную учетную запись](https://azure.microsoft.com/free/)).
* Текущая версия [.NET Core](https://dotnet.microsoft.com/download/dotnet-core).

[!INCLUDE [Create a service principal](./create-service-principal.md)]

[!INCLUDE [Create a resource group](./create-resource-group.md)]

## <a name="create-a-new-c-application"></a>Создание нового приложения C#

Создайте приложение .NET Core. В окне консоли (cmd, PowerShell или Bash) выполните команду `dotnet new`, чтобы создать консольное приложение с именем `azure-management-quickstart`. Эта команда создает простой проект "Hello World" на языке C# с одним файлом исходного кода: *program.cs*. 

```console
dotnet new console -n azure-management-quickstart
```

Измените каталог на созданную папку приложения. Чтобы создать приложение, выполните следующую команду:

```console
dotnet build
```

Выходные данные сборки не должны содержать предупреждений или ошибок. 

```console
...
Build succeeded.
 0 Warning(s)
 0 Error(s)
...
```

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

В каталоге приложения установите клиентскую библиотеку управления Azure для .NET, используя следующую команду:

```console
dotnet add package Microsoft.Azure.Management.CognitiveServices
dotnet add package Microsoft.Azure.Management.Fluent
dotnet add package Microsoft.Azure.Management.ResourceManager.Fluent
```

Если вы используете интегрированную среду разработки Visual Studio, клиентская библиотека доступна в виде загружаемого пакета NuGet.

### <a name="import-libraries"></a>Импорт библиотек

Откройте файл *program.cs* и добавьте следующие операторы `using` в начало файла:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/azure_management_service/create_delete_resource.cs?name=snippet_using)]

## <a name="authenticate-the-client"></a>Аутентификация клиента

Добавьте следующие поля в корень файла *program.cs* и заполните их значения с помощью созданного субъекта-службы и информации учетной записи Azure.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/azure_management_service/create_delete_resource.cs?name=snippet_constants)]

Затем в методе **Main** используйте эти значения для создания объекта **CognitiveServicesManagementClient**. Этот объект необходим для всех операций управления Azure.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/azure_management_service/create_delete_resource.cs?name=snippet_assigns)]

## <a name="call-management-methods"></a>Вызов методов управления

Добавьте следующий код в метод **Main**, чтобы получить список доступных ресурсов, создать пример ресурса, вывести список собственных ресурсов, а затем удалить пример ресурса. Эти методы будут определены позже.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/azure_management_service/create_delete_resource.cs?name=snippet_calls)]

## <a name="create-a-cognitive-services-resource-c"></a>Создание ресурса Cognitive Services (C#)

Чтобы создать ресурс Cognitive Services и подписаться на него, используйте метод **Create**. Этот метод добавляет новый оплачиваемый ресурс в передаваемую группу ресурсов. При создании ресурса необходимо знать, какой вид службы вы хотите использовать, а также выбрать ценовую категорию (или номер SKU) и расположение Azure. Следующий метод использует все эти аргументы и создает ресурс.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/azure_management_service/create_delete_resource.cs?name=snippet_create)]

### <a name="choose-a-service-and-pricing-tier"></a>Выбор службы и ценовой категории

При создании ресурса необходимо знать, какой вид службы вы хотите использовать, а также выбрать [ценовую категорию](https://azure.microsoft.com/pricing/details/cognitive-services/) (или номер SKU). Вы будете использовать эту и другую информацию в качестве параметров при создании ресурса. Чтобы получить список доступных видов служб Cognitive Service, вызовите в скрипте следующий метод:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/azure_management_service/create_delete_resource.cs?name=snippet_list_avail)]

[!INCLUDE [cognitive-services-subscription-types](../../../../includes/cognitive-services-subscription-types.md)]

[!INCLUDE [SKUs and pricing](./sku-pricing.md)]

## <a name="view-your-resources"></a>Просмотр ресурсов

Чтобы просмотреть все ресурсы в учетной записи Azure (для всех групп ресурсов), используйте следующий метод:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/azure_management_service/create_delete_resource.cs?name=snippet_list)]

## <a name="delete-a-resource"></a>Удаление ресурса

Следующий метод удаляет указанные ресурсы из заданной группы ресурсов.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/azure_management_service/create_delete_resource.cs?name=snippet_delete)]

## <a name="run-the-application"></a>Выполнение приложения

Запустите приложение из каталога приложения с помощью команды `dotnet run`.

```dotnet
dotnet run
```

## <a name="see-also"></a>См. также раздел

* Сведения о безопасной работе с Cognitive Services см. в статье **[Проверка подлинности запросов к Azure Cognitive Services](../../authentication.md)** .
* См. статью **[Общие сведения об Azure Cognitive Services](../../what-are-cognitive-services.md)** , в которой можно увидеть список различных категорий в Cognitive Services.
* Список естественных языков, поддерживаемых Cognitive Services, см. в статье **[Поддержка естественного языка](../../language-support.md)** .
* Сведения об использовании Cognitive Services в локальной среде см. в статье **[Использование Cognitive Services в качестве контейнеров](../../cognitive-services-container-support.md)** .
* См. статью **[Планирование затрат и управление ими в Cognitive Services](../../plan-manage-costs.md)** , чтобы получить сведения о том, как оценивать затраты на использование Cognitive Services.
* Дополнительные сведения о пакете SDK для управления см. в **[справочной документации по пакету SDK для управления Cognitive Services](/dotnet/api/overview/azure/cognitiveservices/management)** .
