---
title: Копирование данных из локальной среды в Azure с помощью PowerShell
description: Этот сценарий PowerShell копирует данные из базы данных SQL Server в другое хранилище BLOB-объектов Azure.
ms.service: data-factory
ms.topic: article
ms.author: jingwang
author: linda33wj
ms.custom: seo-lt-2019
ms.date: 10/31/2017
ms.openlocfilehash: 34c5497a8b059260dfe60e8015e62c0f1511f021
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100373391"
---
# <a name="use-powershell-to-create-a-data-factory-pipeline-to-copy-data-from-sql-server-to-azure"></a>Создание конвейера фабрики данных для копирования данных из SQL Server в Azure с помощью PowerShell

Этот пример скрипта PowerShell создает конвейер в фабрике данных Azure, который копирует данные из базы данных SQL Server в хранилище BLOB-объектов Azure.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh-az.md)]

## <a name="prerequisites"></a>Предварительные требования

- **SQL Server.** В этом примере в качестве **исходного** хранилища данных используется база данных SQL Server.
- **Учетная запись хранения Azure.** В этом примере в качестве **места назначения и приемника** будет использоваться хранилище BLOB-объектов Azure. Если у вас нет учетной записи хранения Azure, создайте ее, как описано в статье [Создание учетной записи хранения](../../storage/common/storage-account-create.md) .
- Локальная **Среда выполнения интеграции**. Скачайте MSI-файл из [центра загрузки](https://www.microsoft.com/download/details.aspx?id=39717) и запустите его, чтобы установить локальную среду выполнения интеграции на компьютере.  

### <a name="create-sample-database-in-sql-server"></a>Создание примера базы данных в SQL Server
1. В SQL Server базе данных создайте таблицу с именем **EMP** с помощью следующего скрипта SQL:

   ```sql   
     CREATE TABLE dbo.emp
     (
         ID int IDENTITY(1,1) NOT NULL,
         FirstName varchar(50),
         LastName varchar(50),
         CONSTRAINT PK_emp PRIMARY KEY (ID)
     )
     GO
   ```

2. Вставьте в эту таблицу примеры данных.

   ```sql
     INSERT INTO emp VALUES ('John', 'Doe')
     INSERT INTO emp VALUES ('Jane', 'Doe')
   ```

## <a name="sample-script"></a>Пример скрипта

> [!IMPORTANT]
> Этот скрипт создает JSON-файлы, определяющие сущности фабрики данных (связанную службу, набор данных и конвейер) на жестком диске в папке c:\.

[!code-powershell[main](../../../powershell_scripts/data-factory/copy-from-onprem-sql-server-to-azure-blob/copy-from-onprem-sql-server-to-azure-blob.ps1 "Copy from SQL Server -> Azure Blob Storage")]


## <a name="clean-up-deployment"></a>Очистка развертывания

После выполнения примера скрипта можно удалить группу ресурсов и все связанные с ней ресурсы, выполнив следующую команду.

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
```
Чтобы удалить фабрику данных из группы ресурсов, выполните следующую команду:

```powershell
Remove-AzDataFactoryV2 -Name $dataFactoryName -ResourceGroupName $resourceGroupName
```

## <a name="script-explanation"></a>Описание скрипта

Этот сценарий использует следующие команды:

| Get-Help | Примечания |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [Set-AzDataFactoryV2](/powershell/module/az.datafactory/set-Azdatafactoryv2) | Создали фабрику данных. |
| [New-AzDataFactoryV2LinkedServiceEncryptCredential](/powershell/module/az.datafactory/new-Azdatafactoryv2linkedserviceencryptedcredential) | Шифрует учетные данные в связанной службе и создает новое определение связанной службы с зашифрованными учетными данными.
| [Set-AzDataFactoryV2LinkedService](/powershell/module/az.datafactory/Set-Azdatafactoryv2linkedservice) | Создает в этой фабрике данных связанную службу. Связанная служба вычисляет или привязывает хранилище данных к фабрике данных. |
| [Set-AzDataFactoryV2Dataset](/powershell/module/az.datafactory/Set-Azdatafactoryv2dataset) | Создает набор данных в фабрике данных. Набор данных представляет ввод или вывод для действия в конвейере. |
| [Set-AzDataFactoryV2Pipeline](/powershell/module/az.datafactory/Set-Azdatafactoryv2pipeline) | Создает конвейер в фабрике данных. Конвейер содержит одно или несколько действий, выполняющих определенную операцию. В этом конвейере действие копирования копирует данные из одного расположения в другое в хранилище BLOB-объектов Azure. |
| [Invoke-AzDataFactoryV2Pipeline](/powershell/module/az.datafactory/Invoke-Azdatafactoryv2pipeline) | Создает выполнение для конвейера. Другими словами, запускает конвейер. |
| [Get-AzDataFactoryV2ActivityRun](/powershell/module/az.datafactory/get-Azdatafactoryv2activityrun) | Получает сведения о выполнении действия в конвейере.
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов со всеми вложенными ресурсами. |
|||

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure PowerShell см. в [документации по Azure PowerShell](/powershell/).

Дополнительные примеры сценариев PowerShell для Azure Data Factory приведены [здесь](../samples-powershell.md).