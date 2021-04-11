---
title: Пример скрипта Azure CLI. Публикация ресурса
description: В этой статье показано, как использовать скрипт Azure CLI для публикации ресурса.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: ''
ms.service: media-services
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-azurecli
ms.openlocfilehash: b39056f4cf159fd837e7148b7966e3be57e2da78
ms.sourcegitcommit: edc7dc50c4f5550d9776a4c42167a872032a4151
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105963843"
---
# <a name="cli-example-publish-an-asset"></a>Пример CLI. Публикация ресурса

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

В этой статье показано, как создать потоковый указатель и вернуть URL-адрес для потоковой передачи с помощью сценария Azure CLI. 

## <a name="prerequisites"></a>предварительные требования 

[Создание учетной записи Служб мультимедиа](./account-create-how-to.md).

## <a name="example-script"></a>Пример сценария

[!code-azurecli-interactive[main](../../../cli_scripts/media-services/publish-asset/Publish-Asset.sh "Publish an asset")]

## <a name="next-steps"></a>Дальнейшие действия

[Общие сведения о службах мультимедиа](media-services-overview.md)
