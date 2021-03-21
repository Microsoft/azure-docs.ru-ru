---
title: Перенос данных из базы данных PostgreSQL в дугу Azure с поддержкой PostgreSQL Scale Group
titleSuffix: Azure Arc enabled database services
description: Перенос данных из базы данных PostgreSQL в дугу Azure с поддержкой PostgreSQL Scale Group
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: d9cbfc30b10373ad2a4f4304987dac426b5dcabe
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101643581"
---
# <a name="migrate-postgresql-database-to-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Миграция базы данных PostgreSQL в службу Arc Azure с включенной PostgreSQLной группой серверов

В этом документе описаны действия по получению существующей базы данных PostgreSQL (которая не размещается в службах данных с включенной службой ARC) в вашей службе "Дуга Azure" PostgreSQL "Масштабируемая группа серверов".

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="considerations"></a>Рекомендации

Служба "Дуга Azure Enabled PostgreSQL" — это версия PostgreSQL, которая выполняется с включенным расширением Цитусдата. Таким образом, любое средство, которое работает в PostgreSQL за пределами дуги Azure, должно работать с группой серверов PostgreSQL с поддержкой ARC.


Таким образом, набор средств, используемых сегодня для postgres, должен иметь следующие возможности:
1. Резервное копирование базы данных postgres из экземпляра, размещенного за пределами дуги Azure
2. Восстановите ее в группе серверов PostgreSQL в службе "Дуга Azure" с поддержкой геомасштабирования.

Для вас будет оставлено следующее:
- Сброс параметров сервера
- Сброс контекстов безопасности: повторное создание пользователей, ролей и сброс разрешений...

Для выполнения этой операции резервного копирования и восстановления можно использовать любое средство, способное выполнять резервное копирование и восстановление для postgres. Пример:
- Azure Data Studio и его расширение postgres
- `pgcli`
- `pgAdmin`
- `pg_dump`
- `pg_restore`
- `psql`
- ...

## <a name="example"></a>Пример
Давайте продемонстрируем эти шаги с помощью `pgAdmin` средства.
Примите во внимание следующие настройки.
- **Источник:**  
    Сервер postgres, работающий локально на сервере без операционной системы и именуемый ЖЕАНИДСРВ. Он имеет версию 12 и содержит базу данных с именем Мйонпремпостгресдб, содержащую одну таблицу T1, которая содержит 1 строку :::image type="content" source="media/postgres-hyperscale/migrate-pg-source.jpg" alt-text="миграцию-источник":::

- **Местоназначение**  
    Сервер postgres, работающий в среде Azure Arc и именуемый postgres01. Версия 12. В ней нет базы данных, Кроме стандартной базы данных postgres.  
    :::image type="content" source="media/postgres-hyperscale/migrate-pg-destination.jpg" alt-text="Миграция-назначение":::


### <a name="take-a-backup-of-the-source-database-on-premises"></a>Создание резервной копии базы данных-источника в локальной среде

:::image type="content" source="media/postgres-hyperscale/Migrate-PG-Source-Backup.jpg" alt-text="Миграция-источник — резервное копирование":::

Настройте его:
1. Присвойте ему имя файла: **мисаурцебаккуп**
2. Установка формата **Custom** 
 :::image type="content" source="media/postgres-hyperscale/Migrate-PG-Source-Backup2.jpg" alt-text="Миграция-источник — резервная копия — Настройка":::

Резервное копирование успешно завершено:  
:::image type="content" source="media/postgres-hyperscale/Migrate-PG-Source-Backup3.jpg" alt-text="Миграция-источник — резервное копирование — завершено":::

### <a name="create-an-empty-database-on-the-destination-system-in-your-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Создайте пустую базу данных в целевой системе в вашей службе "Дуга Azure" PostgreSQL "Масштабируемая группа серверов"

> [!NOTE]
> Чтобы зарегистрировать экземпляр postgres в `pgAdmin` средстве, необходимо использовать общедоступный IP-адрес экземпляра в кластере Kubernetes и соответствующим образом задать порт и контекст безопасности. Эти сведения можно найти в `psql` строке конечной точки после выполнения следующей команды:

```console
azdata arc postgres endpoint list -n postgres01
```
, Возвращающий результат, например:
```console
[
  {
    "Description": "PostgreSQL Instance",
    "Endpoint": "postgresql://postgres:<replace with password>@12.345.123.456:1234"
  },
  {
    "Description": "Log Search Dashboard",
    "Endpoint": "https://12.345.123.456:12345/kibana/app/kibana#/discover?_a=(query:(language:kuery,query:'custom_resource_name:\"postgres01\"'))"
  },
  {
    "Description": "Metrics Dashboard",
    "Endpoint": "https://12.345.123.456:12345/grafana/d/postgres-metrics?var-Namespace=arc3&var-Name=postgres01"
  }
]
```

Настроим имя целевой базы данных **RESTORED_MyOnPremPostgresDB**.

:::image type="content" source="media/postgres-hyperscale/migrate-pg-destination-dbcreate.jpg" alt-text="Миграция-назначение-DB-создание" lightbox="media/postgres-hyperscale/migrate-pg-destination-dbcreate.jpg":::

### <a name="restore-the-database-in-your-arc-setup"></a>Восстановление базы данных в настройке Arc

:::image type="content" source="media/postgres-hyperscale/migrate-pg-destination-dbrestore.jpg" alt-text="Мигратре-DB — восстановление":::

