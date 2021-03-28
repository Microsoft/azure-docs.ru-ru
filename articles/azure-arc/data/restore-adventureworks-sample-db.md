---
title: Восстановление образца базы данных AdventureWorks в SQL Управляемый экземпляр
description: Восстановление образца базы данных AdventureWorks в SQL Управляемый экземпляр
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: a6b3577fe820c9dadcb3c8415aa7a951a3283284
ms.sourcegitcommit: c8b50a8aa8d9596ee3d4f3905bde94c984fc8aa2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/28/2021
ms.locfileid: "105641815"
---
# <a name="restore-the-adventureworks-sample-database-into-sql-managed-instance---azure-arc"></a>Восстановление образца базы данных AdventureWorks в SQL Управляемый экземпляр в Azure Arc

[AdventureWorks](/sql/samples/adventureworks-install-configure) — это образец базы данных, содержащей базу данных OLTP, которая часто используется в руководствах и примерах. Он предоставляется и обслуживается корпорацией Майкрософт в составе [репозитория SQL Server Samples GitHub](https://github.com/microsoft/sql-server-samples/tree/master/samples/databases).

В этом документе описывается простой процесс получения образца базы данных AdventureWorks в SQL Управляемый экземпляр — Azure ARC.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="download-the-adventureworks-backup-file"></a>Скачивание файла резервной копии AdventureWorks

Скачайте файл резервной копии AdventureWorks (BAK) в контейнер SQL Управляемый экземпляр. В этом примере используйте команду, `kubectl exec` чтобы удаленно выполнить команду в контейнере SQL управляемый экземпляр, чтобы скачать bak-файл в контейнер. Скачайте этот файл из любого расположения, доступного в, `wget` Если у вас есть другие файлы резервной копии базы данных, которые нужно запрашивать в контейнере SQL управляемый экземпляр. В контейнере SQL Управляемый экземпляр его легко восстановить с помощью стандартного `RESTORE DATABASE` T-SQL.

Выполните команду, подобную следующей, чтобы загрузить bak-файл, заменив значения имени Pod и имени пространства имен, прежде чем запускать его.
> [!NOTE]
>  Чтобы скачать файл из GitHub, вашему контейнеру потребуется подключение к Интернету через 443.

```console
kubectl exec <SQL pod name> -n <namespace name> -c arc-sqlmi -- wget https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks2019.bak -O /var/opt/mssql/data/AdventureWorks2019.bak
```

Пример

```console
kubectl exec sqltest1-0 -n arc -c arc-sqlmi -- wget https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks2019.bak -O /var/opt/mssql/data/AdventureWorks2019.bak
```

## <a name="restore-the-adventureworks-database"></a>Восстановление базы данных AdventureWorks

Аналогичным образом можно выполнить `kubectl` команду Exec, чтобы использовать `sqlcmd` средство CLI, включенное в контейнер SQL управляемый экземпляр, чтобы выполнить команду T-SQL для восстановления базы данных.

Выполните команду, подобную следующей, чтобы восстановить базу данных. Замените значения имени POD, пароля и имени пространства имен, прежде чем запускать его.

```console
kubectl exec <SQL pod name> -n <namespace name> -c arc-sqlmi -- /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P <password> -Q "RESTORE DATABASE AdventureWorks2019 FROM  DISK = N'/var/opt/mssql/data/AdventureWorks2019.bak' WITH MOVE 'AdventureWorks2017' TO '/var/opt/mssql/data/AdventureWorks2019.mdf', MOVE 'AdventureWorks2017_Log' TO '/var/opt/mssql/data/AdventureWorks2019_Log.ldf'"
```
Пример

```console
kubectl exec sqltest1-0 -n arc -- -c arc-sqlmi /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P MyPassword! -Q "RESTORE DATABASE AdventureWorks2019 FROM  DISK = N'/var/opt/mssql/data/AdventureWorks2019.bak' WITH MOVE 'AdventureWorks2017' TO '/var/opt/mssql/data/AdventureWorks2019.mdf', MOVE 'AdventureWorks2017_Log' TO '/var/opt/mssql/data/AdventureWorks2019_Log.ldf'"
```
