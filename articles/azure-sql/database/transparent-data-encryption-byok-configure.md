---
title: Включение SQL TDE с Azure Key Vault
titleSuffix: Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics
description: Узнайте, как настроить базу данных SQL Azure и Azure синапсе Analytics, чтобы приступить к использованию прозрачное шифрование данных (TDE) для шифрования с неактивных данными с помощью PowerShell или Azure CLI.
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.custom: seo-lt-2019 sqldbrb=1, devx-track-azurecli
ms.devlang: ''
ms.topic: how-to
author: jaszymas
ms.author: jaszymas
ms.reviewer: vanto
ms.date: 03/12/2019
ms.openlocfilehash: 610f0a9692c18afbd7bb446959b09bac14d6f629
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102507625"
---
# <a name="powershell-and-the-azure-cli-enable-transparent-data-encryption-with-customer-managed-key-from-azure-key-vault"></a>PowerShell и Azure CLI: включение прозрачное шифрование данных с управляемым клиентом ключом из Azure Key Vault
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

В этой статье объясняется, как использовать ключ из Azure Key Vault для прозрачное шифрование данных (TDE) в базе данных SQL Azure или Azure синапсе Analytics. Чтобы получить дополнительные сведения об интеграции Azure Key Vault и поддержке при создании собственных ключей для TDE, см. статью [Прозрачное шифрование данных в Azure SQL. Поддержка создания собственных ключей](transparent-data-encryption-byok-overview.md).

> [!NOTE] 
> Теперь SQL Azure поддерживает использование ключа RSA, хранящегося в управляемом HSM, как средство защиты TDE. Эта функция доступна в **общедоступной предварительной версии**. Azure Key Vault управляемым HSM — это полностью управляемая высокодоступная облачная служба с одним клиентом, которая позволяет защищать криптографические ключи для облачных приложений, используя проверенный HSM уровня 2 FIPS 140-2. Дополнительные сведения об [управляемых HSM](../../key-vault/managed-hsm/index.yml).

## <a name="prerequisites-for-powershell"></a>Предварительные требования для PowerShell

- Подписка Azure и права администратора для нее.
- [Рекомендуется, но необязательно.] Аппаратный модуль безопасности (HSM) или локальное хранилище ключей для создания локальной копии материала ключа для предохранителя TDE.
- Необходимо установить и запустить Azure PowerShell.
- Создайте Azure Key Vault и ключи для TDE.
  - [Инструкции по использованию аппаратного модуля безопасности (HSM) и Key Vault](../../key-vault/keys/hsm-protected-keys.md)
    - Чтобы использовать хранилище ключей для TDE, обязательно настройте для него следующее свойство:
  - [обратимое удаление](../../key-vault/general/soft-delete-overview.md) и очистка защиты
- Чтобы использовать ключ для TDE, обязательно настройте для него следующие атрибуты:
  - без даты окончания срока действия;
  - не отключено;
  - возможность выполнять операции *получения*, *упаковки ключа* и *распаковки ключа*.
- **(Предварительная версия)** Чтобы использовать управляемый ключ HSM, следуйте инструкциям по [созданию и активации управляемого модуля HSM с помощью Azure CLI](../../key-vault/managed-hsm/quick-create-cli.md)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Инструкции по установке модуля Az см. в статье об [установке Azure PowerShell](/powershell/azure/install-az-ps). Конкретные командлеты см. в разделе [AzureRM. SQL](/powershell/module/AzureRM.Sql/).

Дополнительные сведения о Key Vault см. в [инструкциях по PowerShell из Key Vault](../../key-vault/secrets/quick-create-powershell.md) и [об использовании Key Vault обратимого удаления с помощью PowerShell](../../key-vault/general/key-vault-recovery.md).

> [!IMPORTANT]
> Модуль PowerShell Azure Resource Manager по-прежнему поддерживается, но вся будущая разработка сосредоточена на модуле Az.Sql. Исправления ошибок для модуля AzureRM будут продолжать выпускаться как минимум до декабря 2020 г.  Аргументы команд в модулях Az и AzureRm практически идентичны. Дополнительные сведения о совместимости см. в статье [Знакомство с новым модулем Az для Azure PowerShell](/powershell/azure/new-azureps-module-az).

