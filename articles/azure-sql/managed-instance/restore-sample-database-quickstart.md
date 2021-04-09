---
title: Краткое руководство. Восстановление резервных копий (SSMS)
titleSuffix: Azure SQL Managed Instance
description: В этом кратком руководстве описано, как восстановить резервную копию базы данных в Управляемый экземпляр SQL Azure с помощью SQL Server Management Studio (SSMS).
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: quickstart
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova
ms.date: 12/14/2018
ms.openlocfilehash: 15aead5c9f93205baac28e8b93a88015c9689e3a
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105625319"
---
# <a name="quickstart-restore-a-database-to-azure-sql-managed-instance-with-ssms"></a>Краткое руководство. Восстановление резервной копии базы данных в Управляемый экземпляр SQL Azure с помощью SSMS
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

В этом кратком руководстве показано, как использовать SQL Server Management Studio (SSMS) для восстановления базы данных (Wide World Importers — стандартный файл резервной копии) из Хранилища BLOB-объектов Azure в [Управляемый экземпляр SQL Azure](sql-managed-instance-paas-overview.md).

> [!VIDEO https://www.youtube.com/embed/RxWYojo_Y3Q]

> [!NOTE]
> См. статью [Руководство. Миграция из SQL Server в Управляемый экземпляр базы данных SQL Azure в автономном режиме с помощью DMS](../../dms/tutorial-sql-server-to-managed-instance.md).
> Дополнительные сведения о методах миграции см. в статье [Перенос SQL Server в Управляемый экземпляр SQL Azure](migrate-to-instance-from-sql-server.md).

## <a name="prerequisites"></a>Предварительные требования

В этом кратком руководстве:

- Используйте ресурсы из статьи [Краткое руководство. Создание Управляемого экземпляра Базы данных SQL Azure](instance-create-quickstart.md).
- Необходимо установить последнюю версию [SSMS](/sql/ssms/sql-server-management-studio-ssms).
- SSMS используется для подключения к Управляемому экземпляру SQL. Ознакомьтесь с этими краткими руководствами по подключению.
  - [Включение общедоступной конечной точки](public-endpoint-configure.md) в Управляемом экземпляре SQL. Рекомендуется использовать этот подход для работы с данным руководством.
  - [Подключение к Управляемому экземпляру SQL с виртуальной машины Azure.](connect-vm-instance-configure.md)
  - [Настройка соединения "точка — сеть" с Управляемым экземпляром SQL.](point-to-site-p2s-configure.md)

> [!NOTE]
> Дополнительные сведения о резервном копировании и восстановлении базы данных SQL Server при помощи хранилища BLOB-объектов Azure и [ключа подписанного URL-адреса (SAS)](../../storage/common/storage-sas-overview.md) см. в статье [Краткое руководство. Восстановление резервной копии базы данных в Управляемый экземпляр Базы данных SQL Azure](/sql/relational-databases/backup-restore/sql-server-backup-to-url).

## <a name="restore-from-a-backup-file"></a>Восстановление из файла резервной копии

В SQL Server Management Studio выполните следующие шаги для восстановления базы данных Wide World Importers в Управляемый экземпляр SQL. Файл резервной копии базы данных хранится в предварительно настроенной учетной записи хранения больших двоичных объектов Azure.

1. Откройте SSMS и подключитесь к Управляемому экземпляру.
2. В **обозревателе объектов** щелкните правой кнопкой мыши Управляемый экземпляр и выберите **Создать запрос**, чтобы открыть окно нового запроса.
3. Запустите следующий скрипт SQL, в котором используется предварительно настроенная учетная запись хранения и ключ SAS, чтобы [создать учетные данные](/sql/t-sql/statements/create-credential-transact-sql) в управляемом экземпляре.
 
   > [!IMPORTANT]
   > Объект `CREDENTIAL` должен соответствовать пути к контейнеру, начинаться с `https` и не может содержать косую черту в конце. Аргумент `IDENTITY` должен иметь значение `SHARED ACCESS SIGNATURE`. Объект `SECRET` должен быть маркером подписанного URL-адреса и не может содержать символ `?` в начале.

   ```sql
   CREATE CREDENTIAL [https://mitutorials.blob.core.windows.net/databases]
   WITH IDENTITY = 'SHARED ACCESS SIGNATURE'
   , SECRET = 'sv=2017-11-09&ss=bfqt&srt=sco&sp=rwdlacup&se=2028-09-06T02:52:55Z&st=2018-09-04T18:52:55Z&spr=https&sig=WOTiM%2FS4GVF%2FEEs9DGQR9Im0W%2BwndxW2CQ7%2B5fHd7Is%3D'
   ```

    ![Создание учетных данных](./media/restore-sample-database-quickstart/credential.png)

4. Чтобы проверить учетные данные, запустите следующий скрипт, который использует URL-адрес [контейнера](https://azure.microsoft.com/services/container-instances/) для получения списка файлов резервной копии.

   ```sql
   RESTORE FILELISTONLY FROM URL =
      'https://mitutorials.blob.core.windows.net/databases/WideWorldImporters-Standard.bak'
   ```

    ![список файлов](./media/restore-sample-database-quickstart/file-list.png)

5. Запустите следующий скрипт, чтобы восстановить базу данных Wide World Importers.

   ```sql
   RESTORE DATABASE [Wide World Importers] FROM URL =
     'https://mitutorials.blob.core.windows.net/databases/WideWorldImporters-Standard.bak'
   ```

    ![Снимок экрана: сообщение об успешном выполнении скрипта в обозревателе объектов](./media/restore-sample-database-quickstart/restore.png)

6. Запустите следующий скрипт, чтобы отслеживать состояние восстановления.

   ```sql
   SELECT session_id as SPID, command, a.text AS Query, start_time, percent_complete
      , dateadd(second,estimated_completion_time/1000, getdate()) as estimated_completion_time
   FROM sys.dm_exec_requests r
   CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) a
   WHERE r.command in ('BACKUP DATABASE','RESTORE DATABASE')
   ```

7. По завершении восстановления просмотрите базу данных в обозревателе объектов. Чтобы убедиться, что восстановление базы данных завершено, используйте представление [sys.dm_operation_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database).

> [!NOTE]
> Операция восстановления базы данных является асинхронной и повторяемой. В случае разрыва подключения или истечения времени ожидания в SQL Server Management Studio может возникнуть ошибка. База данных SQL Azure будет пытаться восстановить базу данных в фоновом режиме, и вы сможете отслеживать ход восстановления с помощью представлений [sys.dm_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql) и [sys.dm_operation_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database).
> На некоторых этапах процесса восстановления в системных представлениях вместо фактического имени базы данных будет отображаться ее уникальный идентификатор. [Узнайте больше](./transact-sql-tsql-differences-sql-server.md#restore-statement) о различиях в поведении инструкции `RESTORE`.

## <a name="next-steps"></a>Дальнейшие действия

- Если на шаге 5 восстановление базы данных завершается и появляется сообщение ID 22003 (Идентификатор 22003), создайте новый файл резервной копии, содержащий контрольные суммы резервной копии, и снова выполните восстановление. См. статью [Включение или отключение вычисления контрольных сумм резервных копий во время архивации или восстановления](/sql/relational-databases/backup-restore/enable-or-disable-backup-checksums-during-backup-or-restore-sql-server).
- См. раздел [Резервное копирование SQL Server на URL-адрес — рекомендации и устранение неполадок](/sql/relational-databases/backup-restore/sql-server-backup-to-url-best-practices-and-troubleshooting).
- Обзор вариантов подключения для приложений см. в статье [Подключение приложения к Управляемому экземпляру SQL](connect-application-instance.md).
- Чтобы отправлять запросы с помощью привычных средств или языков, ознакомьтесь со статьей [Краткие руководства. Подключение и создание запросов к Базе данных SQL Azure](../database/connect-query-content-reference-guide.md).
