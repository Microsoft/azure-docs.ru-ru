---
author: IngridAtMicrosoft
ms.service: media-services
ms.topic: include
ms.date: 08/18/2020
ms.author: inhenkel
ms.custom: CLI
ms.openlocfilehash: 9f079be76d4d75f05b4a8e132ac425255eed8558
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "88605965"
---
<!--Create a media services asset CLI-->

Указанная ниже команда Azure CLI создает ресурс Служб мультимедиа. Замените значения `assetName`, `amsAccountName` и `resourceGroup` значениями, с которыми вы работаете в данный момент.

```azurecli
az ams asset create -n assetName -a amsAccountName -g resourceGroup
```
