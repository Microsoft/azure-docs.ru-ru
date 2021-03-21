---
title: Смена предохранителя TDE (PowerShell & Azure CLI)
titleSuffix: Azure SQL Database & Azure Synapse Analytics
description: Узнайте, как поворачивать средство защиты прозрачное шифрование данных (TDE) для сервера в Azure, используемого базой данных SQL Azure и Azure синапсе Analytics, с помощью PowerShell и Azure CLI.
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: seo-lt-2019 sqldbrb=1, devx-track-azurecli
ms.devlang: ''
ms.topic: how-to
author: jaszymas
ms.author: jaszymas
ms.reviewer: vanto
ms.date: 03/12/2019
ms.openlocfilehash: 37931aee24e2dbcab03bca400d58f236601c87e5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93321426"
---
# <a name="rotate-the-transparent-data-encryption-tde-protector"></a>Вращение средства защиты прозрачное шифрование данных (TDE)
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]


В этой статье описывается смена ключа для [сервера](logical-servers.md) с помощью средства защиты TDE от Azure Key Vault. Вращение логического предохранителя TDE для сервера означает переключение на новый асимметричный ключ, защищающий базы данных на сервере. Смена ключа является операцией в сети и может занять несколько секунд, так как при этом выполняется только расшифровка и повторное шифрование ключа шифрования данных базы данных, а не всей базы данных.

В этом руководстве описаны два метода смены предохранителя TDE на сервере.

> [!NOTE]
> Приостановленный выделенный пул SQL в Azure синапсе Analytics необходимо возобновить до смены ключа.

> [!IMPORTANT]
> Не удаляйте предыдущие версии ключа после смены. При смене ключа обычно сохраняются данные, зашифрованные прежними версиями ключа, например резервные копии базы данных.

## <a name="prerequisites"></a>Предварительные условия

- В этом пошаговом руководство предполагается, что вы уже используете ключ из Azure Key Vault в качестве предохранителя TDE для базы данных SQL Azure или Azure синапсе Analytics. Изучите статью о [прозрачном шифровании данных с поддержкой создания собственных ключей](transparent-data-encryption-byok-overview.md).
- Необходимо установить и запустить Azure PowerShell.
- [Рекомендуется, но необязательно.] Предварительно создайте материал ключа для предохранителя TDE в аппаратном модуле безопасности (HSM) или локальном хранилище ключей, а затем импортируйте этот материал в Azure Key Vault. Дополнительные сведения вы найдете в [инструкциях по использованию аппаратного модуля безопасности (HSM) и Key Vault](../../key-vault/general/overview.md).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Инструкции по установке модуля Az см. в статье об [установке Azure PowerShell](/powershell/azure/install-az-ps). Конкретные командлеты см. в разделе [AzureRM. SQL](/powershell/module/AzureRM.Sql/).

> [!IMPORTANT]
> Модуль PowerShell Azure Resource Manager по-прежнему поддерживается, но вся будущая разработка сосредоточена на модуле Az.Sql. Исправления ошибок для модуля AzureRM будут продолжать выпускаться как минимум до декабря 2020 г.  Аргументы команд в модулях Az и AzureRm практически идентичны. Дополнительные сведения о совместимости см. в статье [Знакомство с новым модулем Az для Azure PowerShell](/powershell/azure/new-azureps-module-az).

# <a name="the-azure-cli"></a>[CLI Azure.](#tab/azure-cli)

Сведения об установке см. [в разделе установка Azure CLI](/cli/azure/install-azure-cli).

* * *

## <a name="manual-key-rotation"></a>Смена ключей вручную

При смене ключей вручную используются следующие команды, чтобы добавить совершенно новый ключ, который может находиться под новым именем ключа или даже в другом хранилище ключей. Использование этого подхода поддерживает добавление аналогичного ключа в разные хранилища ключей для поддержки сценариев высокой доступности и географического аварийного восстановления.

> [!NOTE]
> Общая длина имени хранилища ключей и имени ключа не может превышать 94 символа.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Используйте командлеты [Add-азкэйваулткэй](/powershell/module/az.keyvault/Add-AzKeyVaultKey), [Add-азсклсерверкэйваулткэй](/powershell/module/az.sql/add-azsqlserverkeyvaultkey)и [Set-азсклсервертранспарентдатаенкриптионпротектор](/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector) .

