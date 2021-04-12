---
title: Краткое руководство. Маршрутизация событий Кэша Azure для Redis в конечную веб-точку с помощью PowerShell
description: Сведения о том, как с помощью Сетки событий Azure подписаться на события Кэша Azure для Redis, отправить события веб-перехватчику и обработать события в веб-приложении.
ms.date: 1/5/2021
author: curib
ms.author: cauribeg
ms.topic: quickstart
ms.service: cache
ms.openlocfilehash: 6c3b433a8e433f39b723a7155bb6de116857efca
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102508169"
---
# <a name="quickstart-route-azure-cache-for-redis-events-to-web-endpoint-with-powershell"></a>Краткое руководство. Маршрутизация событий Кэша Azure для Redis в конечную веб-точку с помощью PowerShell

"Сетка событий Azure" — это служба обработки событий для облака. При работе с этим кратким руководством вы примените PowerShell, чтобы подписаться на событий Кэша Azure для Redis, активировать такое событие и просмотреть полученные результаты. 

Как правило, события отправляются на конечную точку, которая обрабатывает данные событий и выполняет соответствующие действия. Для простоты в этом кратком руководстве описана отправка событий непосредственно в веб-приложение, которое будет собирать и отображать эти сообщения. Выполнив описанные в этом кратком руководстве действия, вы увидите, что данные событий отправлены в веб-приложение.

## <a name="setup"></a>Настройка

Для работы с этим кратким руководством требуется последняя версия Azure PowerShell. Если вам необходимо выполнить установку или обновление, см. статью [об установке модуля Azure PowerShell](/powershell/azure/install-Az-ps).

## <a name="sign-in-to-azure"></a>Вход в Azure

Чтобы выполнить проверку подлинности, войдите в подписку Azure с помощью команды `Connect-AzAccount` и следуйте инструкциям на экране.

```powershell
Connect-AzAccount
```

В этом примере используется регион **westus2**, который сохраняется в переменной для повсеместного использования.

```powershell
$location = "westus2"
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Разделы Сетки событий развертываются как отдельные ресурсы Azure и должны быть подготовлены в группе ресурсов Azure. Группа ресурсов Azure — это логическая коллекция, в которой выполняется развертывание и администрирование ресурсов Azure.

Создайте группу ресурсов с помощью команды [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).

В следующем примере создается группа ресурсов с именем **gridResourceGroup** в расположении **westus2**.  

```powershell
$resourceGroup = "gridResourceGroup"
New-AzResourceGroup -Name $resourceGroup -Location $location
```

## <a name="create-an-azure-cache-for-redis-instance"></a>Создание экземпляра кэша Azure для Redis 

```powershell
New-AzRedisCache
   -ResourceGroupName <String>
   -Name <String>
   -Location <String>
   [-Size <String>]
   [-Sku <String>]
   [-RedisConfiguration <Hashtable>]
   [-EnableNonSslPort <Boolean>]
   [-TenantSettings <Hashtable>]
   [-ShardCount <Int32>]
   [-MinimumTlsVersion <String>]
   [-SubnetId <String>]
   [-StaticIP <String>]
   [-Tag <Hashtable>]
   [-Zone <String[]>]
   [-DefaultProfile <IAzureContextContainer>]
   [-WhatIf]
   [-Confirm]
   [<CommonParameters>]
```
Дополнительные сведения о создании экземпляра кэша с помощью PowerShell см. в [справочнике по Azure PowerShell](/powershell/module/az.rediscache/new-azrediscache). 

## <a name="create-a-message-endpoint"></a>Создание конечной точки сообщения

Перед подпиской на раздел необходимо создать конечную точку для сообщения о событии. Обычно конечная точка выполняет действия на основе данных событий. Чтобы упростить работу с этим руководством, разверните [готовое веб-приложение](https://github.com/Azure-Samples/azure-event-grid-viewer), которое отображает сообщения о событиях. Развернутое решение содержит план службы приложений, веб-приложение службы приложений и исходный код из GitHub.

Замените `<your-site-name>` уникальным именем для вашего веб-приложения. Имя веб-приложения должно быть уникальным, так как оно включается в запись DNS.

```powershell
$sitename="<your-site-name>"

