---
title: Развертывание веб-приложения с помощью шаблона в Azure Cosmos DB
description: Узнайте, как развернуть учетную запись Azure Cosmos, веб-приложения службы приложений Azure и пример веб-приложения с помощью шаблона Azure Resource Manager.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 06/19/2020
ms.author: mjbrown
ms.openlocfilehash: 55d58a6c4724bd01325db029ed75d77ccc96d0f8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93333585"
---
# <a name="deploy-azure-cosmos-db-and-azure-app-service-with-a-web-app-from-github-using-an-azure-resource-manager-template"></a>Развертывание Azure Cosmos DB и службы приложений Azure с помощью веб-приложения из GitHub с использованием шаблона Azure Resource Manager
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В этом руководстве показано, как выполнить развертывание веб-приложения без касания, которое подключается к Azure Cosmos DB при первом запуске без необходимости вырезания и вставки сведений о подключении из Azure Cosmos DB в `appsettings.json` Параметры приложения служб приложений Azure в портал Azure. Все эти действия выполняются с помощью шаблона Azure Resource Manager в одной операции. В этом примере мы развернем [Azure Cosmos DB TODO Sample](https://github.com/Azure-Samples/cosmos-dotnet-core-todo-app) из [учебника по веб-приложению](sql-api-dotnet-application.md).

Диспетчер ресурсов шаблоны являются достаточно гибкими и позволяют создавать сложные развертывания в любой службе в Azure. Сюда входят такие расширенные задачи, как развертывание приложений из GitHub и Добавление сведений о подключении в параметры приложения службы приложений Azure в портал Azure. В этом руководстве показано, как выполнить следующие действия с помощью одного шаблона диспетчер ресурсов.

* Развертывание учетной записи Azure Cosmos.
* Разверните план размещения службы приложений Azure.
* Разверните службу приложений Azure.
* Вставьте конечную точку и ключи из учетной записи Azure Cosmos в параметры приложения службы приложений в портал Azure.
* Разверните веб-приложение из репозитория GitHub в службе приложений.

Полученное развертывание имеет полностью функциональное веб-приложение, которое может подключаться к Azure Cosmos DB без необходимости вырезания и вставки URL-адреса конечной точки Azure Cosmos DB или ключей проверки подлинности из портал Azure.

## <a name="prerequisites"></a>Предварительные условия

> [!TIP]
> Хотя это руководство не требует наличия опыта работы с шаблонами Azure Resource Manager или JSON, если возникнет необходимость изменить ссылки шаблонов или варианты развертывания, потребуются знания в каждой из этих областей.

## <a name="step-1-deploy-the-template"></a>Шаг 1. Развертывание шаблона

Сначала нажмите кнопку **развернуть в Azure** , чтобы открыть портал Azure для создания пользовательского развертывания. Вы также можете просмотреть шаблон управления ресурсами Azure из [коллекции шаблонов](https://github.com/Azure/azure-quickstart-templates/tree/master/101-cosmosdb-webapp) быстрого запуска Azure.

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="Развертывание в Azure":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-cosmosdb-webapp%2Fazuredeploy.json)

В портал Azure выберите подписку для развертывания, а затем выберите или создайте новую группу ресурсов. Затем введите следующие значения.

:::image type="content" source="./media/create-website/template-deployment.png" alt-text="Снимок экрана пользовательского интерфейса развертывания шаблона":::

* **Регион** — это требуется для Диспетчер ресурсов. Введите тот же регион, который использовался параметром location, в котором находятся ресурсы.
* **Имя приложения** — это имя используется всеми ресурсами для этого развертывания. Обязательно выберите уникальное имя, чтобы избежать конфликтов с существующими Azure Cosmos DB и учетными записями службы приложений.
* **Location** — регион, в котором развернуты ресурсы.
* **Уровень плана службы приложений** — ценовая категория плана службы приложений.
* **Экземпляры плана службы приложений** — количество рабочих ролей для плана службы приложений.
* **URL-адрес репозитория** — репозиторий в веб-приложение на GitHub.
* **Branch** — ветвь для репозитория GitHub.
* **Имя базы данных** — имя базы данных Azure Cosmos.
* **Имя контейнера** — имя контейнера Azure Cosmos.

После заполнения значений нажмите кнопку **создать** , чтобы начать развертывание. Выполнение этого шага займет от 5 до 10 минут.

> [!TIP]
> Шаблон не проверяет, являются ли имя службы приложений Azure и имя учетной записи Azure Cosmos, указанные в шаблоне, допустимыми и доступными. Перед началом развертывания настоятельно рекомендуется проверить доступность имен, которые планируется использовать.


## <a name="step-2-explore-the-resources"></a>Шаг 2. изучение ресурсов

### <a name="view-the-deployed-resources"></a>Просмотр развернутых ресурсов

После развертывания ресурсов в шаблоне можно увидеть каждый из них в группе ресурсов.

:::image type="content" source="./media/create-website/resource-group.png" alt-text="Группа ресурсов":::

### <a name="view-cosmos-db-endpoint-and-keys"></a>Просмотр Cosmos DB конечных точек и ключей

Затем откройте учетную запись Azure Cosmos на портале. На следующем снимке экрана показаны конечные точки и ключи для учетной записи Azure Cosmos.

:::image type="content" source="./media/create-website/cosmos-keys.png" alt-text="Ключи Cosmos":::

### <a name="view-the-azure-cosmos-db-keys-in-application-settings"></a>Просмотр ключей Azure Cosmos DB в параметрах приложения

Затем перейдите к службе приложений Azure в группе ресурсов. Перейдите на вкладку Конфигурация, чтобы просмотреть параметры приложения для службы приложений. Параметры приложения содержат учетную запись Cosmos DB и значения первичного ключа, необходимые для подключения к Cosmos DB, а также имена базы данных и контейнера, переданные из развертывания шаблона.

:::image type="content" source="./media/create-website/application-settings.png" alt-text="Параметры приложения":::

### <a name="view-web-app-in-deployment-center"></a>Просмотр веб-приложения в центре развертывания

Далее перейдите в центр развертывания для службы приложений. Здесь вы увидите, что репозитории указывают на репозиторий GitHub, переданный в шаблон. Кроме того, приведенное ниже состояние указывает на успешное выполнение (активное), означающее, что приложение успешно развернуто и запущено.

:::image type="content" source="./media/create-website/deployment-center.png" alt-text="Центр развертывания":::

### <a name="run-the-web-application"></a>Запуск веб-приложения

Щелкните **Обзор** в верхней части центра развертывания, чтобы открыть веб-приложение. Веб-приложение откроется на начальном экране. Щелкните **создать** и введите данные в поля и нажмите кнопку Сохранить. На полученном экране отображаются данные, сохраненные в Cosmos DB.

:::image type="content" source="./media/create-website/app-home-screen.png" alt-text="Главная страница":::

## <a name="step-3-how-does-it-work"></a>Шаг 3. как это работает

Существует три элемента, которые необходимы для работы.

### <a name="reading-app-settings-at-runtime"></a>Чтение параметров приложения во время выполнения

Сначала приложению необходимо запросить конечную точку и ключ Cosmos DB в `Startup` классе в веб-приложении MVC ASP.NET. [Пример Cosmos DB to do](https://github.com/Azure-Samples/cosmos-dotnet-core-todo-app) можно запустить локально, где можно ввести сведения о подключении в appsettings.js. Однако при развертывании этот файл развертывается вместе с приложением. Если эти линии красными не могут получить доступ к параметрам, указанным в appsettings.js, то будет предпринята попытка использовать параметры приложения в службе приложений Azure.

:::image type="content" source="./media/create-website/startup.png" alt-text="На снимке экрана показан метод с несколькими строковыми переменными, отмеченными красным, включая databaseName, containerName, Account и Key.":::

### <a name="using-special-azure-resource-management-functions"></a>Использование специальных функций управления ресурсами Azure

Чтобы эти значения были доступны приложению при развертывании, шаблон Azure Resource Manager может запросить эти значения из учетной записи Cosmos DB с помощью специальных функций управления ресурсами Azure, включая [Reference](../azure-resource-manager/templates/template-functions-resource.md#reference) и [listKeys](../azure-resource-manager/templates/template-functions-resource.md#listkeys) , которые заключают значения из учетной записи Cosmos DB и вставляют их в значения параметров приложения с именами ключей, совпадающими с тем, что используется в приложении выше в формате "{Section: Key}". Например, `CosmosDb:Account`.

:::image type="content" source="./media/create-website/template-keys.png" alt-text="Ключи шаблона":::

### <a name="deploying-web-apps-from-github"></a>Развертывание веб-приложений из GitHub

Наконец, необходимо развернуть веб-приложение из GitHub в службе приложений. Это делается с помощью приведенного ниже JSON. Есть две вещи, с которыми следует обратить внимание, — это тип и имя для этого ресурса. `"type": "sourcecontrols"` `"name": "web"` Значения свойств и являются жестко запрограммированными и не должны изменяться.

:::image type="content" source="./media/create-website/deploy-from-github.png" alt-text="Развертывание из GitHub":::

## <a name="next-steps"></a>Дальнейшие действия

Поздравляем! Вы развернули Azure Cosmos DB, службу приложений Azure и пример веб-приложения, которое автоматически имеет сведения о соединении, необходимые для подключения к Cosmos DB, всего за одну операцию и без необходимости вырезания и вставки конфиденциальной информации. Используя этот шаблон в качестве отправной точки, вы можете изменить его для развертывания собственных веб-приложений таким же образом.

* Шаблон Azure Resource Manager для этого примера см. в [коллекции шаблонов](https://github.com/Azure/azure-quickstart-templates/tree/master/101-cosmosdb-webapp) быстрого запуска Azure.
* Исходный код для примера приложения переходит в [Cosmos DB для приложения на GitHub](https://github.com/Azure-Samples/cosmos-dotnet-core-todo-app).
