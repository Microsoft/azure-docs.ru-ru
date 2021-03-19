---
title: Отправка событий хранилища больших двоичных объектов Azure в веб-конечную точку (PowerShell) | Документация Майкрософт
description: Используйте службу "Сетка событий Azure", чтобы подписываться на события хранилища BLOB-объектов, запускать события и просматривать результаты. Используйте Azure PowerShell для маршрутизации событий хранилища в веб-конечную точку.
author: normesta
ms.author: normesta
ms.reviewer: dastanfo
ms.date: 08/23/2018
ms.topic: article
ms.service: storage
ms.subservice: blobs
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 06be168ff9dfd55a56578b3afcdab8d984416756
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89078016"
---
# <a name="quickstart-route-storage-events-to-web-endpoint-with-powershell"></a>Краткое руководство. Маршрутизация событий хранилища в конечную точку веб-сайта с помощью PowerShell

"Сетка событий Azure" — это служба обработки событий для облака. В этой статье с помощью Azure PowerShell вы создадите подписку на события хранилища BLOB-объектов и активируете событие, чтобы увидеть результат. 

Как правило, события отправляются на конечную точку, которая обрабатывает данные событий и выполняет соответствующие действия. Но в этой статье для простоты события отправляются в веб-приложение, которое собирает и отображает сообщения.

По завершении можно увидеть, что данные событий отправлены в веб-приложение.

![Просмотр результатов](./media/storage-blob-event-quickstart-powershell/view-results.png)

## <a name="setup"></a>Настройка

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Для работы с этой статьей требуется последняя версия Azure PowerShell. Если вам необходимо выполнить установку или обновление, см. статью [об установке модуля Azure PowerShell](/powershell/azure/install-Az-ps).

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

Темами событий сетки являются ресурсы Azure, которые необходимо поместить в группу ресурсов Azure. Группа ресурсов Azure — это логическая коллекция, в которой выполняется развертывание и администрирование ресурсов Azure.

Создайте группу ресурсов с помощью команды [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).

В следующем примере создается группа ресурсов с именем **gridResourceGroup** в расположении **westus2**.  

```powershell
$resourceGroup = "gridResourceGroup"
New-AzResourceGroup -Name $resourceGroup -Location $location
```

## <a name="create-a-storage-account"></a>Создание учетной записи хранения

События хранилища BLOB-объектов доступны в учетных записях хранения общего назначения версии 2 и учетных записях хранения BLOB-объектов. Учетные записи хранения **общего назначения версии 2** поддерживают все функции для всех служб хранилища, включая большие двоичные объекты, файлы, очереди и таблицы. **Учетная запись хранения BLOB-объектов** — это специализированная учетная запись хранения для хранения неструктурированных данных в виде больших двоичных объектов в службе хранилища Azure. Учетные записи хранилища BLOB-объектов похожи на учетные записи хранения общего назначения и обладают такими же функциями обеспечения устойчивости, надежности, масштабируемости и производительности, которые вы уже используете, а также отличаются полной согласованностью API в плане блочных BLOB-объектов и добавления больших двоичных объектов. Дополнительные сведения см. в статье [Общие сведения об учетной записи хранения Azure](../common/storage-account-overview.md).

Создайте учетную запись хранения BLOB-объектов с репликацией LRS, используя командлет [New-AzStorageAccount](/powershell/module/az.storage/New-azStorageAccount), а затем извлеките контекст учетной записи хранения, который определяет используемую учетную запись хранения. Действуя в учетной записи хранения, ссылайтесь на контекст, вместо того чтобы многократно предоставлять учетные данные. В этом примере создается учетная запись хранения **gridstorage** с локально избыточным хранилищем (LRS). 

> [!NOTE]
> Имена учетных записей хранения находятся в глобальном пространстве имен, поэтому в имя, указанное в этом сценарии, необходимо добавить несколько произвольных знаков.

```powershell
$storageName = "gridstorage"
$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
  -Name $storageName `
  -Location $location `
  -SkuName Standard_LRS `
  -Kind BlobStorage `
  -AccessTier Hot

$ctx = $storageAccount.Context
```

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

