---
title: Блокировка ресурсов для предотвращения изменений
description: Запрещает пользователям обновлять или удалять ресурсы Azure, применяя блокировку для всех пользователей и ролей.
ms.topic: conceptual
ms.date: 03/09/2021
ms.custom: devx-track-azurecli
ms.openlocfilehash: 28c31681b8fbe981cd51db294c91276dfd65d71f
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102619177"
---
# <a name="lock-resources-to-prevent-unexpected-changes"></a>Блокировка ресурсов для предотвращения непредвиденных изменений

Администратор может заблокировать подписку, группу ресурсов или ресурс, чтобы другие пользователи в Организации не могли случайно удалить или изменить критически важные ресурсы. Блокировка переопределяет любые разрешения, которые может иметь пользователь.

Можно установить уровень блокировки **CanNotDelete** или **ReadOnly**. На портале блокировки называются **Удаление** и **Только для чтения**, соответственно.

* **CanNotDelete** означает, что авторизованные пользователи смогут читать и изменять ресурс, но не смогут его удалить.
* **ReadOnly** означает, что авторизованные пользователи смогут читать ресурс, но не смогут его удалить или обновить. Применение этой блокировки подобно ограничению авторизованных пользователей с помощью разрешений, предоставляемых для роли **Читатель**.

## <a name="how-locks-are-applied"></a>Применение блокировок

При применении блокировки к родительской области все ресурсы в этой области наследуют ту же блокировку. Даже ресурсы, добавляемые позже, наследуют блокировку от родительского ресурса. Приоритет имеет наиболее строгая блокировка.

В отличие от управления доступом на основе ролей блокировки управления используются для применения ограничения для всех пользователей и ролей. Дополнительные сведения о настройке разрешений для пользователей и ролей см. в статье [Управление доступом на основе ролей в Azure (Azure RBAC)](../../role-based-access-control/role-assignments-portal.md).

Блокировки Resource Manager применяются только к операциям, выполняемым в плоскости управления, которая включает в себя операции, передаваемые на `https://management.azure.com`. Эти блокировки не мешают ресурсам осуществлять свои собственные операции. Ограничиваются изменения ресурсов, но не операции с ними. Например, блокировка только для чтения на логическом сервере базы данных SQL не позволяет удалить или изменить сервер. Это не мешает создавать, обновлять или удалять данные в базах данных на этом сервере. Транзакции с данными разрешены, так как эти операции не передаются на `https://management.azure.com`.

## <a name="considerations-before-applying-locks"></a>Прежде чем применять блокировки, учтите следующее.

Применение блокировок может привести к непредвиденным результатам, так как некоторые операции, которые, как кажется на первый взгляд, не изменяют ресурс, на самом деле требуют действий, выполнение которых невозможно из-за блокировки. Блокировки предотвращают любые операции, требующие запроса POST к Azure Resource Manager API. Вот несколько распространенных примеров операций, которые блокируются в результате блокировки.

* Блокировка только для чтения в **учетной записи хранения** запрещает пользователям выводить список ключей учетной записи. Операция с [ключами списка](/rest/api/storagerp/storageaccounts/listkeys) службы хранилища Azure обрабатывается с помощью запроса POST, чтобы защитить доступ к ключам учетной записи, которые обеспечивают полный доступ к данным в учетной записи хранения. Если для учетной записи хранения настроена блокировка только для чтения, пользователи, не обладающие ключами учетной записи, должны использовать учетные данные Azure AD для доступа к данным BLOB-объектов или очередей. Блокировка только для чтения также не позволяет назначать роли RBAC Azure, областью действия которых является учетная запись хранения или контейнер данных (контейнер BLOB-объектов или очередь).

* Блокировка "не удается удалить" в **учетной записи хранения** не препятствует удалению или изменению данных в этой учетной записи. Этот тип блокировки защищает учетную запись хранения только от удаления и не защищает данные большого двоичного объекта, очереди, таблицы или файла в этой учетной записи хранения. 

