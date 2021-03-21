---
author: baanders
description: файл включения для Azure Digital Twins — настройка Cloud Shell и расширения Интернета вещей
ms.service: digital-twins
ms.topic: include
ms.date: 7/17/2020
ms.author: baanders
ms.openlocfilehash: e61b6b9a09d759571029db4f01dd8f9d707ca518
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102245102"
---
Чтобы начать работу с Azure Digital Twins, сначала выполните вход в открытом окне [Azure Cloud Shell](https://shell.azure.com) и задайте контекст оболочки для подписки для этого сеанса. Выполните в Cloud Shell следующие команды:

```azurecli-interactive
az login
az account set --subscription "<your-Azure-subscription-ID>"
```
> [!TIP]
> В команде выше вы также можете использовать имя подписки вместо идентификатора. 

Если вы впервые используете эту подписку с Azure Digital Twins, выполните эту команду, чтобы зарегистрироваться в пространстве имен Azure Digital Twins. (Если вы не уверены, то можете запустить команду снова, даже если вы уже запускали ее.)

```azurecli-interactive
az provider register --namespace 'Microsoft.DigitalTwins'
```

Затем добавьте в Cloud Shell расширение [**Microsoft Azure IoT для Azure CLI**](/cli/azure/ext/azure-iot/iot), чтобы разрешить использовать команды для взаимодействия с Azure Digital Twins и другими службами Интернета вещей. 

[!INCLUDE [digital-twins-cloud-shell-extensions.md](digital-twins-cloud-shell-extensions.md)]

Теперь вы готовы к работе с Azure Digital Twins в Cloud Shell.

Это можно проверить в любой момент, выполнив команду `az dt -h`, чтобы вывести список доступных команд Azure Digital Twins верхнего уровня.