New-AzResourceGroupDeployment `
  -ResourceGroupName $resourceGroup `
  -TemplateUri "https://raw.githubusercontent.com/Azure-Samples/azure-event-grid-viewer/master/azuredeploy.json" `
  -siteName $sitename `
  -hostingPlanName viewerhost
```

Завершение развертывания может занять несколько минут. Когда развертывание успешно завершится, откройте веб-приложение и убедитесь, что оно работает. Откройте браузер и перейдите по адресу `https://<your-site-name>.azurewebsites.net`.

Вы увидите сайт, на котором сейчас не отображаются никакие сообщения.

:::image type="content" source="media/cache-event-grid-portal/blank-event-grid-viewer.png" alt-text="Пустой сайт средства просмотра Сетки событий.":::

## <a name="subscribe-to-your-azure-cache-for-redis-event"></a>Подписка на событие Кэша Azure для Redis

На этом этапе вы создадите подписку на раздел, чтобы указать Сетке событий, какие события нужно отслеживать. В следующем примере создается подписка на созданный экземпляр кэша и передается URL-адрес веб-приложения в качестве конечной точки для уведомления о событиях. Конечная точка веб-приложения должна содержать суффикс `/api/updates/`.

```powershell
$cacheId = (Get-AzRedisCache -ResourceGroupName $resourceGroup -Name $cacheName).Id
$endpoint="https://$sitename.azurewebsites.net/api/updates"

New-AzEventGridSubscription `
  -EventSubscriptionName <event_subscription_name> `
  -Endpoint $endpoint `
  -ResourceId $cacheId
```

Теперь снова откройте веб-приложение и убедитесь, что оно успешно получило отправленное событие подтверждения подписки. Щелкните значок с изображением глаза, чтобы развернуть данные события. Сетка событий отправляет событие подтверждения, чтобы конечная точка могла подтвердить, что она готова получать данные события. Веб-приложение содержит код для проверки подписки.

  :::image type="content" source="media/cache-event-grid-portal/subscription-event.png" alt-text="Средство просмотра Сетки событий Azure.":::

## <a name="trigger-an-event-from-azure-cache-for-redis"></a>Активация события из Кэша Azure для Redis

Теперь необходимо активировать событие, чтобы увидеть, как Сетка событий Azure распределяет сообщение к вашей конечной точке.

```powershell
Import-AzRedisCache
      [-ResourceGroupName <String>]
      -Name <String>
      -Files <String[]>
      [-Format <String>]
      [-Force]
      [-PassThru]
      [-DefaultProfile <IAzureContextContainer>]
      [-WhatIf]
      [-Confirm]
      [<CommonParameters>]
```
Дополнительные сведения об импорте с помощью PowerShell см. в [справочнике по Azure PowerShell](/powershell/module/az.rediscache/import-azrediscache). 

Вы активировали событие, а служба "Сетка событий" отправила сообщение в конечную точку, настроенную вами при оформлении подписки. Откройте веб-приложение и просмотрите в нем отправленные события.

```json
[{
"id": "e1ceb52d-575c-4ce4-8056-115dec723cff",
  "eventType": "Microsoft.Cache.ImportRDBCompleted",
  "topic": "/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Cache/Redis/{cache_name}",
  "data": {
    "name": "ImportRDBCompleted",
    "timestamp": "2020-12-10T18:07:54.4937063+00:00",
    "status": "Succeeded"
  },
  "subject": "ImportRDBCompleted",
  "dataversion": "1.0",
  "metadataVersion": "1",
  "eventTime": "2020-12-10T18:07:54.4937063+00:00"
}]

```

## <a name="clean-up-resources"></a>Очистка ресурсов
Если вы планируете дальнейшую работу с этим экземпляром Кэша Azure для Redis и (или) с этой подпиской на события, не удаляйте ресурсы, созданные при работе с этим кратким руководством. Если вы не планируете продолжать работу, используйте приведенную ниже команду, чтобы удалить все ресурсы, созданные в рамках этого краткого руководства.

```powershell
Remove-AzResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>Следующие шаги

Теперь, когда вы знаете, как создавать разделы и подписки на события, ознакомьтесь с дополнительными сведениями о событиях Кэша Azure для Redis и возможностях службы "Сетка событий":

- [Реагирование на события Кэша Azure для Redis](cache-event-grid.md)
- [An introduction to Azure Event Grid](../event-grid/overview.md) (Общие сведения о службе "Сетка событий Azure")