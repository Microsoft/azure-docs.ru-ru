---
title: Мониторинг журналов диагностики с помощью Azure Monitor
description: В этой статье показано, как маршрутизировать и просматривать журналы диагностики с помощью Azure Monitor.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 03/17/2021
ms.author: inhenkel
ms.custom: devx-track-azurecli
ms.openlocfilehash: 0719fede4147943e76ddfea1c5e77388c7c5cc9f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104609744"
---
# <a name="monitor-media-services-diagnostic-logs"></a>Мониторинг журналов диагностики служб мультимедиа

[!INCLUDE [media services api v3 logo](../includes/v3-hr.md)]

[Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/overview.md) позволяет отслеживать метрики и журналы диагностики, которые помогут понять, как работают приложения. Подробное описание этой функции и сведения о том, почему вы хотите использовать метрики и журналы диагностики служб мультимедиа Azure, см. в статье [мониторинг метрик служб мультимедиа и журналов диагностики](monitor-media-services.md).

В этой статье показано, как перенаправлять данные в учетную запись хранения и просматривать их.

## <a name="prerequisites"></a>Предварительные требования

- [Создание учетной записи Служб мультимедиа](../create-account-howto.md).
- Проверьте  [монитор служб мультимедиа](monitor-media-services.md).

## <a name="route-data-to-the-storage-account-using-the-portal"></a>Маршрутизация данных в учетную запись хранения с помощью портала

1. Войдите на портал Azure по адресу https://portal.azure.com.
1. Перейдите к учетной записи служб мультимедиа в и щелкните **параметры диагностики** в разделе **монитор**. Здесь можно увидеть список всех ресурсов в подписке, создающих данные мониторинга с помощью Azure Monitor.

    ![Снимок экрана, в котором показаны параметры диагностики в разделе "Мониторинг".](../media/media-services-diagnostic-logs/logs01.png)

1. Щелкните **Добавить параметр диагностики**.

   Параметр диагностики ресурса — это определение того, *какие* данные мониторинга нужно маршрутизировать из определенного ресурса и *куда* их необходимо отправить.

1. В открывшемся разделе назначьте параметру **Имя** и установите флажок **Архивировать в учетной записи хранения**.

    Выберите учетную запись хранения, в которую нужно отправить журналы, и нажмите кнопку **ОК**.
1. Установите все флажки в разделе **Журнал** и **Метрика**. В зависимости от типа ресурса у вас может быть только один из этих вариантов. Эти флажки определяют, какие категории журналов и данных метрик, доступные для этого типа ресурса, отправляются в выбранное место назначения (в данном случае в учетную запись хранения).

   ![Раздел параметров диагностики](../media/media-services-diagnostic-logs/logs02.png)
1. Задайте для ползунка **Хранение (в днях)** значение 30. Этот ползунок задает количество дней хранения данных мониторинга в учетной записи хранения. Azure Monitor автоматически удаляет данные, которые хранятся дольше заданного количества дней. Нулевое значение для периода хранения означает, что данные будут храниться неограниченно долго.
1. Выберите команду **Сохранить**.

Данные мониторинга из ресурса теперь поступают в учетную запись хранения.

## <a name="route-data-to-the-storage-account-using-the-azure-cli"></a>Перенаправление данных в учетную запись хранения с помощью Azure CLI

Чтобы включить хранение журналов диагностики в учетной записи хранения, выполните следующую `az monitor diagnostic-settings` команду Azure CLI:

```azurecli-interactive
az monitor diagnostic-settings create --name <diagnostic name> \
    --storage-account <name or ID of storage account> \
    --resource <target resource object ID> \
    --resource-group <storage account resource group> \
    --logs '[
    {
        "category": <category name>,
        "enabled": true,
        "retentionPolicy": {
            "days": <# days to retain>,
            "enabled": true
        }
    }]'
```

Пример:

```azurecli-interactive
az monitor diagnostic-settings create --name amsv3diagnostic \
    --storage-account storageaccountforams  \
    --resource "/subscriptions/00000000-0000-0000-0000-0000000000/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount" \
    --resource-group "amsResourceGroup" \
    --logs '[{"category": "KeyDeliveryRequests",  "enabled": true, "retentionPolicy": {"days": 3, "enabled": true }}]'
```

## <a name="view-data-in-the-storage-account-using-the-portal"></a>Просмотр данных в учетной записи хранения с помощью портала

Если выполнены предыдущие шаги, данные будут поступать в учетную запись хранения.

Событие может появиться в учетной записи через пять минут.

