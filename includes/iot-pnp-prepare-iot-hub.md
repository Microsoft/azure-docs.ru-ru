---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 03/17/2020
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: ea5d4ef26fb14e22b871bb4bfa1054cb749d38e8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97673037"
---
## <a name="prepare-an-iot-hub"></a>Подготовка Центра Интернета вещей

Для выполнения инструкций, приведенных в этой статье, необходим центр Интернета вещей Azure в подписке Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

Если вы используете Azure CLI локально, сначала войдите в подписку Azure с помощью `az login`. Если вы используете эти команды в Azure Cloud Shell, то вход выполняется автоматически.

Если вы используете Azure CLI локально, версия `az` должна быть **2.8.0** или выше. В Azure Cloud Shell используется последняя версия. Используйте команду `az --version`, чтобы проверить версию, установленную на компьютере.

Выполните следующую команду, чтобы добавить расширение Интернета вещей Microsoft Azure для Azure CLI в экземпляр:

```azurecli-interactive
az extension add --name azure-iot
```

Если вы еще не используете центр Интернета вещей, выполните следующие команды, чтобы создать в подписке группу ресурсов и центр Интернета вещей ценовой категории "Бесплатный". Замените `<YourIoTHubName>` именем центра по своему выбору.

```azurecli-interactive
az group create --name my-pnp-resourcegroup \
    --location centralus
az iot hub create --name <YourIoTHubName> \
    --resource-group my-pnp-resourcegroup --sku F1
```

Выполните приведенную ниже команду, чтобы создать удостоверение устройства в Центре Интернета вещей. Замените заполнители `<YourIoTHubName>` и `<YourDeviceID>` своими значениями _имени центра Интернета вещей_ и _идентификатора устройства_.

```azurecli-interactive
az iot hub device-identity create --hub-name <YourIoTHubName> --device-id <YourDeviceID>
```
