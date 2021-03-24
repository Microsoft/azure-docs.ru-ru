---
title: Примеры скриптов для Azure CLI
titleSuffix: Azure SQL Database & SQL Managed Instance
description: Примеры сценариев Azure CLI для создания Базы данных SQL Azure и Управляемого экземпляра SQL Azure и управления ими
services: sql-database
ms.service: sql-db-mi
ms.subservice: service
ms.custom: overview-samples, mvc, sqldbrb=2, devx-track-azurecli
ms.devlang: azurecli
ms.topic: sample
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 02/03/2019
ms.openlocfilehash: 439167f29bb53d4a6e90b95826faa56e3c3170da
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94563364"
---
# <a name="azure-cli-samples-for-azure-sql-database-and-sql-managed-instance"></a>Примеры Azure CLI для Базы данных SQL Azure и Управляемого экземпляра SQL 
 
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Вы можете настроить Базу данных SQL Azure и Управляемый экземпляр SQL с помощью <a href="/cli/azure">Azure CLI</a>.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим руководством требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

# <a name="azure-sql-database"></a>[База данных SQL Azure](#tab/single-database)

В следующей таблице содержатся ссылки на примеры сценариев Azure CLI для управления отдельными базами данных и базами данных в пулах в Базе данных SQL Azure. 

|Область|Описание|
|---|---|
|**Создание баз данных в Базе данных SQL Azure**||
| [Создание отдельной базы данных и настройка правила брандмауэра](scripts/create-and-configure-database-cli.md) | Создает Базу данных SQL и настраивает правило брандмауэра уровня сервера. |
| [Создание эластичных пулов и перемещение баз данных в пуле](scripts/move-database-between-elastic-pools-cli.md) | Создает эластичные пулы и перемещает базы данных в пуле, а также изменяет объемы вычислительных ресурсов. |
|**Масштабирование баз данных в Базе данных SQL Azure**||
| [Масштабирование отдельной базы данных](scripts/monitor-and-scale-database-cli.md) | Масштабирует базу данных в Базе данных SQL до другого объема вычислительных ресурсов после запроса на получение сведений о размере базы данных. |
| [Масштабирование эластичного пула](scripts/scale-pool-cli.md) | Масштабирует эластичный пул SQL, изменяя объем его вычислительных ресурсов. |
|**Настройка георепликации и отработка отказа**||
| [Добавление отдельных баз данных в группу отработки отказа](scripts/add-database-to-failover-group-cli.md)| Создает базу данных и группу отработки отказа, добавляет базу данных в эту группу, а потом тестирует отработку отказа на сервер-получатель. |
| [Use PowerShell to add an Azure SQL Database elastic pool to a failover group](../../sql-database/scripts/sql-database-add-elastic-pool-to-failover-group-cli.md) (Добавление эластичного пула Базы данных SQL Azure в группу отработки отказа с помощью PowerShell) | Создает базу данных, добавляет ее в эластичный пул, добавляет эластичный пул в группу отработки отказа, а потом тестирует отработку отказа на сервер-получатель. |
| [Настройка и выполнение отработки отказа для отдельной базы данных с помощью активной георепликации](../../sql-database/scripts/sql-database-setup-geodr-and-failover-database-cli.md)| Настраивает активную георепликацию для базы данных в Базе данных SQL Azure и выполняет для нее отработку отказа во вторичную реплику. |
| [Настройка и выполнение отработки отказа для базы данных в пуле с помощью активной георепликации](../../sql-database/scripts/sql-database-setup-geodr-and-failover-pool-cli.md)| Настраивает активную георепликацию для базы данных в эластичном пуле и выполняет для нее отработку отказа во вторичную реплику. |
| **Аудит и обнаружение угроз** |
| [Настройка аудита и обнаружения угроз](../../sql-database/scripts/sql-database-auditing-and-threat-detection-cli.md)| Настраивает политики аудита и обнаружения угроз для базы данных в Базе данных SQL Azure. |
| **Резервное копирование, восстановление, копирование и импорт базы данных**||
| [Резервное копирование базы данных](../../sql-database/scripts/sql-database-backup-database-cli.md)| Создает резервную копию базы данных в Базе данных SQL в резервной копии хранилища Azure. |
| [Восстановление базы данных](../../sql-database/scripts/sql-database-restore-database-cli.md)| Восстанавливает базу данных в Базе данных SQL из геоизбыточной резервной копии и восстанавливает последнюю резервную копию удаленной базы данных. |
| [Копирование базы данных на новый сервер](../../sql-database/scripts/sql-database-copy-database-to-new-server-cli.md) | Создает копию существующей базы данных в Базе данных SQL на новом сервере. |
| [Импорт базы данных из BACPAC-файла](../../sql-database/scripts/sql-database-import-from-bacpac-cli.md)| Импорт базы данных из BACPAC-файла в Базу данных SQL. |
|||

См. дополнительные сведения об [API управления отдельной базой данных с помощью Azure CLI](single-database-manage.md#the-azure-cli).

# <a name="azure-sql-managed-instance"></a>[Управляемый экземпляр SQL Azure](#tab/managed-instance)

В следующей таблице содержатся ссылки на примеры скриптов Azure CLI для Управляемого экземпляра SQL Azure.

|Область|Описание|
|---|---|
| **Создание Управляемого экземпляра SQL**||
| [Создание Управляемого экземпляра SQL](../../sql-database/scripts/sql-database-create-configure-managed-instance-cli.md)| Создает Управляемый экземпляр SQL. |
| **Настройка прозрачного шифрования данных (TDE)**||
| [Управление прозрачным шифрованием данных в Управляемом экземпляре SQL с использованием Azure Key Vault](../../sql-database/scripts/transparent-data-encryption-byok-sql-managed-instance-cli.md)| Настраивает прозрачное шифрование данных (TDE) в Управляемом экземпляре SQL, используя Azure Key Vault с различными ключевыми сценариями. |
|**Настройка группы отработки отказа**||
| [Настройка группы отработки отказа для Управляемого экземпляра SQL](../../sql-database/scripts/sql-database-add-managed-instance-to-failover-group-cli.md) | Создает два Управляемых экземпляра SQL, добавляет их в группу отработки отказа, а затем проверяет отработку отказа с основного на дополнительный Управляемый экземпляр SQL. |
|||

Дополнительные примеры работы с Управляемым экземпляром SQL см. в сценариях [создание](/archive/blogs/sqlserverstorageengine/create-azure-sql-managed-instance-using-azure-cli), [обновление](/archive/blogs/sqlserverstorageengine/modify-azure-sql-database-managed-instance-using-azure-cli), [перемещение базы данных](/archive/blogs/sqlserverstorageengine/cross-instance-point-in-time-restore-in-azure-sql-database-managed-instance), [работа с](https://medium.com/azure-sqldb-managed-instance/working-with-sql-managed-instance-using-azure-cli-611795fe0b44).

См. дополнительные сведения об [API управления Управляемым экземпляром SQL с помощью Azure CLI](../managed-instance/api-references-create-manage-instance.md#azure-cli-create-and-configure-managed-instances).

---