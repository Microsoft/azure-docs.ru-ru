---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 07/20/2020
ms.author: msmbaldwin
ms.openlocfilehash: 97fd969ddae2f2c222209259cccd6f6f55272524
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99070275"
---
Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими. С помощью команды [az group create](/cli/azure/group#az_group_create) создайте группу ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli
az group create --name "myResourceGroup" -l "EastUS"
```
