---
title: Политики потоковой передачи в службах мультимедиа Azure
description: В этой статье приведены общие сведения о политиках потоковой передачи и их использовании в Службах мультимедиа Azure.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: conceptual
ms.date: 05/28/2019
ms.author: inhenkel
ms.openlocfilehash: 2873486b95f0bb4a9a27e2e050cc6f6d5473a44e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98898178"
---
# <a name="streaming-policies"></a>Политики потоковой передачи

[Политики потоковой передачи](/rest/api/media/streamingpolicies) в Службах мультимедиа Azure версии 3 позволяют определять протоколы потоковой передачи и параметры шифрования для [указателей потоковой передачи](streaming-locators-concept.md). В службах мультимедиа v3 предусмотрены некоторые стандартные политики потоковой передачи, которые можно использовать непосредственно для пробной или рабочей среды. 

Доступные в настоящее время политики потоковой передачи:<br/>
* "Predefined_DownloadOnly"
* "Predefined_ClearStreamingOnly"
* "Predefined_DownloadAndClearStreaming"
* "Predefined_ClearKey"
* "Predefined_MultiDrmCencStreaming" 
* "Predefined_MultiDrmStreaming"

Следующее "дерево решений" помогает выбрать предопределенную политику потоковой передачи для вашего сценария.

> [!IMPORTANT]
> * Свойства **политик потоковой передачи** типа Datetime всегда задаются в формате UTC.
> * Следует разработать ограниченный набор политик для учетной записи Служб мультимедиа и повторно использовать их для указателей потоковой передачи каждый раз, когда требуются те же параметры. Дополнительные сведения см. в разделе [квоты и ограничения](limits-quotas-constraints.md).

## <a name="decision-tree"></a>Дерево решений

Щелкните изображение, чтобы просмотреть его полноразмерную версию.  

[![Схема, на которой показано дерево принятия решений, помогающее выбрать предопределенную политику потоковой передачи для вашего сценария.](./media/streaming-policy/large.png)](./media/streaming-policy/large.png#lightbox)

При шифровании содержимого необходимо создать [политику ключа содержимого](content-key-policy-concept.md). **Политика ключа содержимого** не требуется для очистки потоковой передачи или загрузки. 

Если у вас есть особые требования (например, если нужно указать разные протоколы, использовать пользовательскую службу доставки ключей или необходимо использовать четкую звуковую дорожку), можно [создать](/rest/api/media/streamingpolicies/create) пользовательскую политику потоковой передачи. 

## <a name="get-a-streaming-policy-definition"></a>Получение определения политики потоковой передачи  

Если вы хотите просмотреть определение политики потоковой передачи, используйте [Get](/rest/api/media/streamingpolicies/get) и укажите имя политики. Пример:

### <a name="rest"></a>REST

Запрос:

```
GET https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/contoso/providers/Microsoft.Media/mediaServices/contosomedia/streamingPolicies/clearStreamingPolicy?api-version=2018-07-01
```

Ответ:

```
{
  "name": "clearStreamingPolicy",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/contoso/providers/Microsoft.Media/mediaservices/contosomedia/streamingPolicies/clearStreamingPolicy",
  "type": "Microsoft.Media/mediaservices/streamingPolicies",
  "properties": {
    "created": "2018-08-08T18:29:30.8501486Z",
    "noEncryption": {
      "enabledProtocols": {
        "download": true,
        "dash": true,
        "hls": true,
        "smoothStreaming": true
      }
    }
  }
}
```

## <a name="filtering-ordering-paging"></a>Фильтрации, упорядочивание, разбиение по страницам

Ознакомьтесь с разделом [Фильтрация, упорядочивание и разбиение по страницам сущностей Служб мультимедиа](entities-overview.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Краткое руководство по потоковой передаче видеофайлов — .NET](stream-files-dotnet-quickstart.md)
* [Использование динамического шифрования AES-128 и службы доставки ключей](protect-with-aes128.md)
* [Использование динамического шифрования DRM и службы доставки лицензий](protect-with-drm.md)