[!INCLUDE [event-grid-register-provider-powershell.md](../../../includes/event-grid-register-provider-powershell.md)]

## <a name="subscribe-to-your-storage-account"></a>Подписка на учетную запись хранения

Подпишитесь на раздел, чтобы сообщить сетке событий, какие события следует отслеживанию. В следующем примере подписывается на созданную учетную запись хранения и передается URL-адрес из приложения в качестве конечной точки для уведомления о событии. Конечная точка веб-приложения должна содержать суффикс `/api/updates/`.

```powershell
$storageId = (Get-AzStorageAccount -ResourceGroupName $resourceGroup -AccountName $storageName).Id
$endpoint="https://$sitename.azurewebsites.net/api/updates"

New-AzEventGridSubscription `
  -EventSubscriptionName gridBlobQuickStart `
  -Endpoint $endpoint `
  -ResourceId $storageId
```

Теперь снова откройте веб-приложение и убедитесь, что оно успешно получило отправленное событие подтверждения подписки. Щелкните значок с изображением глаза, чтобы развернуть данные события. Сетка событий отправляет событие подтверждения, чтобы конечная точка могла подтвердить, что она готова получать данные события. Веб-приложение содержит код для проверки подписки.

![Просмотр события подписки](./media/storage-blob-event-quickstart-powershell/view-subscription-event.png)

## <a name="trigger-an-event-from-blob-storage"></a>Активация события из хранилища BLOB-объектов

Теперь необходимо активировать событие, чтобы увидеть, как Сетка событий Azure распределяет сообщение к вашей конечной точке. Сначала давайте создадим контейнер и объект. Затем давайте передадим этот объект в контейнер.

```powershell
$containerName = "gridcontainer"
New-AzStorageContainer -Name $containerName -Context $ctx

echo $null >> gridTestFile.txt

Set-AzStorageBlobContent -File gridTestFile.txt -Container $containerName -Context $ctx -Blob gridTestFile.txt
```

Вы активировали событие, а служба "Сетка событий" отправила сообщение в конечную точку, настроенную вами при оформлении подписки. Откройте веб-приложение и просмотрите в нем отправленные события.

```json
[{
  "topic": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myrg/providers/Microsoft.Storage/storageAccounts/myblobstorageaccount",
  "subject": "/blobServices/default/containers/gridcontainer/blobs/gridTestFile.txt",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2017-08-16T20:33:51.0595757Z",
  "id": "4d96b1d4-0001-00b3-58ce-16568c064fab",
  "data": {
    "api": "PutBlockList",
    "clientRequestId": "Azure-Storage-PowerShell-d65ca2e2-a168-4155-b7a4-2c925c18902f",
    "requestId": "4d96b1d4-0001-00b3-58ce-16568c000000",
    "eTag": "0x8D4E4E61AE038AD",
    "contentType": "application/octet-stream",
    "contentLength": 0,
    "blobType": "BlockBlob",
    "url": "https://myblobstorageaccount.blob.core.windows.net/gridcontainer/gridTestFile.txt",
    "sequencer": "00000000000000EB0000000000046199",
    "storageDiagnostics": {
      "batchId": "dffea416-b46e-4613-ac19-0371c0c5e352"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]

```

## <a name="clean-up-resources"></a>Очистка ресурсов
Если вы планируете продолжать работу с этой учетной записью хранения и подпиской на события, не очищайте ресурсы, созданные при работе с этой статьей. Если вы не планируете продолжать работу, используйте следующую команду, чтобы удалить все ресурсы, созданные в рамках этой статьи.

```powershell
Remove-AzResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы знаете, как создавать разделы и подписки на события, ознакомьтесь с дополнительными сведениями о событиях хранилища BLOB-объектов и возможностях службы "Сетка событий":

- [Reacting to Blob storage events (preview)](storage-blob-event-overview.md) (Реагирование на события хранилища BLOB-объектов)
- [An introduction to Azure Event Grid](../../event-grid/overview.md) (Общие сведения о службе "Сетка событий Azure")
