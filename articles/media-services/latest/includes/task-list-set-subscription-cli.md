---
author: IngridAtMicrosoft
ms.service: media-services
ms.topic: include
ms.date: 08/17/2020
ms.author: inhenkel
ms.custom: CLI
ms.openlocfilehash: ffd053b98a54d3e23eec62427f5c3d82df58954d
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107799885"
---
<!-- List and set subscriptions -->

1. Получите список своих подписок с помощью команды [az account list](/cli/azure/account#az_account_list):

    ```
    az account list --output table
    ```

2. Выполните команду `az account set`, указав идентификатор подписки или имя, на которое нужно переключиться.

    ```
    az account set --subscription "My Demos"
    ```
