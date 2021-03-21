---
title: Копирование данных в массовый среде с помощью PowerShell
description: С помощью этого скрипта PowerShell можно использовать фабрику данных Azure для копирования данных из исходного хранилища данных в хранилище данных назначения в пакетном режиме.
ms.author: jingwang
author: linda33wj
ms.service: data-factory
ms.topic: article
ms.custom: seo-lt-2019
ms.date: 10/31/2017
ms.openlocfilehash: 16cb08d49efcd46e2746c5031793af8b747a2b31
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100373578"
---
# <a name="powershell-script---copy-multiple-tables-in-bulk-by-using-azure-data-factory"></a>Скрипт PowerShell: копирование нескольких таблиц в пакетном режиме с помощью фабрики данных Azure

Этот пример сценария PowerShell копирует данные из нескольких таблиц в базе данных SQL Azure в Azure синапсе Analytics.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh-az.md)]

Перечень компонентов, необходимых для запуска этого примера, см. в [руководстве по копированию в пакетном режиме](../tutorial-bulk-copy.md#prerequisites).

## <a name="sample-script"></a>Пример скрипта

> [!IMPORTANT]
> Этот скрипт создает JSON-файлы, определяющие сущности фабрики данных (связанную службу, набор данных и конвейер) на жестком диске в папке c:\.

[!code-powershell[main](../../../powershell_scripts/data-factory/bulk-copy-from-sql-databse-to-sql-data-warehouse/bulk-copy-from-sql-database-to-sql-data-warehouse.ps1 "Bulk copy from Azure SQL Database => Azure Synapse Analytics")]

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
| [Set-AzDataFactoryV2](/powershell/module/az.datafactory/set-azdatafactoryv2) | Создали фабрику данных. |
| [Set-AzDataFactoryV2LinkedService](/powershell/module/az.datafactory/set-azdatafactoryv2linkedservice) | Создает в этой фабрике данных связанную службу. Связанная служба вычисляет или привязывает хранилище данных к фабрике данных. |
| [Set-AzDataFactoryV2Dataset](/powershell/module/az.datafactory/set-azdatafactoryv2dataset) | Создает набор данных в фабрике данных. Набор данных представляет ввод или вывод для действия в конвейере. | 
| [Set-AzDataFactoryV2Pipeline](/powershell/module/az.datafactory/set-azdatafactoryv2pipeline) | Создает конвейер в фабрике данных. Конвейер содержит одно или несколько действий, выполняющих определенную операцию. В этом конвейере действие копирования копирует данные из одного расположения в другое в хранилище BLOB-объектов Azure. |
| [Invoke-AzDataFactoryV2Pipeline](/powershell/module/az.datafactory/invoke-azdatafactoryv2pipeline) | Создает выполнение для конвейера. Другими словами, запускает конвейер. |
| [Get-AzDataFactoryV2ActivityRun](/powershell/module/az.datafactory/get-azdatafactoryv2activityrun) | Получает сведения о выполнении действия в конвейере. 
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов со всеми вложенными ресурсами. |
|||

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure PowerShell см. в [документации по Azure PowerShell](/powershell/).

Дополнительные примеры сценариев Azure Data Factory PowerShell приведены [здесь](../samples-powershell.md).