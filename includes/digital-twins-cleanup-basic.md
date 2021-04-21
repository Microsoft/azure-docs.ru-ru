---
author: baanders
description: включаемый файл для удаления экземпляра Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 2/4/2021
ms.author: baanders
ms.openlocfilehash: 0d8cc30c0511098caf7b6c47d7f7bd400dc32f1b
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107800116"
---
* **Если вам не нужны ресурсы, созданные при работе с этим руководством**, вы можете удалить их вместе с экземпляром Azure Digital Twins с помощью команды [az group delete](/cli/azure/group#az_group_delete). При этом будут удалены все ресурсы Azure в группе ресурсов, а также сама группа.
    
    > [!IMPORTANT]
    > Удаление группы ресурсов — процесс необратимый. Группа ресурсов и все содержащиеся в ней ресурсы удаляются без возможности восстановления. Будьте внимательны, чтобы случайно не удалить не ту группу ресурсов или не те ресурсы.
    
    Откройте [Azure Cloud Shell](https://shell.azure.com) и выполните приведенную ниже команду, чтобы удалить группу ресурсов и все ее содержимое.
    
    ```azurecli-interactive
    az group delete --name <your-resource-group>
    ```