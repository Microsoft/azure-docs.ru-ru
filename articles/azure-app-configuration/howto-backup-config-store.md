---
title: Автоматическое резервное копирование значений ключей из хранилищ конфигурации приложений Azure
description: Узнайте, как настроить автоматическую архивацию значений ключей между хранилищами конфигурации приложений.
services: azure-app-configuration
author: avanigupta
ms.assetid: ''
ms.service: azure-app-configuration
ms.devlang: csharp
ms.custom: devx-track-dotnet, devx-track-azurecli
ms.topic: how-to
ms.date: 04/27/2020
ms.author: avgupta
ms.openlocfilehash: b3e0bcad7beccc31e1772fbb24ffad7f502b8140
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102454249"
---
# <a name="back-up-app-configuration-stores-automatically"></a>Автоматическое резервное копирование хранилищ конфигурации приложений

В этой статье вы узнаете, как настроить автоматическую архивацию значений ключей из основного хранилища конфигурации приложений Azure в дополнительное хранилище. Автоматическая архивация использует интеграцию службы "Сетка событий Azure" с конфигурацией приложения. 

После настройки автоматического резервного копирования Конфигурация приложения будет публиковать события в службе "Сетка событий Azure" для любых изменений, внесенных в значения ключа в хранилище конфигураций. Служба "Сетка событий" поддерживает различные службы Azure, от которых пользователи могут подписываться на события, генерируемые при создании, обновлении или удалении значений ключей.

## <a name="overview"></a>Обзор

В этой статье вы будете использовать хранилище очередей Azure для получения событий из службы "Сетка событий" и использования триггера таймера для функций Azure для обработки событий в очереди в пакетах. 

При активации функции в зависимости от событий она выберет последние значения ключей, которые были изменены из хранилища конфигурации основного приложения, и соответствующим образом обновите хранилище-получатель. Эта программа установки помогает объединить несколько изменений, которые происходят в течение короткого периода в одной операции резервного копирования, что позволяет избежать чрезмерного выполнения запросов к хранилищам конфигурации приложений.

![Схема, на которой показана архитектура резервной копии хранилища конфигурации приложения.](./media/config-store-backup-architecture.png)

## <a name="resource-provisioning"></a>Подготовка ресурсов

Цель резервного копирования хранилищ конфигураций приложений заключается в использовании нескольких хранилищ конфигураций в разных регионах Azure для повышения географической устойчивости приложения. Для этого первичное и вторичное хранилища должны находиться в разных регионах Azure. Все остальные ресурсы, созданные в этом учебнике, можно подготовить в любом регионе по своему усмотрению. Это связано с тем, что если основной регион не работает, создать резервную копию ничего не получится, пока основной регион не будет доступен снова.

В этом руководстве вы создадите вторичное хранилище в `centralus` регионе и все остальные ресурсы в `westus` регионе.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)].

## <a name="prerequisites"></a>Предварительные условия 

