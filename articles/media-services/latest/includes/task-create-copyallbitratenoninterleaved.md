---
author: IngridAtMicrosoft
ms.service: media-services
ms.topic: include
ms.date: 11/18/2020
ms.author: inhenkel
ms.custom: REST
ms.openlocfilehash: 26492ba71726cefe863b27162dd901c945c3ddb7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96509338"
---
<!--Create a copyAllNonInterleave transform with REST-->

Следующая команда Azure RESTFUL создает преобразование с помощью `copyAllBitrateNonInterleaved` из `#Microsoft.Media.BuiltInStandardEncoderPreset` предустановки. Замените значения `subscriptionID` , `resourceGroup` и значениями, `accountName` с которыми вы работаете в данный момент. Присвойте имя преобразованию, задав `transformName` . 

Чтобы просмотреть все доступные преобразования REST API, см. раздел [преобразования — создание или обновление](/rest/api/media/transforms/createorupdate#definitions).

```REST
PUT https://management.azure.com/subscriptions/{{subscriptionId}}/resourceGroups/{{resourceGroup}}/providers/Microsoft.Media/mediaServices/{{accountName}}/transforms/{{transformName}}?api-version=2020-05-01

```

## <a name="body"></a>Текст

```json
{
    "name": "myTransform",
    "id": "/subscriptions/35c2594a-23da-4fce-b59c-f6fb9513eeeb/resourceGroups/inhenkelRG/providers/Microsoft.Media/mediaservices/inhenkel/transforms/myTransform",
    "type": "Microsoft.Media/mediaservices/transforms",
    "properties": {
        "description": "Basic Transform using a copyAllBitrateNonInterleaved encoding preset from the library of built-in Standard Encoder presets",
        "outputs": [
            {
                "onError": "StopProcessingJob",
                "relativePriority": "Normal",
                "preset": {
                    "@odata.type": "#Microsoft.Media.BuiltInStandardEncoderPreset",
                    "presetName": "CopyAllBitrateNonInterleaved"
                }
            }
        ]
    }
}
```

## <a name="sample-response"></a>Пример ответа
```json
{
  "name":"myTransform",
    "id":"/subscriptions/35c2594a-23da-4fce-b59c-f6fb9513eeeb/resourceGroups/inhenkelRG/providers/Microsoft.Media/mediaservices/inhenkel/transforms/myTransform","type":"Microsoft.Media/mediaservices/transforms","properties":{
        "created":"2020-11-18T19:19:27.4405209Z",
        "description":"Basic Transform using a copyAllBitrateNonInterleaved encoding preset from the library of built-in Standard Encoder presets","lastModified":"2020-11-18T19:19:27.4405209Z",
        "outputs":[
            {
                "onError":"StopProcessingJob","relativePriority":"Normal","preset":{
                "@odata.type":"#Microsoft.Media.BuiltInStandardEncoderPreset",
                "presetName":"CopyAllBitrateNonInterleaved"
        }
      }
    ]
  }
}
```