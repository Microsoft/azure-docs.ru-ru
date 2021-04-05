---
title: Потоковая передача видеофайлов с помощью CLI Служб мультимедиа Azure
description: В этом руководстве показано, как с помощью Azure CLI создать учетную запись Служб мультимедиа Azure, закодировать файл и выполнить его потоковую передачу в Проигрыватель мультимедиа Azure.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
keywords: azure media services, stream
ms.service: media-services
ms.workload: media
ms.topic: tutorial
ms.custom: devx-track-azurecli
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: c78205d7e2b41628de9e8b92c9fa5506e82158cb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98954501"
---
# <a name="tutorial-encode-a-remote-file-based-on-url-and-stream-the-video---azure-cli"></a>Руководство по Кодирование удаленного файла на основе URL-адреса и потоковой передачи видео с помощью Azure CLI

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

В этом кратком руководстве описано, как легко выполнять кодирование и потоковую передачу видео в разных браузерах и на разных устройствах с помощью Служб мультимедиа Azure и Azure CLI. Входное содержимое можно определить с помощью протокола HTTPS, URL-адреса SAS или путей к файлам в хранилище BLOB-объектов Azure.

Пример в этой статье предназначен для кодирования содержимого, которое доступно через URL-адрес HTTPS. Версия 3 Служб мультимедиа сейчас не поддерживает кодирование блочной передачи через URL-адреса HTTPS.

Изучив это руководство, вы сможете выполнить потоковую передачу видео.  

![Воспроизведение видео](./media/stream-files-dotnet-quickstart/final-video.png)

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="create-a-media-services-account"></a>Создание учетной записи служб мультимедиа

Для того, чтобы шифровать, кодировать, анализировать, управлять и выполнять потоковую передачу мультимедиа в Azure, необходимо создать учетную запись Служб мультимедиа. Эта учетная запись должна быть связанна с одной или несколькими учетными записями хранения.

Ваша учетная запись Служб мультимедиа и все связанные учетные записи хранения должны находится в одной подписке Azure. Для ограничения задержки и затрат на исходящий трафик мы рекомендуем использовать учетные записи хранения, которые находятся в том же месте, что и учетная запись Служб мультимедиа.

### <a name="create-a-resource-group"></a>Создание группы ресурсов

```azurecli-interactive
az group create -n amsResourceGroup -l westus2
```

### <a name="create-an-azure-storage-account"></a>Создание учетной записи хранения Azure

В этом примере мы создадим учетную запись LRS категории "Стандартный"версии 2 для общего назначения.

Если вы хотите поэкспериментировать с учетными записями хранения, используйте `--sku Standard_LRS`. При выборе номера SKU для рабочей среды следует рассмотреть вариант применения `--sku Standard_RAGRS`, который предоставляет географическую репликацию для обеспечения непрерывности бизнес-процессов. Дополнительные сведения см. в статье об [учетных записях хранения](/cli/azure/storage/account).

```azurecli-interactive
az storage account create -n amsstorageaccount --kind StorageV2 --sku Standard_LRS -l westus2 -g amsResourceGroup
```

### <a name="create-an-azure-media-services-account"></a>Создание учетной записи служб мультимедиа Azure

```azurecli-interactive
az ams account create --n amsaccount -g amsResourceGroup --storage-account amsstorageaccount -l westus2
```

Вы получите примерно следующий ответ:

```
{
  "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount",
  "location": "West US 2",
  "mediaServiceId": "8b569c2e-d648-4fcb-9035-c7fcc3aa7ddf",
  "name": "amsaccount",
  "resourceGroup": "amsResourceGroupTest",
  "storageAccounts": [
    {
      "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Storage/storageAccounts/amsstorageaccount",
      "resourceGroup": "amsResourceGroupTest",
      "type": "Primary"
    }
  ],
  "tags": null,
  "type": "Microsoft.Media/mediaservices"
}
```

## <a name="start-the-streaming-endpoint"></a>Запуск конечной точки потоковой передачи

Следующая команда Azure CLI запускает установленную по умолчанию **конечную точку потоковой передачи**.

```azurecli-interactive
az ams streaming-endpoint start  -n default -a amsaccount -g amsResourceGroup
```

Вы получите примерно следующий ответ:

```
{
  "accessControl": null,
  "availabilitySetName": null,
  "cdnEnabled": true,
  "cdnProfile": "AzureMediaStreamingPlatformCdnProfile-StandardVerizon",
  "cdnProvider": "StandardVerizon",
  "created": "2019-02-06T21:58:03.604954+00:00",
  "crossSiteAccessPolicies": null,
  "customHostNames": [],
  "description": "",
  "freeTrialEndTime": "2019-02-21T22:05:31.277936+00:00",
  "hostName": "amsaccount-usw22.streaming.media.azure.net",
  "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount/streamingendpoints/default",
  "lastModified": "2019-02-06T21:58:03.604954+00:00",
  "location": "West US 2",
  "maxCacheAge": null,
  "name": "default",
  "provisioningState": "Succeeded",
  "resourceGroup": "amsResourceGroup",
  "resourceState": "Running",
  "scaleUnits": 0,
  "tags": {},
  "type": "Microsoft.Media/mediaservices/streamingEndpoints"
}
```