* Блокировка только для чтения в **учетной записи хранения** не мешает удалению или изменению данных в этой учетной записи. Этот тип блокировки защищает учетную запись хранения от удаления или изменения, а также не защищает данные большого двоичного объекта, очереди, таблицы или файла в этой учетной записи хранения. 

* Кроме того, применение блокировки "только для чтения" к ресурсу **службы приложений** не позволит обозревателю сервера Visual Studio отображать файлы для ресурса, так как для их взаимодействия требуется доступ на запись.

* Блокировка "только для чтения" **группы ресурсов**, которая содержит **виртуальную машину**, не позволяет пользователям запускать или перезапускать виртуальную машину. Для этих операций требуется запрос POST.

* Блокировка "не удается удалить" в **группе ресурсов** не Azure Resource Manager [Автоматическое удаление развертываний](../templates/deployment-history-deletions.md) в журнале. Если вы достигли 800 развертываний в журнале, произойдет сбой развертывания.

* Блокировка "не удается удалить" в **группе ресурсов**, созданной **службой Azure Backup**, приводит к сбою резервного копирования. Служба поддерживает не более 18 точек восстановления. При наличии блокировки служба резервного копирования не может удалять точки восстановления. Более подробные сведения см. в разделе [Часто задаваемые вопросы о резервном копировании виртуальных машин Azure](../../backup/backup-azure-vm-backup-faq.yml).

* Блокировка "только для чтения" в **подписке** не позволяет **Azure Advisor** работать корректно. Advisor не удается сохранять результаты запросов.

## <a name="who-can-create-or-delete-locks"></a>Кто может создавать или удалять блокировки

Для создания или удаления блокировок управления необходим доступ к действию `Microsoft.Authorization/*` или `Microsoft.Authorization/locks/*`. Из встроенных ролей эти действия предоставляются только **владельцу** и **администратору доступа пользователей**.

## <a name="managed-applications-and-locks"></a>Управляемые приложения и блокировки

Некоторые службы Azure, например Azure Databricks, используют для реализации этой службы [управляемые приложения](../managed-applications/overview.md). В этом случае служба создает две группы ресурсов. Одна группа ресурсов содержит обзор службы и не заблокирована. Другая группа ресурсов содержит инфраструктуру для этой службы и заблокирована.

При попытке удалить инфраструктурную группу ресурсов отображается ошибка с уведомлением о том, что группа ресурсов заблокирована. Если попытаться удалить блокировку инфраструктурной группы ресурсов, отображается ошибка с уведомлением о том, что удалить блокировку невозможно, так как она принадлежит системному приложению.

Вместо этого необходимо удалить службу, в результате чего будет удалена и инфраструктурная группа ресурсов.

В случае с управляемыми приложениями выбирайте развернутую вами службу.

![Выберите службу](./media/lock-resources/select-service.png)

Обратите внимание, что служба содержит ссылку на **группу управляемых ресурсов**. Эта группа ресурсов содержит инфраструктуру и заблокирована. Удалить ее напрямую невозможно.

![Показать управляемую группу](./media/lock-resources/show-managed-group.png)

Чтобы удалить для службы все, включая заблокированную инфраструктурную группу ресурсов, выберите **Удалить** для этой службы.

![Удаление службы](./media/lock-resources/delete-service.png)

## <a name="configure-locks"></a>Настройка блокировок

### <a name="portal"></a>Портал

[!INCLUDE [resource-manager-lock-resources](../../../includes/resource-manager-lock-resources.md)]

### <a name="arm-template"></a>Шаблон ARM

При использовании шаблона Azure Resource Manager (шаблон ARM) для развертывания блокировки необходимо помнить об области блокировки и области развертывания. Чтобы применить блокировку в области развертывания, например заблокировать группу ресурсов или подписку, не устанавливайте свойство Scope. При блокировке ресурса в области развертывания задайте свойство Scope.

