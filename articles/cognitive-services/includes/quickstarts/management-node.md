---
title: Краткое руководство. Клиентская библиотека управления Azure для Node.js
description: В этом кратком руководстве описано, как приступить к работе с клиентской библиотекой управления Azure для Node.js.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 3/22/2021
ms.author: pafarley
ms.openlocfilehash: 41f6c8e260968eacd04249b3f887d4865907df0d
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104879690"
---
[Справочная документация](/javascript/api/@azure/arm-cognitiveservices/) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/arm-cognitiveservices) | [Пакет (NPM)](https://www.npmjs.com/package/@azure/arm-cognitiveservices) | [Примеры](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/arm-cognitiveservices#sample-code)

## <a name="javascript-prerequisites"></a>Предварительные требования JavaScript

* Действующая подписка Azure ([создайте бесплатную учетную запись](https://azure.microsoft.com/free/)).
* Текущая версия [Node.js](https://nodejs.org/)

[!INCLUDE [Create a service principal](./create-service-principal.md)]

[!INCLUDE [Create a resource group](./create-resource-group.md)]

## <a name="create-a-new-nodejs-application"></a>создание приложения Node.js;

В окне консоли (например, cmd, PowerShell или Bash) создайте новый каталог для приложения и перейдите в него. 

```console
mkdir myapp && cd myapp
```

Выполните команду `npm init`, чтобы создать приложение узла с помощью файла `package.json`. 

```console
npm init
```

Прежде чем начать, создайте файл с именем _index.js_.

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

Установите следующие пакеты npm:

```console
npm install @azure/arm-cognitiveservices
npm install @azure/ms-rest-js
npm install @azure/ms-rest-nodeauth
```

Файл `package.json` этого приложения будет дополнен зависимостями.

### <a name="import-libraries"></a>Импорт библиотек

Откройте скрипт _index.js_ и импортируйте следующие библиотеки.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/azure_management_service/create_delete_resource.js?name=snippet_imports)]

## <a name="authenticate-the-client"></a>Аутентификация клиента

Добавьте следующие поля в корень скрипта и заполните их значения с помощью созданного субъекта-службы и информации учетной записи Azure.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/azure_management_service/create_delete_resource.js?name=snippet_constants)]

Затем добавьте следующую функцию `quickstart` для выполнения основных действий программы. Первый блок кода конструирует объект **CognitiveServicesManagementClient**, используя переменные учетных данных, указанные выше. Этот объект необходим для всех операций управления Azure.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/azure_management_service/create_delete_resource.js?name=snippet_main_auth)]

## <a name="call-management-functions"></a>Вызов функций управления

Добавьте следующий код в конец функции `quickstart`, чтобы получить список доступных ресурсов, создать пример ресурса, вывести список собственных ресурсов, а затем удалить пример ресурса. Эти функции будут определены позже.

## <a name="create-a-cognitive-services-resource-nodejs"></a>Создание ресурса Cognitive Services (Node.js)

Чтобы создать ресурс Cognitive Services и подписываться на него, используйте функцию **Create**. Эта функция добавляет новый оплачиваемый ресурс в передаваемую группу ресурсов. При создании ресурса необходимо знать, какой вид службы вы хотите использовать, а также выбрать ценовую категорию (или номер SKU) и расположение Azure. Следующая функция использует все эти аргументы и создает ресурс.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/azure_management_service/create_delete_resource.js?name=snippet_create)]

### <a name="choose-a-service-and-pricing-tier"></a>Выбор службы и ценовой категории

При создании ресурса необходимо знать, какой вид службы вы хотите использовать, а также выбрать [ценовую категорию](https://azure.microsoft.com/pricing/details/cognitive-services/) (или номер SKU). Вы будете использовать эту и другую информацию в качестве параметров при создании ресурса. Следующая функция выводит список доступных "видов" служб Cognitive Services.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/azure_management_service/create_delete_resource.js?name=snippet_list_avail)]

[!INCLUDE [cognitive-services-subscription-types](../../../../includes/cognitive-services-subscription-types.md)]

[!INCLUDE [SKUs and pricing](./sku-pricing.md)]

## <a name="view-your-resources"></a>Просмотр ресурсов

Чтобы просмотреть все ресурсы в учетной записи Azure (для всех групп ресурсов), используйте следующую функцию:

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/azure_management_service/create_delete_resource.js?name=snippet_list)]

## <a name="delete-a-resource"></a>Удаление ресурса

Следующая функция удаляет указанные ресурсы из заданной группы ресурсов.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/azure_management_service/create_delete_resource.js?name=snippet_delete)]

## <a name="run-the-application"></a>Выполнение приложения

Добавьте следующий код в нижнюю часть скрипта, чтобы вызывать основную функцию `quickstart` при обработке ошибок.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/azure_management_service/create_delete_resource.js?name=snippet_main)]

Затем в окне консоли запустите приложение с помощью команды `node`.

```console
node index.js
```

## <a name="see-also"></a>См. также раздел

* Сведения о безопасной работе с Cognitive Services см. в статье **[Проверка подлинности запросов к Azure Cognitive Services](../../authentication.md)** .
* См. статью **[Общие сведения об Azure Cognitive Services](../../what-are-cognitive-services.md)** , в которой можно увидеть список различных категорий в Cognitive Services.
* Список естественных языков, поддерживаемых Cognitive Services, см. в статье **[Поддержка естественного языка](../../language-support.md)** .
* Сведения об использовании Cognitive Services в локальной среде см. в статье **[Использование Cognitive Services в качестве контейнеров](../../cognitive-services-container-support.md)** .
* См. статью **[Планирование затрат и управление ими в Cognitive Services](../../plan-manage-costs.md)** , чтобы получить сведения о том, как оценивать затраты на использование Cognitive Services.
* Дополнительные сведения о пакете SDK для управления см. в **[справочной документации по пакету SDK для управления Azure](/javascript/api/@azure/arm-cognitiveservices/)** .