## <a name="assign-an-azure-active-directory-azure-ad-identity-to-your-server"></a>Назначьте серверу удостоверение Azure Active Directory (Azure AD)

При наличии существующего [сервера](logical-servers.md)используйте следующую команду, чтобы добавить удостоверение Azure Active Directory (Azure AD) на сервер:

   ```powershell
   $server = Set-AzSqlServer -ResourceGroupName <SQLDatabaseResourceGroupName> -ServerName <LogicalServerName> -AssignIdentity
   ```

При создании сервера используйте командлет [New-азсклсервер](/powershell/module/az.sql/new-azsqlserver) с тегом-Identity, чтобы добавить удостоверение Azure AD во время создания сервера:

   ```powershell
   $server = New-AzSqlServer -ResourceGroupName <SQLDatabaseResourceGroupName> -Location <RegionName> `
       -ServerName <LogicalServerName> -ServerVersion "12.0" -SqlAdministratorCredentials <PSCredential> -AssignIdentity
   ```

## <a name="grant-key-vault-permissions-to-your-server"></a>Предоставление серверу разрешений на доступ к Key Vault

Используйте командлет [Set-азкэйваултакцессполици](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy) , чтобы предоставить серверу доступ к хранилищу ключей, прежде чем использовать ключ из него для TDE.

   ```powershell
   Set-AzKeyVaultAccessPolicy -VaultName <KeyVaultName> `
       -ObjectId $server.Identity.PrincipalId -PermissionsToKeys get, wrapKey, unwrapKey
   ```
Чтобы добавить разрешения для сервера на управляемый модуль HSM, добавьте на сервер локальную роль RBAC "шифрование криптографической службы HSM". Это позволит серверу выполнять операции получения, переноса ключей и распаковки ключей в управляемых HSM.
[Инструкции по подготовке доступа сервера к управляемому HSM](../../key-vault/managed-hsm/role-management.md)

## <a name="add-the-key-vault-key-to-the-server-and-set-the-tde-protector"></a>Добавление на сервер ключа из Key Vault и настройка предохранителя TDE

- Используйте командлет [Get-азкэйваулткэй](/powershell/module/az.keyvault/get-azkeyvaultkey) , чтобы получить идентификатор ключа из хранилища ключей.
- Используйте командлет [Add-азсклсерверкэйваулткэй](/powershell/module/az.sql/add-azsqlserverkeyvaultkey) , чтобы добавить ключ из Key Vault на сервер.
- Используйте командлет [Set-азсклсервертранспарентдатаенкриптионпротектор](/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector) , чтобы задать ключ в качестве предохранителя TDE для всех ресурсов сервера.
- Используйте командлет [Get-азсклсервертранспарентдатаенкриптионпротектор](/powershell/module/az.sql/get-azsqlservertransparentdataencryptionprotector) , чтобы убедиться, что средство защиты TDE настроено как предназначенное.

> [!NOTE]
> **(Предварительная версия)** Для управляемых ключей HSM используйте команду AZ. SQL 2.11.1 версии PowerShell.

> [!NOTE]
> Общая длина имени хранилища ключей и имени ключа не может превышать 94 символа.

> [!TIP]
> Пример идентификатора KeyId из Key Vault: <br/>https://contosokeyvault.vault.azure.net/keys/Key1/1a1a2b2b3c3c4d4d5e5e6f6f7g7g8h8h
>
> Пример KeyId из управляемого модуля HSM:<br/>https://contosoMHSM.managedhsm.azure.net/keys/myrsakey

```powershell
# add the key from Key Vault to the server
Add-AzSqlServerKeyVaultKey -ResourceGroupName <SQLDatabaseResourceGroupName> -ServerName <LogicalServerName> -KeyId <KeyVaultKeyId>

# set the key as the TDE protector for all resources under the server
Set-AzSqlServerTransparentDataEncryptionProtector -ResourceGroupName <SQLDatabaseResourceGroupName> -ServerName <LogicalServerName> `
   -Type AzureKeyVault -KeyId <KeyVaultKeyId>

