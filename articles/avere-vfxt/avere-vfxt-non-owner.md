---
title: Обходной путь для тех, кто не является владельцем решения Avere vFXT для Azure
description: Обходной путь, предоставляющий пользователям без разрешений владельца подписки доступ к развертыванию Avere vFXT для Azure
author: ekpgh
ms.service: avere-vfxt
ms.topic: how-to
ms.date: 12/19/2019
ms.author: rohogue
ms.openlocfilehash: 0d9b1060ee35af6cbc2e1b95b0f7813072c52d2e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85505381"
---
# <a name="authorize-non-owners-to-deploy-avere-vfxt"></a>Разрешение на развертывание Avere vFXT для пользователей без роли владельца

Здесь приведены инструкции, которые позволяют пользователям без привилегий владельца подписки создавать систему Avere vFXT для Azure.

(Рекомендуемый способ развертывания системы Avere vFXT — попросить пользователя с правами владельца выполнить действия по созданию, как описано в статье [Подготовка к созданию Avere vFXT](avere-vfxt-prereqs.md).)  

Обходной путь заключается в создании дополнительной роли доступа, которая предоставляет пользователям необходимые разрешения для установки кластера. Эта роль должна быть создана владельцем подписки и назначена им соответствующим пользователям.

Владелец подписки также должен [принять условия использования](avere-vfxt-prereqs.md) для образа Avere vFXT из marketplace.

> [!IMPORTANT]
> Все эти шаги должны выполняться пользователем с правами владельца в подписке, которая будет использоваться для кластера.

1. Скопируйте эти строки и сохраните их в файле (например, `averecreatecluster.json`). Используйте идентификатор подписки в операторе `AssignableScopes`.

   ```json
   {
       "AssignableScopes": ["/subscriptions/<SUBSCRIPTION_ID>"],
       "Name": "avere-create-cluster",
       "IsCustom": "true"
       "Description": "Can create Avere vFXT clusters",
       "NotActions": [],
       "Actions": [
           "Microsoft.Authorization/*/read",
           "Microsoft.Authorization/roleAssignments/*",
           "Microsoft.Authorization/roleDefinitions/*",
           "Microsoft.Compute/*/read",
           "Microsoft.Compute/availabilitySets/*",
           "Microsoft.Compute/virtualMachines/*",
           "Microsoft.Network/*/read",
           "Microsoft.Network/networkInterfaces/*",
           "Microsoft.Network/routeTables/write",
           "Microsoft.Network/routeTables/delete",
           "Microsoft.Network/routeTables/routes/delete",
           "Microsoft.Network/virtualNetworks/subnets/join/action",
           "Microsoft.Network/virtualNetworks/subnets/read",

           "Microsoft.Resources/subscriptions/resourceGroups/read",
           "Microsoft.Resources/subscriptions/resourceGroups/resources/read",
           "Microsoft.Storage/*/read",
           "Microsoft.Storage/storageAccounts/listKeys/action"
       ],
   }
   ```

1. Чтобы создать роль, выполните эту команду:

   `az role definition create --role-definition <PATH_TO_FILE>`

    Пример

    ```azurecli
    az role definition create --role-definition ./averecreatecluster.json
    ```

1. Назначьте эту роль пользователю, который будет создавать кластер:

   `az role assignment create --assignee <USERNAME> --scope /subscriptions/<SUBSCRIPTION_ID> --role 'avere-create-cluster'`

По завершении этого процесса роль назначает любому пользователю следующие разрешения для подписки:

* создание и настройка сетевой инфраструктуры;
* создание контроллера кластера;
* выполнение сценариев создания кластера из контроллера кластера.