Настройте восстановление.
1. Укажите файл, содержащий резервную копию для восстановления: **мисаурцебаккуп**
2. Для параметра формат задайте значение **Custom или tar** 
    :::image type="content" source="media/postgres-hyperscale/migrate-pg-destination-dbrestore2.jpg" alt-text="Migrate-DB-Restore-configure"::: .

3. Щелкните **Восстановить**.  

   Восстановление выполнено успешно.  
   :::image type="content" source="media/postgres-hyperscale/migrate-pg-destination-dbrestore3.jpg" alt-text="Миграция-DB-Restore-Completed":::

### <a name="verify-that-the-database-was-successfully-restored-in-your-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Убедитесь, что база данных успешно восстановлена в службе "Дуга Azure Enabled" PostgreSQL "Масштабируемая группа серверов"

Используйте один из следующих методов.

**Из `pgAdmin` :**  

Разверните экземпляр postgres, размещенный в вашей настройке дуги Azure. В базе данных появится таблица, которая была восстановлена, и при выборе данных будет отображаться та же строка, что и в локальном экземпляре:

   :::image type="content" source="media/postgres-hyperscale/migrate-pg-destination-dbrestoreverif.jpg" alt-text="Migrate-DB-Restore-Verification":::

**`psql`Изнутри настройки дуги Azure:**  

В настройке дуги можно использовать `psql` для подключения к экземпляру postgres, установить контекст базы данных в значение `RESTORED_MyOnPremPostgresDB` и запросить данные:

1. Перечислите конечные точки, чтобы помочь в `psql` строке подключения:

   ```console
   azdata arc postgres endpoint list -n postgres01
   [
     {
       "Description": "PostgreSQL Instance",
       "Endpoint": "postgresql://postgres:<replace with password>@12.345.123.456:1234"
     },
     {
       "Description": "Log Search Dashboard",
       "Endpoint": "https://12.345.123.456:12345/kibana/app/kibana#/discover?_a=(query:(language:kuery,query:'custom_resource_name:\"postgres01\"'))"
     },
     {
       "Description": "Metrics Dashboard",
       "Endpoint": "https://12.345.123.456:12345/grafana/d/postgres-metrics?var-Namespace=arc3&var-Name=postgres01"
     }
   ]
   ```

1. В `psql` строке подключения используйте параметр, `-d` чтобы указать имя базы данных. Выполнив приведенную ниже команду, вы получите запрос на ввод пароля:

   ```console
   psql -d RESTORED_MyOnPremPostgresDB -U postgres -h 10.0.0.4 -p 32639
   ```

   `psql` соединяющ.

   ```output
   Password for user postgres:
   psql (10.12 (Ubuntu 10.12-0ubuntu0.18.04.1), server 12.3 (Debian 12.3-1.pgdg100+1))
   WARNING: psql major version 10, server major version 12.
         Some psql features might not work.
   SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
   Type "help" for help.

   RESTORED_MyOnPremPostgresDB=#   
   ```

1. Выберите таблицу, и вы увидите данные, восстановленные из локального экземпляра postgres:

   ```console
   RESTORED_MyOnPremPostgresDB=# select * from t1;
   ```

   ```output
    col1 |    col2
   ------+-------------
       1 | BobbyIsADog
   (1 row)
   ```

> [!NOTE]
> - Вы не увидите настолько значительных преимуществ в области производительности, пока не PostgreSQL в службе "Дуга" Azure, пока не будет выполнено масштабирование и сегменты и распределения данных между рабочими узлами PostgreSQLной группы серверов. См. раздел [дальнейшие действия](#next-steps).
>
> - Сейчас невозможно выполнить переход в Azure Arc на существующий экземпляр postgres, который будет выполняться локально или в любом другом облаке. Иными словами, невозможно установить некоторый тип "агент Arc Azure" на имеющемся экземпляре postgres, чтобы сделать установку postgres доступной для службы "Дуга" Azure. Вместо этого необходимо создать новый экземпляр postgres и передавать в него данные. Для этого можно использовать описанный выше метод, или вы можете использовать любой выбранный вами инструмент ETL.

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с основными понятиями и руководствами по использованию службы "база данных Azure для PostgreSQL", чтобы распределить данные между несколькими узлами PostgreSQL, а также получить преимущества от всех возможностей базы данных Azure для PostgreSQL Scale.
    * [Узлы и таблицы](../../postgresql/concepts-hyperscale-nodes.md)
    * [Определение типа приложения](../../postgresql/concepts-hyperscale-app-type.md)
    * [Выбор столбца распределения](../../postgresql/concepts-hyperscale-choose-distribution-column.md)
    * [Совместное размещение таблиц](../../postgresql/concepts-hyperscale-colocation.md)
    * [Распространение и изменение таблиц](../../postgresql/howto-hyperscale-modify-distributed-tables.md)
    * [Проектирование базы данных с несколькими клиентами](../../postgresql/tutorial-design-database-hyperscale-multi-tenant.md)*
    * [Разработка панели мониторинга для аналитики в режиме реального времени](../../postgresql/tutorial-design-database-hyperscale-realtime.md)*

> * В этих документах пропустите разделы **Вход в портал Azure** и **Создайте базу данных Azure для postgres-Scale (Цитус)**. Реализуйте оставшиеся шаги в развертывании Azure ARC. Эти разделы относятся к службе "база данных Azure для PostgreSQL" (Цитус), предлагаемой в качестве службы PaaS в облаке Azure, но другие части документов напрямую применимы к вашей геомасштабированию Azure с включенной PostgreSQL.

- [Масштабирование группы серверов с Гипермасштабированием Базы данных Azure для PostgreSQL](scale-out-postgresql-hyperscale-server-group.md)