Следующий шаблон применяет блокировку к группе ресурсов, в которой она развернута. Обратите внимание, что в ресурсе блокировки нет свойства Scope, так как область блокировки соответствует области развертывания. Этот шаблон развертывается на уровне группы ресурсов.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {  
    },
    "resources": [
        {
            "type": "Microsoft.Authorization/locks",
            "apiVersion": "2016-09-01",
            "name": "rgLock",
            "properties": {
                "level": "CanNotDelete",
                "notes": "Resource Group should not be deleted."
            }
        }
    ]
}
```

Чтобы создать группу ресурсов и заблокировать ее, разверните следующий шаблон на уровне подписки.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "rgName": {
            "type": "string"
        },
        "rgLocation": {
            "type": "string"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Resources/resourceGroups",
            "apiVersion": "2019-10-01",
            "name": "[parameters('rgName')]",
            "location": "[parameters('rgLocation')]",
            "properties": {}
        },
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2020-06-01",
            "name": "lockDeployment",
            "resourceGroup": "[parameters('rgName')]",
            "dependsOn": [
                "[resourceId('Microsoft.Resources/resourceGroups/', parameters('rgName'))]"
            ],
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "type": "Microsoft.Authorization/locks",
                            "apiVersion": "2016-09-01",
                            "name": "rgLock",
                            "properties": {
                                "level": "CanNotDelete",
                                "notes": "Resource group and its resources should not be deleted."
                            }
                        }
                    ],
                    "outputs": {}
                }
            }
        }
    ],
    "outputs": {}
}
```

При применении блокировки к **ресурсу** в группе ресурсов добавьте свойство Scope. В поле область задайте имя ресурса для блокировки.