# confirm the TDE protector was configured as intended
Get-AzSqlServerTransparentDataEncryptionProtector -ResourceGroupName <SQLDatabaseResourceGroupName> -ServerName <LogicalServerName>
```

## <a name="turn-on-tde"></a>Включение прозрачного шифрования данных

Чтобы включить TDE, используйте командлет [Set-азсклдатабасетранспарентдатаенкриптион](/powershell/module/az.sql/set-azsqldatabasetransparentdataencryption) .

```powershell
Set-AzSqlDatabaseTransparentDataEncryption -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -ServerName <LogicalServerName> -DatabaseName <DatabaseName> -State "Enabled"
```

Теперь для базы данных или хранилища данных включено прозрачное шифрование данных с ключом шифрования из Key Vault.

## <a name="check-the-encryption-state-and-encryption-activity"></a>Проверка состояния и активности шифрования

Используйте [Get-азсклдатабасетранспарентдатаенкриптион](/powershell/module/az.sql/get-azsqldatabasetransparentdataencryption) , чтобы получить состояние шифрования и [Get-азсклдатабасетранспарентдатаенкриптионактивити](/powershell/module/az.sql/get-azsqldatabasetransparentdataencryptionactivity) , чтобы проверить ход шифрования для базы данных или хранилища данных.

```powershell
# get the encryption state
Get-AzSqlDatabaseTransparentDataEncryption -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -ServerName <LogicalServerName> -DatabaseName <DatabaseName> `

# check the encryption progress for a database or data warehouse
Get-AzSqlDatabaseTransparentDataEncryptionActivity -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -ServerName <LogicalServerName> -DatabaseName <DatabaseName>  
```

# <a name="the-azure-cli"></a>[CLI Azure.](#tab/azure-cli)

Чтобы установить требуемую версию Azure CLI (версии 2,0 или более поздней) и подключиться к подписке Azure, см. статью [Установка и настройка Межплатформенного Command-Lineного интерфейса azure 2,0](/cli/azure/install-azure-cli).

Дополнительные сведения о Key Vault см. в статьях [управление Key Vault с помощью интерфейса командной строки 2,0](../../key-vault/general/manage-with-cli2.md) и [Использование Key Vault обратимого удаления с помощью интерфейса командной строки](../../key-vault/general/key-vault-recovery.md).

## <a name="assign-an-azure-ad-identity-to-your-server"></a>Назначение серверу удостоверения Azure AD

```azurecli
# create server (with identity) and database
az sql server create --name <servername> --resource-group <rgname>  --location <location> --admin-user <user> --admin-password <password> --assign-identity
az sql db create --name <dbname> --server <servername> --resource-group <rgname>
```

> [!TIP]
> Нужно, чтобы "principalID" не создавал сервер. На следующем шаге для назначения разрешений хранилища ключей используется идентификатор объекта.

## <a name="grant-key-vault-permissions-to-your-server"></a>Предоставление серверу разрешений на доступ к Key Vault

```azurecli
# create key vault, key and grant permission
az keyvault create --name <kvname> --resource-group <rgname> --location <location> --enable-soft-delete true
az keyvault key create --name <keyname> --vault-name <kvname> --protection software
az keyvault set-policy --name <kvname>  --object-id <objectid> --resource-group <rgname> --key-permissions wrapKey unwrapKey get
```

> [!TIP]
> Для следующего шага сохраните URI или идентификатор нового ключа, например https://contosokeyvault.vault.azure.net/keys/Key1/1a1a2b2b3c3c4d4d5e5e6f6f7g7g8h8h.

## <a name="add-the-key-vault-key-to-the-server-and-set-the-tde-protector"></a>Добавление на сервер ключа из Key Vault и настройка предохранителя TDE

```azurecli
# add server key and update encryption protector
az sql server key create --server <servername> --resource-group <rgname> --kid <keyID>
az sql server tde-key set --server <servername> --server-key-type AzureKeyVault  --resource-group <rgname> --kid <keyID>
```

