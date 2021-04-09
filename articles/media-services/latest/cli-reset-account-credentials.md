---
title: Сброс данных учетной записи с помощью CLI
description: Используйте сценарий Azure CLI, что сбросить учетные данные учетной записи и вернуть параметры app.config.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: ''
ms.service: media-services
ms.devlang: azurecli
ms.topic: troubleshooting
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-azurecli
ms.openlocfilehash: cc605a08147da1d076b302e515a4ebe8d411a782
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101091843"
---
# <a name="azure-cli-example-reset-the-account-credentials"></a>Пример для Azure CLI: Сбросьте данные учетной записи

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

В сценарии Azure CLI в этой статье показано, как сбросить учетные данные учетной записи и вернуть параметры app.config.

## <a name="prerequisites"></a>Предварительные требования

[Создание учетной записи Служб мультимедиа](./create-account-howto.md).

## <a name="example-script"></a>Пример сценария

```azurecli-interactive
# Update the following variables for your own settings:
resourceGroup=amsResourceGroup
amsAccountName=amsmediaaccountname

az ams account sp reset-credentials \
  --account-name $amsAccountName \
  --resource-group $resourceGroup
 ```

## <a name="next-steps"></a>Дальнейшие действия

* [az ams](/cli/azure/ams)
* [Сброс учетных данных](/cli/azure/ams/account/sp#az-ams-account-sp-reset-credentials)