В следующем примере показан шаблон, который позволяет создать план обслуживания приложений, веб-сайт и блокировку на веб-сайте. Область блокировки задается для веб-сайта.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "hostingPlanName": {
      "type": "string"
    },
    "location": {
        "type": "string",
        "defaultValue": "[resourceGroup().location]"
    }
  },
  "variables": {
    "siteName": "[concat('ExampleSite', uniqueString(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Web/serverfarms",
      "apiVersion": "2020-06-01",
      "name": "[parameters('hostingPlanName')]",
      "location": "[parameters('location')]",
      "sku": {
        "tier": "Free",
        "name": "f1",
        "capacity": 0
      },
      "properties": {
        "targetWorkerCount": 1
      }
    },
    {
      "type": "Microsoft.Web/sites",
      "apiVersion": "2020-06-01",
      "name": "[variables('siteName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', parameters('hostingPlanName'))]"
      ],
      "properties": {
        "serverFarmId": "[parameters('hostingPlanName')]"
      }
    },
    {
      "type": "Microsoft.Authorization/locks",
      "apiVersion": "2016-09-01",
      "name": "siteLock",
      "scope": "[concat('Microsoft.Web/sites/', variables('siteName'))]",
      "dependsOn": [
        "[resourceId('Microsoft.Web/sites', variables('siteName'))]"
      ],
      "properties": {
        "level": "CanNotDelete",
        "notes": "Site should not be deleted."
      }
    }
  ]
}
```

### <a name="azure-powershell"></a>Azure PowerShell

Вы можете заблокировать развернутые ресурсы с помощью Azure PowerShell, выполнив команду [New-AzResourceLock](/powershell/module/az.resources/new-azresourcelock).

Чтобы заблокировать ресурс, укажите имя и тип ресурса, а также имя группы ресурсов.

```azurepowershell-interactive
New-AzResourceLock -LockLevel CanNotDelete -LockName LockSite -ResourceName examplesite -ResourceType Microsoft.Web/sites -ResourceGroupName exampleresourcegroup
```

Чтобы заблокировать группу ресурсов, укажите ее имя.

```azurepowershell-interactive
New-AzResourceLock -LockName LockGroup -LockLevel CanNotDelete -ResourceGroupName exampleresourcegroup
```

Для получения информации о блокировке используйте команду [Get-AzResourceLock](/powershell/module/az.resources/get-azresourcelock). Чтобы получить все блокировки в подписке, выполните следующую команду:

```azurepowershell-interactive
Get-AzResourceLock
```

Чтобы получить все блокировки для ресурса, выполните следующую команду:

```azurepowershell-interactive
Get-AzResourceLock -ResourceName examplesite -ResourceType Microsoft.Web/sites -ResourceGroupName exampleresourcegroup
```

Чтобы получить все блокировки для группы ресурсов, выполните следующую команду:

```azurepowershell-interactive
Get-AzResourceLock -ResourceGroupName exampleresourcegroup
```

Чтобы удалить блокировку для ресурса, используйте:

```azurepowershell-interactive
$lockId = (Get-AzResourceLock -ResourceGroupName exampleresourcegroup -ResourceName examplesite -ResourceType Microsoft.Web/sites).LockId
Remove-AzResourceLock -LockId $lockId
```

Чтобы удалить блокировку для группы ресурсов, используйте:

```azurepowershell-interactive
$lockId = (Get-AzResourceLock -ResourceGroupName exampleresourcegroup).LockId
Remove-AzResourceLock -LockId $lockId
```

### <a name="azure-cli"></a>Azure CLI

Вы можете заблокировать развернутые ресурсы с помощью Azure CLI, выполнив команду [az lock create](/cli/azure/lock#az-lock-create).

Чтобы заблокировать ресурс, укажите имя и тип ресурса, а также имя группы ресурсов.

```azurecli
az lock create --name LockSite --lock-type CanNotDelete --resource-group exampleresourcegroup --resource-name examplesite --resource-type Microsoft.Web/sites
```

Чтобы заблокировать группу ресурсов, укажите ее имя.

```azurecli
az lock create --name LockGroup --lock-type CanNotDelete --resource-group exampleresourcegroup
```

Чтобы получить сведения о блокировке, выполните команду [az lock list](/cli/azure/lock#az-lock-list). Чтобы получить все блокировки в подписке, выполните следующую команду:

```azurecli
az lock list
```

Чтобы получить все блокировки для ресурса, выполните следующую команду:

```azurecli
az lock list --resource-group exampleresourcegroup --resource-name examplesite --namespace Microsoft.Web --resource-type sites --parent ""
```

Чтобы получить все блокировки для группы ресурсов, выполните следующую команду:

```azurecli
az lock list --resource-group exampleresourcegroup
```

Чтобы удалить блокировку для ресурса, используйте:

```azurecli
lockid=$(az lock show --name LockSite --resource-group exampleresourcegroup --resource-type Microsoft.Web/sites --resource-name examplesite --output tsv --query id)
az lock delete --ids $lockid
```

Чтобы удалить блокировку для группы ресурсов, используйте:

```azurecli
lockid=$(az lock show --name LockSite --resource-group exampleresourcegroup  --output tsv --query id)
az lock delete --ids $lockid
```

### <a name="rest-api"></a>REST API

Вы можете заблокировать развернутые ресурсы с помощью [REST API для блокировок управления](/rest/api/resources/managementlocks). REST API позволяет создавать и удалять блокировки и получать информацию о существующих блокировках.

Чтобы создать блокировку, выполните следующую команду:

```http
PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/locks/{lock-name}?api-version={api-version}
```

Областью может быть подписка, группа ресурсов или ресурс. Вы можете назначить блокировке любое имя. В качестве версии API используйте **2016-09-01**.

В запросе включите объект JSON, который задает свойства блокировки.

```json
{
  "properties": {
  "level": "CanNotDelete",
  "notes": "Optional text notes."
  }
}
```

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о том, как логично упорядочить свои ресурсы, см. в разделе [Использование тегов для организации ресурсов](tag-resources.md).
* Ограничения и соглашения можно применять внутри подписки с помощью настраиваемых политик. Дополнительные сведения см. в статье [Что такое служба "Политика Azure"](../../governance/policy/overview.md).
* Инструкции по использованию Resource Manager для эффективного управления подписками в организациях см. в статье [Корпоративный каркас Azure: рекомендуемая система управления подписками](/azure/architecture/cloud-adoption-guide/subscription-governance).