> [!NOTE]
> Общая длина имени хранилища ключей и имени ключа не может превышать 94 символа.

## <a name="turn-on-tde"></a>Включение прозрачного шифрования данных

```azurecli
# enable encryption
az sql db tde set --database <dbname> --server <servername> --resource-group <rgname> --status Enabled
```

Теперь в базе данных или хранилище данных существует включенный TDE с ключом шифрования, управляемым клиентом, в Azure Key Vault.

## <a name="check-the-encryption-state-and-encryption-activity"></a>Проверка состояния и активности шифрования

```azurecli
# get encryption scan progress
az sql db tde list-activity --database <dbname> --server <servername> --resource-group <rgname>  

# get whether encryption is on or off
az sql db tde show --database <dbname> --server <servername> --resource-group <rgname>
```

* * *

## <a name="useful-powershell-cmdlets"></a>Полезные командлеты PowerShell

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

- Чтобы отключить TDE, используйте командлет [Set-азсклдатабасетранспарентдатаенкриптион](/powershell/module/az.sql/set-azsqldatabasetransparentdataencryption) .

   ```powershell
   Set-AzSqlDatabaseTransparentDataEncryption -ServerName <LogicalServerName> -ResourceGroupName <SQLDatabaseResourceGroupName> `
      -DatabaseName <DatabaseName> -State "Disabled"
   ```

- Используйте командлет [Get-азсклсерверкэйваулткэй](/powershell/module/az.sql/get-azsqlserverkeyvaultkey) , чтобы получить список ключей Key Vault, добавленных на сервер.

   ```powershell
   # KeyId is an optional parameter, to return a specific key version
   Get-AzSqlServerKeyVaultKey -ServerName <LogicalServerName> -ResourceGroupName <SQLDatabaseResourceGroupName>
   ```

- Используйте [Remove-азсклсерверкэйваулткэй](/powershell/module/az.sql/remove-azsqlserverkeyvaultkey) для удаления ключа Key Vault с сервера.

   ```powershell
   # the key set as the TDE Protector cannot be removed
   Remove-AzSqlServerKeyVaultKey -KeyId <KeyVaultKeyId> -ServerName <LogicalServerName> -ResourceGroupName <SQLDatabaseResourceGroupName>
   ```

# <a name="the-azure-cli"></a>[CLI Azure.](#tab/azure-cli)

- Общие параметры базы данных см. в разделе [AZ SQL](/cli/azure/sql).

- Сведения о параметрах ключей хранилища см. в разделе [AZ SQL Server Key](/cli/azure/sql/server/key).

- Сведения о параметрах TDE см. в разделе [AZ SQL Server TDE-Key](/cli/azure/sql/server/tde-key) и [AZ SQL DB TDE](/cli/azure/sql/db/tde).

* * *

## <a name="troubleshooting"></a>Устранение неполадок

При возникновении ошибок выполните следующие проверки.

- Если не удается найти хранилище ключей, убедитесь, что вы используете подходящую подписку.

   # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

   ```powershell
   Get-AzSubscription -SubscriptionId <SubscriptionId>
   ```

   # <a name="the-azure-cli"></a>[CLI Azure.](#tab/azure-cli)

   ```powershell
   az account show - s <SubscriptionId>
   ```

   * * *

- Если не удается добавить на сервер новый ключ или указать новый в качестве предохранителя TDE, проверьте следующее:
   - ключ не должен иметь дату окончания срока действия;
   - для ключа должны быть включены операции *получения*, *упаковки ключа* и *распаковки ключа*.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как сменить средство защиты TDE сервера в соответствии с требованиями безопасности: [поворачивайте средство защиты прозрачное шифрование данных с помощью PowerShell](transparent-data-encryption-byok-key-rotation.md).
- Сведения о том, как удалить потенциально скомпрометированный предохранитель TDE в случае угрозы безопасности, вы найдете в [этой статье](transparent-data-encryption-byok-remove-tde-protector.md).