1. На портале перейдите к разделу **Учетные записи хранения**, который можно найти в области навигации слева.
1. Идентифицируйте учетную запись хранения, созданную в предыдущем разделе, и щелкните ее.
1. Щелкните **BLOB-объекты**, а затем в контейнере с меткой **Insights — Logs-кэйделиверирекуестс**. Это контейнер, в котором находятся журналы. Данные мониторинга разбиваются на контейнеры по ИДЕНТИФИКАТОРу ресурса, а затем по дате и времени.
1. Перейдите к файлу PT1H.json, щелкнув контейнеры, чтобы получить идентификаторы ресурса, дату и время. Щелкните файл PT1H.json, а затем — **Скачать**.

 Теперь можно просмотреть событие JSON, сохраненное в учетной записи хранения.

### <a name="examples-of-pt1hjson"></a>Примеры PT1H.js

#### <a name="clear-key-delivery-log"></a>Очистить журнал доставки ключей

```json
{
  "time": "2019-05-21T00:07:33.2820450Z",
  "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-0000000000000/RESOURCEGROUPS/amsResourceGroup/PROVIDERS/MICROSOFT.MEDIA/MEDIASERVICES/AMSACCOUNT",
  "operationName": "MICROSOFT.MEDIA/MEDIASERVICES/CONTENTKEYS/READ",
  "operationVersion": "1.0",
  "category": "KeyDeliveryRequests",
  "resultType": "Succeeded",
  "resultSignature": "OK",
  "durationMs": 253,
  "identity": {
    "authorization": {
      "issuer": "myIssuer",
      "audience": "myAudience"
    },
    "claims": {
      "urn:microsoft:azure:mediaservices:contentkeyidentifier": "e4276e1d-c012-40b1-80d0-ac15808b9277",
      "nbf": "1558396914",
      "exp": "1558400814",
      "iss": "myIssuer",
      "aud": "myAudience"
    }
  },
  "level": "Informational",
  "location": "westus2",
  "properties": {
    "requestId": "fb5c2b3a-bffa-4434-9c6f-73d689649add",
    "keyType": "Clear",
    "keyId": "e4276e1d-c012-40b1-80d0-ac15808b9277",
    "policyName": "SharedContentKeyPolicyUsedByAllAssets",
    "tokenType": "JWT",
    "statusMessage": "OK"
  }
}
```

#### <a name="widevine-encrypted-key-delivery-log"></a>Журнал доставки зашифрованного ключа Widevine

```json
{
  "time": "2019-05-20T23:15:22.7088747Z",
  "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-0000000000000/RESOURCEGROUPS/amsResourceGroup/PROVIDERS/MICROSOFT.MEDIA/MEDIASERVICES/AMSACCOUNT",
  "operationName": "MICROSOFT.MEDIA/MEDIASERVICES/CONTENTKEYS/READ",
  "operationVersion": "1.0",
  "category": "KeyDeliveryRequests",
  "resultType": "Succeeded",
  "resultSignature": "OK",
  "durationMs": 69,
  "identity": {
    "authorization": {
      "issuer": "myIssuer",
      "audience": "myAudience"
    },
    "claims": {
      "urn:microsoft:azure:mediaservices:contentkeyidentifier": "0092d23a-0a42-4c5f-838e-6d1bbc6346f8",
      "nbf": "1558392430",
      "exp": "1558396330",
      "iss": "myIssuer",
      "aud": "myAudience"
    }
  },
  "level": "Informational",
  "location": "westus2",
  "properties": {
    "requestId": "49613dd2-16aa-4595-a6e0-4e68beae6d37",
    "keyType": "Widevine",
    "keyId": "0092d23a-0a42-4c5f-838e-6d1bbc6346f8",
    "policyName": "DRMContentKeyPolicy",
    "tokenType": "JWT",
    "statusMessage": "OK"
  }
}
```

## <a name="additional-notes"></a>Дополнительные замечания

* Widevine — это служба, которая предоставляется компанией Google Inc. и подпадает под условия предоставления услуг и политику конфиденциальности Google Inc.

## <a name="see-also"></a>См. также раздел

* [Метрики Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/data-platform.md)
* [Журналы диагностики Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/essentials/platform-logs-overview.md)
* [Как получить и использовать данные журнала из ресурсов Azure](https://docs.microsoft.com/azure/azure-monitor/essentials/platform-logs-overview.md)

## <a name="next-steps"></a>Дальнейшие действия

[Метрики мониторинга](media-services-metrics-howto.md)
