---
title: Примеры скриптов Azure PowerShell.
description: Используйте примеры скриптов Azure PowerShell для создания и администрирования ресурсов Базы данных SQL Azure и Управляемого экземпляра SQL Azure.
services: sql-database
ms.service: sql-db-mi
ms.subservice: development
ms.custom: sqldbrb=2
ms.devlang: PowerShell
ms.topic: sample
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/25/2019
ms.openlocfilehash: 7d732f0e42bc5eef8f72a2b9ce2adee28deb9a67
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101690980"
---
# <a name="azure-powershell-samples-for-azure-sql-database-and-azure-sql-managed-instance"></a>Примеры сценариев Azure PowerShell для Базы данных SQL Azure и Управляемого экземпляра SQL Azure
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Службы "База данных SQL Azure" и "Управляемый экземпляр SQL Azure" позволяют настраивать базы данных, экземпляры и пулы с помощью Azure PowerShell.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально, вам понадобится AZ PowerShell 1.4.0 или последующей версии для работы с этим руководством. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="azure-sql-database"></a>[База данных SQL Azure](#tab/single-database)

В следующей таблице содержатся ссылки на примеры сценариев Azure PowerShell для Базы данных SQL Azure.

|Ссылка|Описание|
|---|---|
|**Создание и настройка отдельных баз данных и эластичных пулов**||
| [Создание отдельной базы данных и настройка правила брандмауэра на уровне сервера](scripts/create-and-configure-database-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Этот скрипт PowerShell создает отдельную базу данных и настраивает правило брандмауэра для IP-адресов на уровне сервера. |
| [Создание эластичных пулов и перемещение баз данных в пуле](scripts/move-database-between-elastic-pools-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Этот скрипт PowerShell создает эластичные пулы, перемещает базы данных в составе пулов и изменяет объем вычислительных ресурсов.|
|**Настройка георепликации и отработка отказа**||
| [Настройка и обработка отказа для отдельной базы данных с помощью активной георепликации](scripts/setup-geodr-and-failover-database-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Этот сценарий PowerShell настраивает активную георепликацию для отдельной базы данных и выполняет для нее отработку отказа на вторичную реплику. |
| [Настройка и обработка отказа для базы данных в пуле с помощью активной георепликации](scripts/setup-geodr-and-failover-elastic-pool-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Этот скрипт PowerShell настраивает активную георепликацию для базы данных в эластичном пуле и выполняет для нее отработку отказа на вторичную реплику. |
|**Настройка группы отработки отказа**||
| [Настройка группы отработки отказа для отдельной базы данных](scripts/add-database-to-failover-group-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Этот скрипт PowerShell создает базу данных и группу отработки отказа, добавляет базу данных в эту группу и тестирует отработку отказа на сервер-получатель. |
| [Use PowerShell to add an Azure SQL Database elastic pool to a failover group](scripts/add-elastic-pool-to-failover-group-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) (Добавление эластичного пула Базы данных SQL Azure в группу отработки отказа с помощью PowerShell) | Этот скрипт PowerShell создает базу данных, добавляет ее в эластичный пул, добавляет эластичный пул в группу отработки отказа и тестирует отработку отказа на сервер-получатель. |
|**Масштабирование отдельной базы данных и эластичного пула**||
| [Масштабирование отдельной базы данных](scripts/monitor-and-scale-database-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Этот скрипт PowerShell отслеживает метрики производительности отдельной базы данных, увеличивает объем вычислительных ресурсов и создает правило генерации оповещений для одной из метрик производительности. |
| [Масштабирование эластичного пула](scripts/monitor-and-scale-pool-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Этот сценарий PowerShell отслеживает метрики производительности эластичного пула, увеличивает объем его вычислительных ресурсов и создает правило генерации оповещений для одной из метрик производительности. |
| **Аудит и обнаружение угроз** |
| [Настройка аудита и обнаружения угроз](scripts/auditing-threat-detection-powershell-configure.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Этот скрипт PowerShell настраивает политики аудита и обнаружения угроз для базы данных. |
| **Восстановление, копирование и импорт базы данных**||
| [Восстановление базы данных](scripts/restore-database-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Этот сценарий PowerShell восстанавливает базу данных из геоизбыточной резервной копии и восстанавливает последнюю резервную копию удаленной базы данных. |
| [Копирование базы данных на новый сервер](scripts/copy-database-to-new-server-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Этот сценарий PowerShell создает копию существующей базы данных на новом сервере. |
| [Импорт базы данных из BACPAC-файла](scripts/import-from-bacpac-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Этот сценарий PowerShell импортирует базу данных в базу данных SQL Azure из BACPAC-файла. |
| **Синхронизация данных между базами данных**||
| [Синхронизация данных между базами данных](scripts/sql-data-sync-sync-data-between-sql-databases.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Этот сценарий PowerShell настраивает технологию "Синхронизация данных" для синхронизации между несколькими базами данных в Базе данных SQL Azure. |
| [Синхронизация данных между базой данных SQL и локальной базой данных SQL Server](scripts/sql-data-sync-sync-data-between-azure-onprem.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Этот скрипт PowerShell настраивает синхронизацию данных между базой данных в Базе данных SQL Azure и локальной базой данных SQL Server. |
| [Изменение схемы синхронизации данных SQL](scripts/update-sync-schema-in-sync-group.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Этот сценарий PowerShell добавляет или удаляет элементы в схеме синхронизации данных. |
|||

См. раздел [PowerShell](single-database-manage.md#powershell). 

## <a name="azure-sql-managed-instance"></a>[Управляемый экземпляр SQL Azure](#tab/managed-instance)

В следующей таблице содержатся ссылки на примеры сценариев Azure PowerShell для Управляемого экземпляра SQL Azure.

|Ссылка|Описание|
|---|---|
|**Создание и настройка управляемых экземпляров**||
| [Создание управляемого экземпляра и управление им](../managed-instance/scripts/create-configure-managed-instance-powershell.md) | Этот скрипт PowerShell создает и администрирует управляемый экземпляр с помощью Azure PowerShell. |
| [Создание и администрирование управляемого экземпляра с помощью шаблона Azure Resource Manager](../managed-instance/create-template-quickstart.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Этот скрипт PowerShell создает и администрирует управляемый экземпляр с помощью Azure PowerShell и шаблона Azure Resource Manager.|
| [Восстановление базы данных в управляемый экземпляр в другом географическом регионе](../managed-instance/scripts/restore-geo-backup.md) | Этот скрипт PowerShell восстанавливает резервную копию базы данных в другом регионе. Такой сценарий называется аварийным геовосстановлением. |
| **Настройка прозрачного шифрования данных**||
| [Управление прозрачным шифрованием данных в управляемом экземпляре с использованием собственного ключа из Azure Key Vault](../managed-instance/scripts/transparent-data-encryption-byok-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Этот скрипт PowerShell настраивает прозрачное шифрование данных в сценарии создания собственных ключей для Управляемого экземпляра SQL Azure с помощью ключа из Azure Key Vault.|
|**Настройка группы отработки отказа**||
| [Настройка группы отработки отказа для управляемого экземпляра](../managed-instance/scripts/add-to-failover-group-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Этот скрипт PowerShell создает два управляемых экземпляра, добавляет их в группу отработки отказа, а затем проверяет отработку отказа с основного на дополнительный управляемый экземпляр. |
|||

Дополнительные сведения о [командлетах PowerShell для Управляемого экземпляра Azure SQL](../managed-instance/api-references-create-manage-instance.md#powershell-create-and-configure-managed-instances).

---

## <a name="additional-resources"></a>Дополнительные ресурсы

В примерах, приведенных на этой странице, используются [командлеты PowerShell](/powershell/module/az.sql/) для создания ресурсов SQL Azure и управления ими. Дополнительные командлеты для выполнения запросов и множества других задач баз данных доступны в модуле [sqlserver](/powershell/module/sqlserver/). Подробные сведения см. в статье [SQL Server PowerShell](/sql/powershell/sql-server-powershell/).