```powershell
# add a new key to Key Vault
Add-AzKeyVaultKey -VaultName <keyVaultName> -Name <keyVaultKeyName> -Destination <hardwareOrSoftware>

# add the new key from Key Vault to the server
Add-AzSqlServerKeyVaultKey -KeyId <keyVaultKeyId> -ServerName <logicalServerName> -ResourceGroup <SQLDatabaseResourceGroupName>
  
# set the key as the TDE protector for all resources under the server
Set-AzSqlServerTransparentDataEncryptionProtector -Type AzureKeyVault -KeyId <keyVaultKeyId> `
   -ServerName <logicalServerName> -ResourceGroup <SQLDatabaseResourceGroupName>
```

# <a name="the-azure-cli"></a>[CLI Azure.](#tab/azure-cli)

Используйте команду [AZ keyvault Key Create](/cli/azure/keyvault/key#az-keyvault-key-create), [AZ SQL Server Key Create](/cli/azure/sql/server/key#az-sql-server-key-create)и [AZ SQL Server TDE-Key Set](/cli/azure/sql/server/tde-key#az-sql-server-tde-key-set) .

```azurecli
# add a new key to Key Vault
az keyvault key create --name <keyVaultKeyName> --vault-name <keyVaultName> --protection <hsmOrSoftware>

# add the new key from Key Vault to the server
az sql server key create --kid <keyVaultKeyId> --resource-group <SQLDatabaseResourceGroupName> --server <logicalServerName>

# set the key as the TDE protector for all resources under the server
az sql server tde-key set --server-key-type AzureKeyVault --kid <keyVaultKeyId> --resource-group <SQLDatabaseResourceGroupName> --server <logicalServerName>
```

* * *

## <a name="switch-tde-protector-mode"></a>Переключение режима предохранителя TDE

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

- Чтобы переключить средство защиты TDE с управляемого Майкрософт на режим BYOK, используйте командлет [Set-азсклсервертранспарентдатаенкриптионпротектор](/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector) .

   ```powershell
   Set-AzSqlServerTransparentDataEncryptionProtector -Type AzureKeyVault `
       -KeyId <keyVaultKeyId> -ServerName <logicalServerName> -ResourceGroup <SQLDatabaseResourceGroupName>
   ```

- Чтобы переключить средство защиты TDE из режима BYOK на управляемые корпорацией Майкрософт, используйте командлет [Set-азсклсервертранспарентдатаенкриптионпротектор](/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector) .

   ```powershell
   Set-AzSqlServerTransparentDataEncryptionProtector -Type ServiceManaged `
       -ServerName <logicalServerName> -ResourceGroup <SQLDatabaseResourceGroupName>
   ```

# <a name="the-azure-cli"></a>[CLI Azure.](#tab/azure-cli)

В следующих примерах используется [AZ SQL Server TDE-Key Set](/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector).

- Чтобы переключить средство защиты TDE с управляемого Майкрософт на режим BYOK,

   ```azurecli
   az sql server tde-key set --server-key-type AzureKeyVault --kid <keyVaultKeyId> --resource-group <SQLDatabaseResourceGroupName> --server <logicalServerName>
   ```

- Чтобы переключить средство защиты TDE из режима BYOK на управляемые корпорацией Майкрософт,

   ```azurecli
   az sql server tde-key set --server-key-type ServiceManaged --resource-group <SQLDatabaseResourceGroupName> --server <logicalServerName>
   ```

* * *

## <a name="next-steps"></a>Дальнейшие действия

- В случае угрозы безопасности Узнайте, как удалить потенциально скомпрометированный предохранитель TDE: [удалите потенциально скомпрометированный ключ](transparent-data-encryption-byok-remove-tde-protector.md).

- Приступая к работе с Azure Key Vault интеграцией и создание собственных ключей поддержки TDE: [включите TDE с помощью собственного ключа из Key Vault с помощью PowerShell](transparent-data-encryption-byok-configure.md).