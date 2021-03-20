---
title: Включение прозрачное шифрование данных для Stretch Database (T-SQL)
description: Включение прозрачного шифрования данных (TDE) для SQL Server Stretch Database в Azure TSQL
services: sql-server-stretch-database
documentationcenter: ''
ms.assetid: 27753d91-9ca2-4d47-b34d-b5e2c2f029bb
ms.service: sql-server-stretch-database
ms.workload: data-management
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 01/23/2017
author: blazem-msft
ms.author: blazem
ms.reviewer: jroth
manager: jroth
ms.custom: seo-lt-2019
ms.openlocfilehash: 772341d046186e46b79ad7b11170e1bad23a3a6f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95024200"
---
# <a name="enable-transparent-data-encryption-tde-for-stretch-database-on-azure-transact-sql"></a>Включение прозрачного шифрования данных (TDE) для Stretch Database в Azure (Transact-SQL)
> [!div class="op_single_selector"]
> * [Портал Azure](sql-server-stretch-database-encryption-tde.md)
> * [TSQL](sql-server-stretch-database-tde-tsql.md)
>
>

Прозрачное шифрование данных (TDE) помогает защититься от угрозы вредоносных атак за счет шифрования и расшифровки базы данных, связанных резервных копий и файлов журналов транзакций при хранении в реальном времени, не внося изменения в само приложение.

При использовании TDE хранилище всей базы данных шифруется с помощью симметричного ключа, который называется ключом шифрования базы данных. Ключ шифрования базы данных защищается встроенным сертификатом сервера. Каждый сервер Azure обладает уникальным встроенным сертификатом. Майкрософт автоматически меняет эти сертификаты каждые 90 дней. Общее описание TDE см. в статье [Прозрачное шифрование данных (TDE)].

## <a name="enabling-encryption"></a>Включение шифрования
Чтобы включить прозрачное шифрование для базы данных Azure, где хранятся данные, перенесенные из Базы данных SQL Server Stretch, выполните следующее.

1. Подключитесь к базе данных *master* на сервере Azure, где находится база данных, указав имя входа администратора или члена роли **dbmanager** в базе данных master.
2. Выполните следующий оператор для шифрования базы данных:

```sql
ALTER DATABASE [database_name] SET ENCRYPTION ON;
```

## <a name="disabling-encryption"></a>Отключение шифрования
Чтобы отключить прозрачное шифрование для базы данных Azure, где хранятся данные, перенесенные из Базы данных SQL Server Stretch, выполните следующее.

1. Подключитесь к *главной* базе данных, указав логин администратора или члена роли **dbmanager** в главной базе данных.
2. Выполните следующий оператор для шифрования базы данных:

```sql
ALTER DATABASE [database_name] SET ENCRYPTION OFF;
```

## <a name="verifying-encryption"></a>Проверка шифрования
Чтобы проверить состояние шифрования для базы данных Azure, где хранятся данные, перенесенные из Базы данных SQL Server Stretch, выполните следующее.

1. Подключитесь к *главной* базе данных или к базе данных экземпляра, указав логин администратора или члена роли **dbmanager** в главной базе данных.
2. Выполните следующий оператор для шифрования базы данных:

```sql
SELECT
    [name],
    [is_encrypted]
FROM
    sys.databases;
```

Результат ```1``` означает зашифрованную, а ```0``` — незашифрованную базу данных.

<!--Anchors-->
[Прозрачное шифрование данных (TDE)]: /sql/relational-databases/security/encryption/transparent-data-encryption


<!--Image references-->

<!--Link references-->