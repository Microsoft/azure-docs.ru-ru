---
author: baanders
description: включаемый файл для удаления экземпляра Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 2/4/2021
ms.author: baanders
ms.openlocfilehash: 9a02c4f5c5699b4a6308bfaa519fa9eb776414d6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102244812"
---
* **Если вам не нужны ресурсы, созданные при работе с этим руководством**, вы можете удалить их вместе с экземпляром Azure Digital Twins с помощью команды [az group delete](/cli/azure/group#az-group-delete). При этом будут удалены все ресурсы Azure в группе ресурсов, а также сама группа.
    
    > [!IMPORTANT]
    > Удаление группы ресурсов — процесс необратимый. Группа ресурсов и все содержащиеся в ней ресурсы удаляются без возможности восстановления. Будьте внимательны, чтобы случайно не удалить не ту группу ресурсов или не те ресурсы.
    
    Откройте [Azure Cloud Shell](https://shell.azure.com) и выполните приведенную ниже команду, чтобы удалить группу ресурсов и все ее содержимое.
    
    ```azurecli-interactive
    az group delete --name <your-resource-group>
    ```