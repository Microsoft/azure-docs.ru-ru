---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 07/20/2020
ms.author: msmbaldwin
ms.openlocfilehash: 97fd969ddae2f2c222209259cccd6f6f55272524
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99070275"
---
Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими. С помощью команды [az group create](/cli/azure/group#az_group_create) создайте группу ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli
az group create --name "myResourceGroup" -l "EastUS"
```
