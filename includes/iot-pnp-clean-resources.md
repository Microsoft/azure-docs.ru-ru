---
author: dominicbetts
ms.author: dominicbetts
ms.service: iot-pnp
ms.topic: include
ms.date: 11/15/2019
ms.openlocfilehash: 647226091c3e15a19bf8262c23e62d95018488b3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99831213"
---
Если вы планируете продолжить работу с дополнительными статьями об IoT Plug and Play, вы можете сохранить и повторно задействовать ресурсы, которые вы использовали в этой статье. В противном случае вы можете удалить их, чтобы избежать дополнительных расходов.

Вы можете одновременно удалить центр и зарегистрированное устройство, удалив всю группу ресурсов с помощью приведенной команды Azure CLI. Не выполняйте данную команду, если эти ресурсы находятся в группе ресурсов с другими нужными вам ресурсами.

```azurecli-interactive
az group delete --name <YourResourceGroupName>
```

Чтобы удалить только Центр Интернета вещей, выполните следующую команду с помощью Azure CLI:

```azurecli-interactive
az iot hub delete --name <YourIoTHubName>
```

Чтобы удалить только то удостоверение устройства, которое вы зарегистрировали в Центре Интернета вещей, выполните следующую команду с помощью интерфейса командной строки Azure:

```azurecli-interactive
az iot hub device-identity delete --hub-name <YourIoTHubName> --device-id <YourDeviceID>
```

Вы также можете удалить клонированные примеры файлов с компьютера, на котором выполняется разработка.
