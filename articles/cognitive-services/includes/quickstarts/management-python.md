---
title: Краткое руководство. Клиентская библиотека управления Azure для Python
description: В этом кратком руководстве описано, как приступить к работе с клиентской библиотекой управления Azure для Python.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 08/05/2020
ms.author: pafarley
ms.openlocfilehash: fb908cdcf3e235654effc043de29e599a48179d4
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104879710"
---
[Справочная документация](/python/api/azure-mgmt-cognitiveservices/azure.mgmt.cognitiveservices) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-mgmt-cognitiveservices) | [Пакет (PyPi)](https://pypi.org/project/azure-mgmt-cognitiveservices/) | [Примеры](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-mgmt-cognitiveservices/tests)

## <a name="python-prerequisites"></a>Предварительные требования Python

* Действующая подписка Azure ([создайте бесплатную учетную запись](https://azure.microsoft.com/free/)).
* [Python 3.x](https://www.python.org/)

[!INCLUDE [Create a service principal](./create-service-principal.md)]

[!INCLUDE [Create a resource group](./create-resource-group.md)]

## <a name="create-a-new-python-application"></a>Создание приложения Python

Создайте новое приложение Python в предпочитаемом редакторе или интегрированной среде разработки и перейдите к своему проекту в окне консоли.

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

Клиентскую библиотеку можно установить с помощью следующей команды:

```console
pip install azure-mgmt-cognitiveservices
```

Если вы используете интегрированную среду разработки Visual Studio, клиентская библиотека доступна в виде загружаемого пакета NuGet.

### <a name="import-libraries"></a>Импорт библиотек

Откройте скрипт Python и импортируйте следующие библиотеки.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_imports)]

## <a name="authenticate-the-client"></a>Аутентификация клиента

Добавьте следующие поля в корень скрипта и заполните их значения с помощью созданного субъекта-службы и информации учетной записи Azure.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_constants)]

Затем добавьте следующий код для создания объекта **CognitiveServicesManagementClient**. Этот объект необходим для всех операций управления Azure.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_auth)]

## <a name="create-a-cognitive-services-resource-python"></a>Создание ресурса Cognitive Services (Python)

Чтобы создать ресурс Cognitive Services и подписываться на него, используйте функцию **Create**. Эта функция добавляет новый оплачиваемый ресурс в передаваемую группу ресурсов. При создании ресурса необходимо знать, какой вид службы вы хотите использовать, а также выбрать ценовую категорию (или номер SKU) и расположение Azure. Следующая функция использует все эти аргументы и создает ресурс.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_create)]

### <a name="choose-a-service-and-pricing-tier"></a>Выбор службы и ценовой категории

При создании ресурса необходимо знать, какой вид службы вы хотите использовать, а также выбрать [ценовую категорию](https://azure.microsoft.com/pricing/details/cognitive-services/) (или номер SKU). Вы будете использовать эту и другую информацию в качестве параметров при создании ресурса. Следующая функция выводит список доступных "видов" служб Cognitive Services.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_list_avail)]

[!INCLUDE [cognitive-services-subscription-types](../../../../includes/cognitive-services-subscription-types.md)]

[!INCLUDE [SKUs and pricing](./sku-pricing.md)]

## <a name="view-your-resources"></a>Просмотр ресурсов

Чтобы просмотреть все ресурсы в учетной записи Azure (для всех групп ресурсов), используйте следующую функцию:

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_list)]

## <a name="delete-a-resource"></a>Удаление ресурса

Следующая функция удаляет указанные ресурсы из заданной группы ресурсов.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_delete)]

## <a name="call-management-functions"></a>Вызов функций управления

Добавьте следующий код в нижнюю часть скрипта, чтобы вызывать указанные выше функции. Этот код перечисляет доступные ресурсы, создает пример ресурса, перечисляет собственные ресурсы, а затем удаляет пример ресурса.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_calls)]

## <a name="run-the-application"></a>Выполнение приложения

Запустите приложение из командной строки с помощью команды `python`.

```console
python <your-script-name>.py
```

## <a name="see-also"></a>См. также раздел

* Сведения о безопасной работе с Cognitive Services см. в статье **[Проверка подлинности запросов к Azure Cognitive Services](../../authentication.md)** .
* См. статью **[Общие сведения об Azure Cognitive Services](../../what-are-cognitive-services.md)** , в которой можно увидеть список различных категорий в Cognitive Services.
* Список естественных языков, поддерживаемых Cognitive Services, см. в статье **[Поддержка естественного языка](../../language-support.md)** .
* Сведения об использовании Cognitive Services в локальной среде см. в статье **[Использование Cognitive Services в качестве контейнеров](../../cognitive-services-container-support.md)** .
* См. статью **[Планирование затрат и управление ими в Cognitive Services](../../plan-manage-costs.md)** , чтобы получить сведения о том, как оценивать затраты на использование Cognitive Services.
* Дополнительные сведения о пакете SDK для управления см. в **[справочной документации по пакету SDK для управления Azure](/python/api/azure-mgmt-cognitiveservices/azure.mgmt.cognitiveservices)** .
