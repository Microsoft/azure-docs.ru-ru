---
title: Конфигурация данных об использовании и диагностике Azure SQL
description: Узнайте, как настроить данные об использовании и диагностике в Azure SQL Server.
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 08/04/2020
ms.openlocfilehash: 81b2aabbae148faec06c9ab420bdfecde475b4bb
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97605012"
---
# <a name="azure-sql-edge-usage-and-diagnostics-data-configuration"></a>Конфигурация данных об использовании и диагностике Azure SQL

По умолчанию Azure SQL ребр собирает сведения о том, как клиенты используют приложение. В частности, Azure SQL ребр собирает сведения о процессах развертывания, использовании и производительности. Эти сведения помогают корпорации Майкрософт улучшать продукты и удовлетворять ожидания клиентов. Например, корпорация Майкрософт собирает сведения о том, какие типы кодов ошибок сталкиваются с клиентами, чтобы мы могли исправить связанные с ними ошибки, улучшить документацию по использованию Azure SQL ребра и определить, следует ли добавлять к продукту компоненты для лучшего обслуживания клиентов.

При этом, используя этот механизм, корпорация Майкрософт не собирает следующие данные:

- Любые значения из пользовательских таблиц.
- Любые учетные данные для входа или другие сведения для проверки подлинности.
- Любые личные данные или клиентский пользователь.

В примере ниже показано, какие именно сведения об использовании компонентов помогают нам улучшить продукт.

Ниже приведен пример запроса из запросов, используемых для сбора данных об использовании и диагностике. Запрос определяет количество и типы различных источников потоковых данных, используемых в Azure SQL Server. Эти данные помогают корпорации Майкрософт понять, какие источники данных потоковой передачи используются чаще всего, чтобы корпорация Майкрософт могла улучшить производительность и взаимодействие с пользователями, связанными с этими источниками данных. 

```sql
select 
count(*) as [count], sum(inputs) as inputs, sum(outputs) as outputs, sum(linked_to_job) 
as linked_to_job, data_source_type
from ( 
select isnull(value,'unknown') as data_source_type, inputs, outputs, linked_to_job
from 
    ( 
        select 
        convert(sysname, lower(substring(ds.location, 0, charindex('://', ds.location))), 1) as data_source_type, 
        isnull(inputs, 0) as inputs, isnull(outputs, 0) as outputs, isnull(js.stream_id/js.stream_id, 0) as linked_to_job 
        from sys.external_streams es
        join sys.external_data_sources ds 
             on es.data_source_id = ds.data_source_id
        left join 
            ( 
            select stream_id, max(cast(is_input as int)) inputs, max(cast(is_output as int)) outputs 
            from sys.external_job_streams group by stream_id 
            ) js
             on js.stream_id = es.object_id 
    ) ds
left join 
    (
        select value from string_split('edgehub,sqlserver,kafka', ',')) as known_ep on data_source_type = value 
    ) known_ds
group by data_source_type
```

## <a name="disable-usage-and-diagnostic-data-collection"></a>Отключение сбора данных об использовании и диагностике

Сбор данных об использовании и диагностике в Azure SQL Server можно отключить с помощью любого из приведенных ниже методов.

> [!NOTE]
> Данные об использовании и диагностике не могут быть отключены для версии разработчика.

### <a name="disable-usage-and-diagnostics-using-environment-variables"></a>Отключение использования и диагностики с помощью переменных среды

Чтобы отключить сбор данных об использовании и диагностике на границе Azure SQL, добавьте следующую переменную среды и задайте для нее значение `*False*` . Дополнительные сведения о настройке Azure SQL с помощью переменных среды см. в разделе [Настройка с помощью переменных среды](configure.md#configure-by-using-environment-variables).

`MSSQL_TELEMETRY_ENABLED = TRUE | FALSE`

- TRUE — включает сбор данных об использовании и диагностике. Это конфигурация по умолчанию.
- FALSE — отключает сбор данных об использовании и диагностике.

### <a name="disable-usage-and-diagnostics-using-mssqlconf-file"></a>Отключение использования и диагностики с помощью файла MSSQL. conf

Чтобы отключить сбор данных об использовании и диагностике на границе Azure SQL, добавьте следующие строки в файл MSSQL. conf на постоянном диске хранилища, сопоставленном с папкой/Вар/ОПТ/мсскл/в модуле SQL Server. Дополнительные сведения о настройке Azure SQL с помощью файла MSSQL. conf см. в разделе [Настройка с помощью файла MSSQL. conf](configure.md#configure-by-using-an-mssqlconf-file).

```ini
[telemetry]
customerfeedback = false
```

## <a name="local-audit-of-usage-and-diagnostic-data-collection"></a>Локальный аудит сбора данных об использовании и диагностике

Локальный компонент аудита Azure SQL и сбора диагностических данных может записывать данные, собранные службой, в указанную папку, представляющую данные (журналы), которые будут отправляться в корпорацию Майкрософт. Локальный аудит позволяет клиентам просмотреть все данные, которые корпорация Майкрософт собирает с помощью этой функции для обеспечения соответствия, выполнения нормативных требований или соблюдения конфиденциальности.

### <a name="enable-local-audit-of-usage-and-diagnostics-data"></a>Включить локальный аудит данных об использовании и диагностике

Включение локального аудита и данных диагностики на границе Azure SQL

1. Создайте целевой каталог для нового локального хранилища журналов аудита. Этот целевой каталог может находиться либо на узле, либо в контейнере. В приведенном ниже примере целевой каталог создается в том же томе подключения, который сопоставлен с/Вар/ОПТ/мсскл/путем в SQL Server.

   ```bash
   sudo mkdir <host mount path>/audit
   ```

2. Настройте аудит данных об использовании и диагностике, используя либо переменные среды, либо файл MSSQL. conf.

   - Использование переменных среды. Добавьте следующую переменную среды в развертывание SQL ребра и укажите целевой каталог для файлов аудита.
   
     `*MSSQL_TELEMETRY_DIR = <host mount path>/audit*`
   
   - С помощью файла MSSQL. conf добавьте следующие строки в файл MSSQL. conf и укажите целевой каталог для файлов аудита.
       ```ini
       [telemetry]
       userrequestedlocalauditdirectory  = <host mount path>/audit
       ```  

## <a name="next-steps"></a>Дальнейшие действия

- [Подключение к службе SQL Azure для пограничных вычислений](connect.md)
- [Создание комплексного решения для Интернета вещей с помощью SQL для пограничных вычислений](tutorial-deploy-azure-resources.md)
