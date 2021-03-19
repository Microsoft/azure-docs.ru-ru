---
title: Регионы Azure, в которых доступен Индексатор видео
titleSuffix: Azure Media Services
description: В этой статье рассказывается о регионах Azure, в которых доступен индексатор видео служб мультимедиа Azure.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 09/14/2020
ms.author: juliako
ms.openlocfilehash: 6ebdb22f50efbefc695f9752c6e6fc333571828c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90530949"
---
# <a name="azure-regions-in-which-video-indexer-exists"></a>Регионы Azure, в которых доступен Индексатор видео

API Индексатора видео содержат параметр **location**, для которого следует задать регион Azure, к которому должен направляться вызов. Это должен быть [регион Azure, в котором доступен Индексатор видео](https://azure.microsoft.com/global-infrastructure/services/?products=cognitive-services&regions=all).

## <a name="locations"></a>Расположения

`location`Параметру должно быть присвоено имя кода региона Azure в качестве значения. При использовании индексатора видео в режиме предварительного просмотра следует поместиться в `"trial"` качестве значения. `trial` значение по умолчанию для `location` параметра. В противном случае для получения имени кода региона Azure, в котором находится ваша учетная запись, а также для маршрутизации вызова, можно использовать портал Azure или выполнить команду [Azure CLI](/cli/azure) .

### <a name="azure-portal"></a>Портал Azure

1. Войдите на веб-сайт [Индексатора видео](https://www.videoindexer.ai/).
1. Выберите **учетные записи пользователей** в правом верхнем углу страницы.
1. Найдите расположение учетной записи в правом верхнем углу.  

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/location/location1.png" alt-text="Расположение":::
    
###  <a name="cli-command"></a>Команда CLI

```azurecli-interactive
az account list-locations
```

Когда вы запустите показанную выше строку, вы увидите список всех регионов Azure. Перейдите к региону Azure со значением *displayName*, которое вы ищете, и используйте его *имя* для параметра **location**.

Например, для региона Azure "западная часть США 2" (показан ниже) для параметра **location** нужно использовать значение westus2.

```json
   {
      "displayName": "West US 2",
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/locations/westus2",
      "latitude": "47.233",
      "longitude": "-119.852",
      "name": "westus2",
      "subscriptionId": null
    }
```

## <a name="next-steps"></a>Дальнейшие действия

- [Customize a Language model with the Video Indexer APIs](customize-language-model-with-api.md) (Настройка языковой модели с помощью API Индексатора видео)
- [Customize a Brands model with the Video Indexer API](customize-brands-model-with-api.md) (Настройка модели торговых марок с помощью API Индексатора видео)
- [Customize a Person model with the Video Indexer API](customize-person-model-with-api.md) (Настройка модели Person с помощью API Индексатора видео)