- [Visual Studio 2019](https://visualstudio.microsoft.com/vs) с рабочей нагрузкой разработки Azure.

- [пакет SDK для .NET Core](https://dotnet.microsoft.com/download);

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Для работы с этим руководством требуется версия 2.3.1 или более поздняя Azure CLI. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure — это логическая коллекция, в которой выполняется развертывание и администрирование ресурсов Azure.

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group).

В следующем примере создается группа ресурсов с именем `<resource_group_name>` в расположении именем `westus`.  Замените `<resource_group_name>` уникальным именем для группы ресурсов.

```azurecli-interactive
resourceGroupName="<resource_group_name>"
az group create --name $resourceGroupName --location westus
```

## <a name="create-app-configuration-stores"></a>Создание хранилищ конфигурации приложений

Создайте хранилище конфигурации основного и дополнительного приложений в разных регионах.
Замените `<primary_appconfig_name>` и `<secondary_appconfig_name>` уникальными именами для хранилищ конфигураций. Имя каждого магазина должно быть уникальным, так как оно используется в качестве DNS-имени.

```azurecli-interactive
primaryAppConfigName="<primary_appconfig_name>"
secondaryAppConfigName="<secondary_appconfig_name>"
az appconfig create \
  --name $primaryAppConfigName \
  --location westus \
  --resource-group $resourceGroupName\
  --sku standard

az appconfig create \
  --name $secondaryAppConfigName \
  --location centralus \
  --resource-group $resourceGroupName\
  --sku standard
```

## <a name="create-a-queue"></a>Создание очереди

Создайте учетную запись хранения и очередь для получения событий, опубликованных службой "Сетка событий".

```azurecli-interactive
storageName="<unique_storage_name>"
queueName="<queue_name>"
az storage account create -n $storageName -g $resourceGroupName -l westus --sku Standard_LRS
az storage queue create --name $queueName --account-name $storageName --auth-mode login
```

[!INCLUDE [event-grid-register-provider-cli.md](../../includes/event-grid-register-provider-cli.md)]

## <a name="subscribe-to-your-app-configuration-store-events"></a>Подписка на события хранилища конфигурации приложения

Вы подписываетесь на эти два события из основного хранилища конфигурации приложений:

- `Microsoft.AppConfiguration.KeyValueModified`
- `Microsoft.AppConfiguration.KeyValueDeleted`

Следующая команда создает подписку на сетку событий для двух событий, отправляемых в очередь. Для типа конечной точки задано значение `storagequeue` , а для конечной точки ЗАДАН идентификатор очереди. Замените на `<event_subscription_name>` имя вашего выбора для подписки на события.

```azurecli-interactive
storageId=$(az storage account show --name $storageName --resource-group  $resourceGroupName --query id --output tsv)
queueId="$storageId/queueservices/default/queues/$queueName"
appconfigId=$(az appconfig show --name $primaryAppConfigName --resource-group $resourceGroupName --query id --output tsv)
eventSubscriptionName="<event_subscription_name>"
az eventgrid event-subscription create \
  --source-resource-id $appconfigId \
  --name $eventSubscriptionName \
  --endpoint-type storagequeue \
  --endpoint $queueId \
  --included-event-types Microsoft.AppConfiguration.KeyValueModified Microsoft.AppConfiguration.KeyValueDeleted 
```

## <a name="create-functions-for-handling-events-from-queue-storage"></a>Создание функций для обработки событий из хранилища очередей

### <a name="set-up-with-ready-to-use-functions"></a>Настройка готовых функций

В этой статье вы будете работать с функциями C#, которые имеют следующие свойства:
- Стек среды выполнения .NET Core 3,1
- Среда выполнения функций Azure версии 3. x
- Функция, активируемая таймером каждые 10 минут

Чтобы упростить резервное копирование данных, мы [протестировали и опубликовали функцию](https://github.com/Azure/AppConfiguration/tree/master/examples/ConfigurationStoreBackup) , которую можно использовать без внесения каких-либо изменений в код. Скачайте файлы проекта и [опубликуйте их в собственном приложении функции Azure из Visual Studio](../azure-functions/functions-develop-vs.md#publish-to-azure).

> [!IMPORTANT]
> Не вносите никаких изменений в переменные среды в скачанном коде. Необходимые параметры приложения будут созданы в следующем разделе.
>

### <a name="build-your-own-function"></a>Создание собственной функции

Если приведенный выше пример кода не соответствует вашим требованиям, можно также создать собственную функцию. Для завершения резервного копирования функция должна иметь возможность выполнять следующие задачи:
- Периодически Читайте содержимое очереди, чтобы проверить, содержит ли она уведомления из сетки событий. Сведения о реализации см. в [пакете SDK очереди хранилища](../storage/queues/storage-quickstart-queues-dotnet.md) .
- Если очередь содержит [уведомления о событиях из сетки событий](./concept-app-configuration-event.md#event-schema), извлеките все уникальные `<key, label>` сведения из сообщений о событиях. Сочетание клавиш и метки — это уникальный идентификатор для изменений «ключ-значение» в основном хранилище.
- Чтение всех параметров из основного хранилища. Обновите только те параметры в вторичном хранилище, которые имеют соответствующее событие в очереди. Удалите все параметры из вторичного хранилища, которые присутствовали в очереди, но отсутствуют в основном хранилище. [Пакет SDK для настройки приложений](https://github.com/Azure/AppConfiguration#sdks) можно использовать для программного доступа к хранилищам конфигурации.
- Удаление сообщений из очереди, если во время обработки не возникало исключений.
- Реализуйте обработку ошибок в соответствии с вашими потребностями. См. предыдущий пример кода, чтобы увидеть некоторые распространенные исключения, которые может потребоваться решить.

Дополнительные сведения о создании функции см. в статье [Создание функции в Azure, которая активируется таймером](../azure-functions/functions-create-scheduled-function.md) и [разрабатывает функции Azure с помощью Visual Studio](../azure-functions/functions-develop-vs.md).


> [!IMPORTANT]
> Используйте оптимальное значение для выбора расписания таймера в зависимости от частоты внесения изменений в основное хранилище конфигураций. Слишком частое выполнение функции может привести к завершению запросов на регулирование хранилища.
>


## <a name="create-function-app-settings"></a>Создание параметров приложения функции

Если вы используете предоставленную вами функцию, в приложении-функции потребуются следующие параметры приложения:
- `PrimaryStoreEndpoint`: Конечная точка для основного хранилища конфигурации приложений. Например, `https://{primary_appconfig_name}.azconfig.io`.
- `SecondaryStoreEndpoint`: Конечная точка для дополнительного хранилища конфигурации приложения. Например, `https://{secondary_appconfig_name}.azconfig.io`.
- `StorageQueueUri`: URI очереди. Например, `https://{unique_storage_name}.queue.core.windows.net/{queue_name}`.

Следующая команда создает необходимые параметры приложения в приложении функции. Замените `<function_app_name>` на имя приложения-функции.

```azurecli-interactive
functionAppName="<function_app_name>"
primaryStoreEndpoint="https://$primaryAppConfigName.azconfig.io"
secondaryStoreEndpoint="https://$secondaryAppConfigName.azconfig.io"
storageQueueUri="https://$storageName.queue.core.windows.net/$queueName"
az functionapp config appsettings set --name $functionAppName --resource-group $resourceGroupName --settings StorageQueueUri=$storageQueueUri PrimaryStoreEndpoint=$primaryStoreEndpoint SecondaryStoreEndpoint=$secondaryStoreEndpoint
```


## <a name="grant-access-to-the-managed-identity-of-the-function-app"></a>Предоставление доступа к управляемому удостоверению приложения функции

Используйте следующую команду или [портал Azure](../app-service/overview-managed-identity.md#add-a-system-assigned-identity) , чтобы добавить управляемое системой удостоверение, назначенное для приложения-функции.

```azurecli-interactive
az functionapp identity assign --name $functionAppName --resource-group $resourceGroupName
```

> [!NOTE]
> Для выполнения требуемого создания ресурсов и управления ролями учетной записи требуются `Owner` разрешения в соответствующей области (ваша подписка или группа ресурсов). Если вам нужна помощь с назначением ролей, Узнайте, [как назначать роли Azure с помощью портал Azure](../role-based-access-control/role-assignments-portal.md).

Используйте следующие команды или [портал Azure](./howto-integrate-azure-managed-service-identity.md#grant-access-to-app-configuration) , чтобы предоставить управляемому удостоверению приложения функции доступ к хранилищам конфигурации приложения. Используйте следующие роли:
- Назначьте `App Configuration Data Reader` роль в основном хранилище конфигураций приложений.
- Назначьте `App Configuration Data Owner` роль в хранилище конфигурации дополнительного приложения.

```azurecli-interactive
functionPrincipalId=$(az functionapp identity show --name $functionAppName --resource-group  $resourceGroupName --query principalId --output tsv)
primaryAppConfigId=$(az appconfig show -n $primaryAppConfigName --query id --output tsv)
secondaryAppConfigId=$(az appconfig show -n $secondaryAppConfigName --query id --output tsv)

az role assignment create \
    --role "App Configuration Data Reader" \
    --assignee $functionPrincipalId \
    --scope $primaryAppConfigId

az role assignment create \
    --role "App Configuration Data Owner" \
    --assignee $functionPrincipalId \
    --scope $secondaryAppConfigId
```

Используйте следующую команду или [портал Azure](../storage/common/storage-auth-aad-rbac-portal.md#assign-azure-roles-using-the-azure-portal) , чтобы предоставить управляемому удостоверению приложения функции доступ к очереди. Назначьте `Storage Queue Data Contributor` роль в очереди.

```azurecli-interactive
az role assignment create \
    --role "Storage Queue Data Contributor" \
    --assignee $functionPrincipalId \
    --scope $queueId
```

## <a name="trigger-an-app-configuration-event"></a>Активация события Конфигурации приложений

Чтобы убедиться, что все работает, можно создать, обновить или удалить значение ключа из основного хранилища. Это изменение должно автоматически отображаться в дополнительном хранилище через несколько секунд после того, как таймер активирует функции Azure.

```azurecli-interactive
az appconfig kv set --name $primaryAppConfigName --key Foo --value Bar --yes
```

Вы активировали событие. Через некоторое время в сетке событий будет отправлено уведомление о событии в очередь. *После следующего запланированного выполнения функции* просмотрите параметры конфигурации в дополнительном хранилище, чтобы узнать, содержит ли он обновленное значение ключа из основного хранилища.

> [!NOTE]
> Вы можете [активировать функцию вручную](../azure-functions/functions-manually-run-non-http.md) во время тестирования и устранения неполадок, не дожидаясь запланированного триггера таймера.

Убедившись в успешном выполнении функции резервного копирования, вы увидите, что ключ теперь имеется в дополнительном хранилище.

```azurecli-interactive
az appconfig kv show --name $secondaryAppConfigName --key Foo
```

```json
{
  "contentType": null,
  "etag": "eVgJugUUuopXnbCxg0dB63PDEJY",
  "key": "Foo",
  "label": null,
  "lastModified": "2020-04-27T23:25:08+00:00",
  "locked": false,
  "tags": {},
  "value": "Bar"
}
```

## <a name="troubleshooting"></a>Устранение неполадок

Если вы не видите новый параметр в дополнительном хранилище:

- Убедитесь, что функция резервного копирования была активирована *после* создания параметра в основном хранилище.
- Возможно, сетка событий не смогла отправить уведомление о событии в очередь вовремя. Проверьте, все ли ваша очередь содержит уведомление о событии из основного хранилища. Если это так, снова запустите функцию резервного копирования.
- Проверьте [журналы функций Azure](../azure-functions/functions-create-scheduled-function.md#test-the-function) на наличие ошибок или предупреждений.
- Используйте [портал Azure](../azure-functions/functions-how-to-use-azure-function-app-settings.md#get-started-in-the-azure-portal) , чтобы убедиться, что приложение-функция Azure содержит правильные значения параметров приложения, которые функции Azure пытаются считать.
- Вы также можете настроить мониторинг и оповещения для функций Azure с помощью [Application Insights Azure](../azure-functions/functions-monitoring.md?tabs=cmd). 


## <a name="clean-up-resources"></a>Очистка ресурсов
Если вы планируете продолжить работу с этой конфигурацией приложения и подпиской на события, не очищайте ресурсы, созданные в этой статье. Если вы не планируете продолжать работу, удалите ресурсы, созданные в этой статье, с помощью следующей команды.

```azurecli-interactive
az group delete --name $resourceGroupName
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали, как настроить автоматическую архивацию значений ключей, Узнайте больше о том, как можно повысить геоустойчивость приложения.

- [Устойчивость и аварийное восстановление](concept-disaster-recovery.md)
