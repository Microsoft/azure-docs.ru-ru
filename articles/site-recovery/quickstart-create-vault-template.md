---
title: Краткое руководство. Создание хранилища Служб восстановления Azure с помощью шаблона Azure Resource Manager.
description: Из этого краткого руководства вы узнаете, как с помощью шаблона Azure Resource Manager (шаблона ARM) создать хранилище Служб восстановления Azure.
ms.date: 04/29/2020
ms.topic: quickstart
ms.custom:
- subject-armqs
- mode-arm
ms.openlocfilehash: 11a88b5485ad970802a65af31daccdb30a1c86df
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107533249"
---
# <a name="quickstart-create-a-recovery-services-vault-using-an-arm-template"></a>Краткое руководство. Создание хранилища Служб восстановления с помощью шаблона ARM

В этом кратком руководстве описано, как создать хранилище Служб восстановления Azure с помощью шаблона Azure Resource Manager (шаблона ARM). Служба [Azure Site Recovery](site-recovery-overview.md) помогает реализовать стратегию непрерывности бизнес-процессов и аварийного восстановления (BCDR), чтобы ваши бизнес-приложения оставались подключенными во время запланированных и незапланированных простоев. Site Recovery управляет аварийным восстановлением локальных виртуальных машин и виртуальных машин Azure, включая операции репликации, отработки отказа и восстановления.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-recovery-services-vault-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет активной подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем приступать к работе.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-recovery-services-vault-create/).

:::code language="json" source="~/quickstart-templates/101-recovery-services-vault-create/azuredeploy.json":::

В шаблоне определено два ресурса Azure:

- [Microsoft.RecoveryServices/vaults](/azure/templates/microsoft.recoveryservices/vaults) — создание хранилища.
- [Microsoft.RecoveryServices/vaults/backupstorageconfig](/rest/api/backup/backupresourcestorageconfigs) — настройка параметров избыточности резервного копирования хранилища.

Шаблон включает необязательные параметры для конфигурации резервного копирования хранилища. Хранилище может быть локально избыточным (LRS) или геоизбыточным (GRS). См. сведения о [настройке избыточности хранилища](../backup/backup-create-rs-vault.md#set-storage-redundancy).

Дополнительные шаблоны, связанные с Azure Recovery Services, см. в статье [Шаблоны быстрого запуска Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Recoveryservices&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Развертывание шаблона

Чтобы развернуть шаблон, укажите значения для параметров **Подписка**, **Группа ресурсов** и **Имя хранилища**.

1. Чтобы войти в Azure и открыть шаблон, выберите **Развертывание в Azure**.

   [![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-recovery-services-vault-create%2Fazuredeploy.json)

1. Введите или выберите следующие значения.

   :::image type="content" source="media/quickstart-create-vault-template/create-vault-template.png" alt-text="Шаблон для создания хранилища Служб восстановления.":::

   - **Подписка**. Выберите подписку Azure.
   - **Группа ресурсов.** Выберите существующую группу или щелкните **Создать новую**, чтобы добавить группу.
   - **Расположение.** По умолчанию используется расположение группы ресурсов. Этот параметр становится недоступным после выбора группы ресурсов.
   - **Имя хранилища.** Укажите имя хранилища.
   - **Изменить тип хранилища.** Значение по умолчанию — **false**. Выберите значение **true**, только если необходимо изменить тип хранения в хранилище.
   - **Тип хранения в хранилище.** Значение по умолчанию — **GloballyRedundant**. Если для параметра изменения типа хранилища установлено значение **true**, выберите **LocallyRedundant**.
   - **Расположение.** Функция `[resourceGroup().location]` по умолчанию соответствует расположению группы ресурсов. Чтобы изменить расположение, введите значение, например **westus**.
   - Установите флажок **Я принимаю указанные выше условия**.

1. Чтобы начать развертывание хранилища, нажмите кнопку **Приобрести**. После успешного развертывания отобразится уведомление.

   :::image type="content" source="media/quickstart-create-vault-template/deployment-success.png" alt-text="Развертывание хранилища успешно выполнено.":::

## <a name="validate-the-deployment"></a>Проверка развертывания

Чтобы убедиться, что хранилище создано, используйте Azure CLI или Azure PowerShell.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
echo "Enter the resource group name:" &&
read resourceGroupName &&
echo "Enter the vault name:" &&
read vaultName &&
az backup vault show --name $vaultName --resource-group $resourceGroupName &&
az backup vault backup-properties show --name $vaultName --resource-group $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resouceGroupName = Read-Host -Prompt "Enter the resource group name"
$vaultName = Read-Host -Prompt "Enter the vault name"
$vaultBackupConfig = Get-AzRecoveryServicesVault -Name $vaultName
Get-AzRecoveryServicesVault -ResourceGroupName $resouceGroupName -Name $vaultName
Get-AzRecoveryServicesBackupProperty -Vault $vaultBackupConfig
Write-Host "Press [ENTER] to continue..."
```

---

Следующие выходные данные — это фрагмент данных хранилище:

# <a name="cli"></a>[CLI](#tab/CLI)

```Output
"id": "/subscriptions/<Subscription Id>/resourceGroups/myResourceGroup
         /providers/Microsoft.RecoveryServices/vaults/myVault"
"location": "eastus"
"name": "myVault"
"resourceGroup": "myResourceGroup"

"storageModelType": "GeoRedundant"
"storageType": "GeoRedundant"
"type": "Microsoft.RecoveryServices/vaults/backupstorageconfig"
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```Output
Name              : myVault
Type              : Microsoft.RecoveryServices/vaults
Location          : eastus
ResourceGroupName : myResourceGroup
SubscriptionId    : <Subscription Id>

BackupStorageRedundancy
-----------------------
GeoRedundant
```

---

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете использовать новые ресурсы, никаких действий не требуется. В противном случае вы можете удалить группу ресурсов и хранилище, созданные в ходе работы с этим кратким руководством. Чтобы удалить группу ресурсов и входящие в нее ресурсы с помощью Azure CLI или Azure PowerShell, сделайте следующее:

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
echo "Enter the resource group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resouceGroupName = Read-Host -Prompt "Enter the resource group name"
Remove-AzResourceGroup -Name $resouceGroupName
Write-Host "Press [ENTER] to continue..."
```

---

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как создать хранилище Служб восстановления. Чтобы узнать больше об аварийном восстановлении, переходите к следующему краткому руководству.

> [!div class="nextstepaction"]
> [Настройка аварийного восстановления](azure-to-azure-quickstart.md)
