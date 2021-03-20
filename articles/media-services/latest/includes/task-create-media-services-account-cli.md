---
author: IngridAtMicrosoft
ms.service: media-services
ms.topic: include
ms.date: 08/17/2020
ms.author: inhenkel
ms.custom: CLI
ms.openlocfilehash: 5c0341087cdd348e973da5faaa1f90081780c9c8
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88602259"
---
<!--Create a media services account -->

Указанная ниже команда Azure CLI создает новую учетную запись Служб мультимедиа. Можно заменить следующие значения: `amsaccount` `storageaccountforams` (должно соответствовать значению, указанному для учетной записи хранения) и `amsResourceGroup` (должно совпадать со значением, назначенным для группы ресурсов).  

```azurecli
az ams account create --name amsaccount -g amsResourceGroup --storage-account storageaccountforams -l westus2
```