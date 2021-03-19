---
title: Перенос базы данных из SQL Server в SQL Управляемый экземпляр с поддержкой Arc
description: Перенос базы данных из SQL Server в SQL Управляемый экземпляр с поддержкой Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 86563b0a44bade2cedaf76af3c247821756111fe
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90939634"
---
# <a name="migrate-sql-server-to-azure-arc-enabled-sql-managed-instance"></a>Миграция: SQL Server в SQL Управляемый экземпляр с поддержкой Arc

В этом сценарии описано, как выполнить миграцию базы данных из экземпляра SQL Server в управляемый экземпляр SQL Azure в службе "Дуга Azure" с помощью двух различных методов резервного копирования и восстановления.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="use-azure-blob-storage"></a>Использование хранилища BLOB-объектов Azure 

Используйте хранилище BLOB-объектов Azure для миграции на SQL Управляемый экземпляр с включенной службой ARC.

Этот метод использует хранилище BLOB-объектов Azure в качестве временного места хранения, к которому можно выполнить резервное копирование, а затем восстановить из.

### <a name="prerequisites"></a>Предварительные требования

- [Установите Azure Data Studio](install-client-tools.md).
- [Установка Обозреватель службы хранилища Azure](https://azure.microsoft.com/features/storage-explorer/)
- Подписка Azure.

### <a name="step-1-provision-azure-blob-storage"></a>Шаг 1. Инициализация хранилища BLOB-объектов Azure

1. Выполните действия, описанные в статье [Создание учетной записи хранилища BLOB-объектов Azure](../../storage/blobs/storage-blob-create-account-block-blob.md?tabs=azure-portal) .
1. Запустить Обозреватель службы хранилища Azure
1. [Войдите в Azure](../../vs-azure-tools-storage-manage-with-storage-explorer.md?tabs=windows#sign-in-to-azure) , чтобы получить доступ к хранилищу BLOB-объектов, созданному на предыдущем шаге.
1. Щелкните правой кнопкой мыши учетную запись хранилища BLOB-объектов и выберите **создать контейнер BLOB-объектов** , чтобы создать контейнер, в котором будет храниться файл резервной копии.

### <a name="step-2-get-storage-blob-credentials"></a>Шаг 2. получение учетных данных BLOB-объекта хранилища

1. В Обозреватель службы хранилища Azure щелкните правой кнопкой мыши контейнер больших двоичных объектов, который был только что создан, и выберите **получить подпись общего доступа** .

1. Выберите **Чтение**, **запись** и **список** .

1. Выберите **Создать**.

   Запишите URI и строку запроса с этого экрана. Они понадобятся в последующих шагах. Нажмите кнопку " **Копировать** ", чтобы сохранить его в блокноте или OneNote и т. д.

1. Закройте окно **подписанного общего доступа** .

### <a name="step-3-backup-database-file-to-azure-blob-storage"></a>Шаг 3. Резервное копирование файла базы данных в хранилище BLOB-объектов Azure

На этом шаге мы подключимся к источнику SQL Server и создадим файл резервной копии базы данных, которую мы хотим перенести в SQL Управляемый экземпляр-дугу Azure.

1. Запустить Azure Data Studio
1. Подключение к экземпляру SQL Server с базой данных, которую необходимо перенести в SQL Управляемый экземпляр, в службе "Дуга Azure"
1. Щелкните правой кнопкой мыши базу данных и выберите команду **создать запрос** .
1. Подготовьте запрос в следующем формате, заменив заполнители, указываемые с `<...>` помощью сведений из подписанного URL, на предыдущих шагах.  После замены значений выполните запрос.

   ```sql
   IF NOT EXISTS  
   (SELECT * FROM sys.credentials
   WHERE name = 'https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>')  
   CREATE CREDENTIAL [https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>]
     WITH IDENTITY = 'SHARED ACCESS SIGNATURE',  
      SECRET = '<SAS_TOKEN>';  
   ```

1. Аналогичным образом подготовьте команду **BACKUP DATABASE** , как показано ниже, чтобы создать файл резервной копии в контейнере больших двоичных объектов.  После замены значений выполните запрос.

   ```sql
   BACKUP DATABASE <database name> TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>'
   ```

1. Откройте Обозреватель службы хранилища Azure и проверьте, что файл резервной копии, созданный на предыдущем шаге, отображается в контейнере больших двоичных объектов.

### <a name="step-4-restore-the-database-from-azure-blob-storage-to-sql-managed-instance---azure-arc"></a>Шаг 4. Восстановление базы данных из хранилища BLOB-объектов Azure в SQL Управляемый экземпляр в Azure Arc

1. В Azure Data Studio Войдите и подключитесь к службе SQL Управляемый экземпляр (Azure ARC).
1. Разверните узел **системные базы данных**, щелкните правой кнопкой мыши базу данных **master** и выберите **создать запрос**.
1. В окне редактора запросов Подготовьте и выполните тот же запрос из предыдущего шага, чтобы создать учетные данные.

   ```sql
   IF NOT EXISTS  
   (SELECT * FROM sys.credentials
   WHERE name = 'https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>')  
   CREATE CREDENTIAL [https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>]
     WITH IDENTITY = 'SHARED ACCESS SIGNATURE',  
     SECRET = '<SAS_TOKEN>';  
   ```

1. Подготовьте и выполните приведенную ниже команду, чтобы убедиться, что файл резервной копии доступен для чтения и не поврежден.

   ```console
   RESTORE FILELISTONLY FROM URL = 'https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>/<file name>.bak'
   ```

1. Подготовьте и выполните команду **RESTORE DATABASE** , как показано ниже, чтобы восстановить файл резервной копии в базе данных на SQL управляемый экземпляр — в службе "Дуга Azure".

   ```sql
   RESTORE DATABASE <database name> FROM URL = 'https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>/<file name>'
   WITH MOVE 'Test' to '/var/opt/mssql/data/<file name>.mdf'
   ,MOVE 'Test_log' to '/var/opt/mssql/data/<file name>.ldf'
   ,RECOVERY  
   ,REPLACE  
   ,STATS = 5;  
   GO
   ```

Дополнительные сведения о резервном копировании на URL-адрес см. здесь:

[Резервное копирование в документы URL](/sql/relational-databases/backup-restore/sql-server-backup-to-url)

[Резервное копирование на URL-адрес с помощью SQL Server Management Studio (SSMS)](/sql/relational-databases/tutorial-sql-server-backup-and-restore-to-azure-blob-storage-service)

-------

## <a name="method-2-copy-the-backup-file-into-an-azure-sql-managed-instance---azure-arc-pod-using-kubectl"></a>Способ 2. копирование файла резервной копии в Azure SQL Управляемый экземпляр — модуль ARC для Azure с помощью kubectl

Этот метод показывает, как создать файл резервной копии, созданный с помощью любого метода, а затем скопировать его в локальное хранилище в экземпляре-устройстве управляемого экземпляра SQL Azure, чтобы можно было выполнить восстановление из подобной обычной файловой системы Windows или Linux. В этом сценарии вы будете использовать команду, `kubectl cp` чтобы скопировать файл из одного места в файловую систему Pod.

### <a name="prerequisites"></a>Предварительные требования

- Установка и настройка kubectl для указания на кластер Kubernetes, в котором развернуты службы данных Azure Arc
- Установите такой инструмент, как Azure Data Studio или SQL Server сервер управления, подключенный к SQL Server, где нужно создать файл резервной копии, или существующий bak-файл уже создан в локальной файловой системе.

### <a name="step-1-backup-the-database-if-you-havent-already"></a>Шаг 1. Создайте резервную копию базы данных, если вы еще этого не сделали

Создайте резервную копию SQL Server базы данных по локальному пути, как при обычной SQL Server резервном копировании на диск:

 ```sql
BACKUP DATABASE Test
TO DISK = 'c:\tmp\test.bak'
WITH FORMAT, MEDIANAME = 'Test’ ;
GO
```

### <a name="step-2-copy-the-backup-file-into-the-pods-file-system"></a>Шаг 2. копирование файла резервной копии в файловую систему Pod

Найдите имя Pod, в котором развернут экземпляр SQL. Обычно он должен выглядеть следующим образом: `pod/<sqlinstancename>-0`

Получите список всех модулей Pod, выполнив команду.

 ```console
kubectl get pods -n <namespace of data controller>
```

Пример

Скопируйте файл резервной копии из локального хранилища в модуль SQL в кластере.

 ```console
kubectl cp <source file location> <pod name>:var/opt/mssql/data/<file name> -n <namespace name>

#Example:
kubectl cp C:\Backupfiles\test.bak sqlinstance1-0:var/opt/mssql/data/test.bak -n arc
```

### <a name="step-3-restore-the-database"></a>Шаг 3. Восстановление базы данных

Подготовка и выполнение команды Restore для восстановления файла резервной копии в управляемый экземпляр SQL Azure с помощью Azure Arc

```sql
RESTORE DATABASE test FROM DISK = '/var/opt/mssql/data/<file name>.bak'
WITH MOVE '<database name>' to '/var/opt/mssql/data/<file name>.mdf'  
,MOVE '<database name>' to '/var/opt/mssql/data/<file name>_log.ldf'  
,RECOVERY  
,REPLACE  
,STATS = 5;  
GO
```

Пример

```sql
RESTORE DATABASE test FROM DISK = '/var/opt/mssql/data/test.bak'
WITH MOVE 'test' to '/var/opt/mssql/data/test.mdf'  
,MOVE 'test' to '/var/opt/mssql/data/test_log.ldf'  
,RECOVERY  
,REPLACE  
,STATS = 5;  
GO
```


## <a name="next-steps"></a>Дальнейшие действия

[Дополнительные сведения о функциях и возможностях SQL Управляемый экземпляр с поддержкой ARC в Azure](managed-instance-features.md)

[Начните работу, создав контроллер данных.](create-data-controller.md)

[Контроллер данных уже создан? Создание SQL Управляемый экземпляр с поддержкой дуги Azure](create-sql-managed-instance.md)