Если конечная точка потоковой передачи уже выполняется, вы получите такое сообщение:

```
(InvalidOperation) The server cannot execute the operation in its current state.
```

## <a name="create-a-transform-for-adaptive-bitrate-encoding"></a>Создание преобразования для кодирования с адаптивной скоростью

Создайте **преобразование** для настройки общих задач кодирования или анализа видеоматериалов. В этом примере мы выполним кодирование с адаптивной скоростью. Затем мы отправим задание для преобразования, которое мы создали. Задание содержит запрос к Службам мультимедиа на применение преобразования к указанному входному содержимому видео или аудио.

```azurecli-interactive
az ams transform create --name testEncodingTransform --preset AdaptiveStreaming --description 'a simple Transform for Adaptive Bitrate Encoding' -g amsResourceGroup -a amsaccount
```

Вы получите примерно следующий ответ:

```
{
  "created": "2019-02-15T00:11:18.506019+00:00",
  "description": "a simple Transform for Adaptive Bitrate Encoding",
  "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount/transforms/testEncodingTransform",
  "lastModified": "2019-02-15T00:11:18.506019+00:00",
  "name": "testEncodingTransform",
  "outputs": [
    {
      "onError": "StopProcessingJob",
      "preset": {
        "odatatype": "#Microsoft.Media.BuiltInStandardEncoderPreset",
        "presetName": "AdaptiveStreaming"
      },
      "relativePriority": "Normal"
    }
  ],
  "resourceGroup": "amsResourceGroup",
  "type": "Microsoft.Media/mediaservices/transforms"
}
```

## <a name="create-an-output-asset"></a>Создание выходного ресурса

Создайте выходной **ресурс** для использования в качестве выходных данных задания кодирования.

```azurecli-interactive
az ams asset create -n testOutputAssetName -a amsaccount -g amsResourceGroup
```

Вы получите примерно следующий ответ:

```
{
  "alternateId": null,
  "assetId": "96427438-bbce-4a74-ba91-e38179b72f36",
  "container": null,
  "created": "2019-02-14T23:58:19.127000+00:00",
  "description": null,
  "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount/assets/testOutputAssetName",
  "lastModified": "2019-02-14T23:58:19.127000+00:00",
  "name": "testOutputAssetName",
  "resourceGroup": "amsResourceGroup",
  "storageAccountName": "amsstorageaccount",
  "storageEncryptionFormat": "None",
  "type": "Microsoft.Media/mediaservices/assets"
}
```

## <a name="start-a-job-by-using-https-input"></a>Запуск задания с использованием входных данных HTTPS

При отправке заданий для обработки видео, необходимо указать Службам мультимедиа расположение входного видео. Это можно сделать, указав URL-адрес протокола HTTPS в качестве входных данных задания (как показано в этом примере).

При запуске `az ams job start` можно задать метку для выходных данных задания. Затем можно использовать метку для определения предназначения выходного ресурса.

- Если вы присваиваете значение метке, задайте для --output-assets значение assetname=label.
- Если вы не присваиваете значение метке, задайте для --output-assets значение assetname=.

  Обратите внимание, что мы добавляем знак "=" к `output-assets`.

```azurecli-interactive
az ams job start --name testJob001 --transform-name testEncodingTransform --base-uri 'https://nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/' --files 'Ignite-short.mp4' --output-assets testOutputAssetName= -a amsaccount -g amsResourceGroup
```

Вы получите примерно следующий ответ:

```
{
  "correlationData": {},
  "created": "2019-02-15T05:08:26.266104+00:00",
  "description": null,
  "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount/transforms/testEncodingTransform/jobs/testJob001",
  "input": {
    "baseUri": "https://nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/",
    "files": [
      "Ignite-short.mp4"
    ],
    "label": null,
    "odatatype": "#Microsoft.Media.JobInputHttp"
  },
  "lastModified": "2019-02-15T05:08:26.266104+00:00",
  "name": "testJob001",
  "outputs": [
    {
      "assetName": "testOutputAssetName",
      "error": null,
      "label": "",
      "odatatype": "#Microsoft.Media.JobOutputAsset",
      "progress": 0,
      "state": "Queued"
    }
  ],
  "priority": "Normal",
  "resourceGroup": "amsResourceGroup",
  "state": "Queued",
  "type": "Microsoft.Media/mediaservices/transforms/jobs"
}
```

### <a name="check-status"></a>Проверка состояния

Проверьте состояние задания через пять минут. Должно появиться состояние "Завершено". Если оно не завершено, проверьте еще раз через несколько минут. После того как появится состояние "Завершено", перейдите к следующему шагу и создайте **указатель потоковой передачи**.

```azurecli-interactive
az ams job show -a amsaccount -g amsResourceGroup -t testEncodingTransform -n testJob001
```

