---
title: Миграция экземпляра, управляемого сертификатом TDE
description: Перенос сертификата, защищающего ключ шифрования базы данных, с прозрачное шифрование данных в Azure SQL Управляемый экземпляр
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: MladjoA
ms.author: mlandzic
ms.reviewer: sstein, jovanpop
ms.date: 07/21/2020
ms.openlocfilehash: c465da3d5d812ea7e811cbe59318122700c6e786
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97824660"
---
# <a name="migrate-a-certificate-of-a-tde-protected-database-to-azure-sql-managed-instance"></a>Перенос сертификата базы данных, защищенной TDE, в Azure SQL Управляемый экземпляр
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

При миграции базы данных, защищенной [прозрачное шифрование данных (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption) , в Azure SQL управляемый экземпляр с помощью параметра Native Restore, необходимо выполнить миграцию соответствующего сертификата из SQL Server экземпляра перед восстановлением базы данных. В этой статье описывается процесс переноса сертификата вручную в Azure SQL Управляемый экземпляр.

> [!div class="checklist"]
>
> * Экспорт сертификата в файл обмена личной информацией (PFX)
> * Извлечение сертификата из файла в основную строку 64
> * Отправка с помощью командлета PowerShell

Альтернативный вариант использования полностью управляемой службы для простой миграции как базы данных, защищенной TDE, так и соответствующего сертификата, см. в статье [Перенос локальной базы данных в Azure SQL управляемый экземпляр с помощью Azure Database Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md).

> [!IMPORTANT]
> Перенесенный сертификат используется только для восстановления базы данных, защищенной только для TDE. Вскоре после восстановления перенесенный сертификат заменяется другим предохранителем, либо управляемым службой сертификатом, либо асимметричным ключом из хранилища ключей в зависимости от типа TDE, установленного в экземпляре.

## <a name="prerequisites"></a>Предварительные условия

Чтобы выполнить действия, описанные в этой статье, необходимо следующее.

* Средство командной строки[Pvk2Pfx](/windows-hardware/drivers/devtest/pvk2pfx), установленное на локальном сервере или другом компьютере с доступом к сертификату, экспортированному как файл. Средство Pvk2Pfx входит в состав [набора драйверов Enterprise для Windows](/windows-hardware/drivers/download-the-wdk), автономной среды командной строки.
* [Windows PowerShell](/powershell/scripting/install/installing-windows-powershell) версии 5.0 или более поздней.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Вам потребуются:

* Модуль Azure PowerShell [установлен и обновлен](/powershell/azure/install-az-ps).
* [AZ. SQL Module](https://www.powershellgallery.com/packages/Az.Sql).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

> [!IMPORTANT]
> Модуль PowerShell Azure Resource Manager по-прежнему поддерживается Управляемый экземпляр SQL Azure, но вся будущая разработка предназначена для модуля AZ. SQL. Сведения об этих командлетах см. в разделе [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Аргументы для команд в модуле AZ и в модулях AzureRM существенно идентичны.

Выполните следующие команды в PowerShell, чтобы установить или обновить модуль:

```azurepowershell
Install-Module -Name Az.Sql
Update-Module -Name Az.Sql
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli).

* * *

## <a name="export-the-tde-certificate-to-a-pfx-file"></a>Экспорт сертификата TDE в PFX-файл

Сертификат можно экспортировать непосредственно из исходного экземпляра SQL Server или из хранилища сертификатов, если он хранится там.

### <a name="export-the-certificate-from-the-source-sql-server-instance"></a>Экспорт сертификата из исходного экземпляра SQL Server

Выполните следующие действия, чтобы экспортировать сертификат с SQL Server Management Studio и преобразовать его в формат PFX. Универсальные имена *TDE_Cert* и *full_path* используются для сертификатов, имен файлов и путей с помощью шагов. Их должны заменить фактические имена.

1. В среде SSMS откройте новое окно запроса и подключитесь к экземпляру SQL Server источника.

1. Используйте следующий скрипт для перечисления баз данных, защищенных TDE, и получения имени сертификата, защищающего шифрование переносимой базы данных:

   ```sql
   USE master
   GO
   SELECT db.name as [database_name], cer.name as [certificate_name]
   FROM sys.dm_database_encryption_keys dek
   LEFT JOIN sys.certificates cer
   ON dek.encryptor_thumbprint = cer.thumbprint
   INNER JOIN sys.databases db
   ON dek.database_id = db.database_id
   WHERE dek.encryption_state = 3
   ```

   ![Список сертификатов TDE](./media/tde-certificate-migrate/onprem-certificate-list.png)

1. Выполните следующее действие, чтобы экспортировать сертификат в пару файлов (.cer и .pvk), сохраняя данные открытого и закрытого ключа.

   ```sql
   USE master
   GO
   BACKUP CERTIFICATE TDE_Cert
   TO FILE = 'c:\full_path\TDE_Cert.cer'
   WITH PRIVATE KEY (
     FILE = 'c:\full_path\TDE_Cert.pvk',
     ENCRYPTION BY PASSWORD = '<SomeStrongPassword>'
   )
   ```

   ![Резервное копирование сертификата TDE](./media/tde-certificate-migrate/backup-onprem-certificate.png)

1. Используйте консоль PowerShell для копирования сведений о сертификатах из пары вновь созданных файлов в PFX-файл с помощью средства Pvk2Pfx:

   ```cmd
   .\pvk2pfx -pvk c:/full_path/TDE_Cert.pvk  -pi "<SomeStrongPassword>" -spc c:/full_path/TDE_Cert.cer -pfx c:/full_path/TDE_Cert.pfx
   ```

### <a name="export-the-certificate-from-a-certificate-store"></a>Экспорт сертификата из хранилища сертификатов

Если сертификат хранится в SQL Server хранилище сертификатов локального компьютера, его можно экспортировать, выполнив следующие действия.

1. Откройте консоль PowerShell и выполните следующую команду, чтобы открыть оснастку "сертификаты" консоли управления (MMC):

   ```cmd
   certlm
   ```

2. Чтобы просмотреть список сертификатов, в оснастке MMC "сертификаты" разверните узел пути личных > сертификатов.

3. Щелкните сертификат правой кнопкой мыши и выберите пункт **Экспорт**.

4. Следуйте указаниям мастера, чтобы экспортировать сертификат и закрытый ключ в формат PFX.

## <a name="upload-the-certificate-to-azure-sql-managed-instance-using-an-azure-powershell-cmdlet"></a>Отправка сертификата в Azure SQL Управляемый экземпляр с помощью командлета Azure PowerShell

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Начните с шагов подготовки в PowerShell.

   ```azurepowershell
   # import the module into the PowerShell session
   Import-Module Az
   # connect to Azure with an interactive dialog for sign-in
   Connect-AzAccount
   # list subscriptions available and copy id of the subscription target the managed instance belongs to
   Get-AzSubscription
   # set subscription for the session
   Select-AzSubscription <subscriptionId>
   ```

2. После завершения всех подготовительных действий выполните следующие команды, чтобы передать сертификат в кодировке Base-64 в целевой управляемый экземпляр:

   ```azurepowershell
   $fileContentBytes = Get-Content 'C:/full_path/TDE_Cert.pfx' -AsByteStream
   $base64EncodedCert = [System.Convert]::ToBase64String($fileContentBytes)
   $securePrivateBlob = $base64EncodedCert  | ConvertTo-SecureString -AsPlainText -Force
   $password = "<password>"
   $securePassword = $password | ConvertTo-SecureString -AsPlainText -Force
   Add-AzSqlManagedInstanceTransparentDataEncryptionCertificate -ResourceGroupName "<resourceGroupName>" `
       -ManagedInstanceName "<managedInstanceName>" -PrivateBlob $securePrivateBlob -Password $securePassword
   ```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Сначала необходимо [настроить хранилище ключей Azure](../../key-vault/general/manage-with-cli2.md) с помощью *PFX* -файла.

1. Начните с шагов подготовки в PowerShell.

   ```azurecli
   # connect to Azure with an interactive dialog for sign-in
   az login

   # list subscriptions available and copy id of the subscription target the managed instance belongs to
   az account list

   # set subscription for the session
   az account set --subscription <subscriptionId>
   ```

1. После завершения всех подготовительных действий выполните следующие команды, чтобы передать сертификат в кодировке Base-64 в целевой управляемый экземпляр:

   ```azurecli
   az sql mi tde-key set --server-key-type AzureKeyVault --kid "<keyVaultId>" `
       --managed-instance "<managedInstanceName>" --resource-group "<resourceGroupName>"
   ```

* * *

Теперь сертификат доступен для указанного управляемого экземпляра, и резервная копия соответствующей базы данных, защищенной с помощью TDE, может быть успешно восстановлена.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как выполнить миграцию сертификата, защищающего ключ шифрования базы данных, с прозрачное шифрование данных, из локального или экземпляра SQL Server IaaS в Управляемый экземпляр Azure SQL.

Сведения о восстановлении резервной копии базы данных в Управляемый экземпляр SQL Azure см. в статье [Восстановление резервной копии базы данных в управляемый экземпляр Azure SQL](restore-sample-database-quickstart.md) .