## <a name="create-a-streaming-locator-and-get-a-path"></a>Создания указателя потоковой передачи и получения пути

После выполнения кодирования необходимо сделать видео в выходном ресурсе доступным для воспроизведения клиентами. Чтобы сделать это, сначала необходимо создать указатель потоковой передачи. Затем создайте URL-адреса потоковой передачи, которые смогут использовать клиенты.

### <a name="create-a-streaming-locator"></a>Создание указателя потоковой передачи

```azurecli-interactive
az ams streaming-locator create -n testStreamingLocator --asset-name testOutputAssetName --streaming-policy-name Predefined_ClearStreamingOnly  -g amsResourceGroup -a amsaccount 
```

Вы получите примерно следующий ответ:

```
{
  "alternativeMediaId": null,
  "assetName": "output-3b6d7b1dffe9419fa104b952f7f6ab76",
  "contentKeys": [],
  "created": "2019-02-15T04:35:46.270750+00:00",
  "defaultContentKeyPolicyName": null,
  "endTime": "9999-12-31T23:59:59.999999+00:00",
  "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount/streamingLocators/testStreamingLocator",
  "name": "testStreamingLocator",
  "resourceGroup": "amsResourceGroup",
  "startTime": null,
  "streamingLocatorId": "e01b2be1-5ea4-42ca-ae5d-7fe704a5962f",
  "streamingPolicyName": "Predefined_ClearStreamingOnly",
  "type": "Microsoft.Media/mediaservices/streamingLocators"
}
```

### <a name="get-streaming-locator-paths"></a>Получение путей указателя потоковой передачи

```azurecli-interactive
az ams streaming-locator get-paths -a amsaccount -g amsResourceGroup -n testStreamingLocator
```

Вы получите примерно следующий ответ:

```
{
  "downloadPaths": [],
  "streamingPaths": [
    {
      "encryptionScheme": "NoEncryption",
      "paths": [
        "/e01b2be1-5ea4-42ca-ae5d-7fe704a5962f/ignite.ism/manifest(format=m3u8-aapl)"
      ],
      "streamingProtocol": "Hls"
    },
    {
      "encryptionScheme": "NoEncryption",
      "paths": [
        "/e01b2be1-5ea4-42ca-ae5d-7fe704a5962f/ignite.ism/manifest(format=mpd-time-csf)"
      ],
      "streamingProtocol": "Dash"
    },
    {
      "encryptionScheme": "NoEncryption",
      "paths": [
        "/e01b2be1-5ea4-42ca-ae5d-7fe704a5962f/ignite.ism/manifest"
      ],
      "streamingProtocol": "SmoothStreaming"
    }
  ]
}
```

Скопируйте путь HTTP Live Streaming (HLS). В этом случае он выглядит так: `/e01b2be1-5ea4-42ca-ae5d-7fe704a5962f/ignite.ism/manifest(format=m3u8-aapl)`.

## <a name="build-the-url"></a>Создание URL-адреса

### <a name="get-the-streaming-endpoint-host-name"></a>Получения имени узла конечной точки потоковой передачи

```azurecli-interactive
az ams streaming-endpoint list -a amsaccount -g amsResourceGroup -n default
```

Скопируйте значение `hostName`. В этом случае он выглядит так: `amsaccount-usw22.streaming.media.azure.net`.

### <a name="assemble-the-url"></a>Создание URL-адреса

"https:// " + &lt;значение hostName value&gt; + &lt;значение пути Hls&gt;

Ниже приведен пример:

`https://amsaccount-usw22.streaming.media.azure.net/7f19e783-927b-4e0a-a1c0-8a140c49856c/ignite.ism/manifest(format=m3u8-aapl)`

## <a name="test-playback-by-using-azure-media-player"></a>Тестовое воспроизведение с помощью Проигрывателя мультимедиа Azure

> [!NOTE]
> Если проигрыватель размещен на сайте с протоколом HTTPS, убедитесь, что URL-адрес начинается с https.

1. Откройте браузер и перейдите к [https://aka.ms/azuremediaplayer/](https://aka.ms/azuremediaplayer/).
2. В поле **URL-адрес** вставьте URL-адрес, который вы создали в предыдущем разделе. Можно вставить URL-адрес в формате HLS, Dash или Smooth. Проигрыватель мультимедиа Azure будет автоматически использовать соответствующий протокол потоковой передачи для воспроизведения на вашем устройстве.
3. Выберите **Update Player** (Обновить проигрыватель).

>[!NOTE]
>Проигрыватель мультимедиа Azure можно использовать для тестирования, но его нельзя применять в рабочей среде.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам больше не нужны ресурсы в группе ресурсов, включая Службы мультимедиа и учетные записи хранения, созданные в рамках этого руководства, удалите группу ресурсов.

Выполните следующую команду Azure CLI.

```azurecli-interactive
az group delete --name amsResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

[Общие сведения о службах мультимедиа](media-services-overview.md)
