---
title: Функции Apache Cassandra, поддерживаемые API Cassandra для Azure Cosmos DB
description: Сведения о поддержке функций Apache Cassandra в API Cassandra для Azure Cosmos DB
author: TheovanKraay
ms.author: thvankra
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: overview
ms.date: 09/14/2020
ms.openlocfilehash: f48935edd5746b0f3f3440e249024cbe2901317b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100374377"
---
# <a name="apache-cassandra-features-supported-by-azure-cosmos-db-cassandra-api"></a>Функции Apache Cassandra, поддерживаемые API Cassandra для Azure Cosmos DB 
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Azure Cosmos DB — это глобально распределенная многомодельная служба базы данных Майкрософт. Взаимодействие с API Cassandra для Azure Cosmos DB осуществляется посредством клиентских [драйверов](https://cassandra.apache.org/doc/latest/getting_started/drivers.html?highlight=driver) Cassandra с открытым исходным кодом, совместимым с [двоичным сетевым протоколом](https://github.com/apache/cassandra/blob/trunk/doc/native_protocol_v4.spec) Cassandra Query Language (CQL) версии 4. 

С помощью API Cassandra для Azure Cosmos DB вы можете воспользоваться преимуществами API-интерфейсов Apache Cassandra, а также корпоративными возможностями, доступными в Azure Cosmos DB. В их число входят [глобальное распределение](distribute-data-globally.md), [секционирование автоматического горизонтального увеличения масштаба](cassandra-partitioning.md), гарантии доступности и задержки, шифрование при хранении, создание резервных копий и многое другое.

## <a name="cassandra-protocol"></a>Протокол Cassandra 

API Cassandra для Azure Cosmos DB совместим с API-интерфейсом Cassandra Query Language (CQL) версии 3.11 (обратно совместим с версией 2.x). Ниже перечислены поддерживаемые команды CQL, средства, ограничения и исключения. Любой драйвер клиента, который распознает эти протоколы, должен иметь возможность подключения к API Cassandra для Azure Cosmos DB.

## <a name="cassandra-driver"></a>Драйвер Cassandra

API Cassandra для Azure Cosmos DB поддерживает следующие версии драйверов Cassandra:

* [Java 3.5 и выше](https://github.com/datastax/java-driver)  
* [C# 3.5 и выше](https://github.com/datastax/csharp-driver)  
* [Nodejs 3.5 и выше](https://github.com/datastax/nodejs-driver)  
* [Python 3.15 и выше](https://github.com/datastax/python-driver)  
* [C++ 2.9](https://github.com/datastax/cpp-driver)   
* [PHP 1.3](https://github.com/datastax/php-driver)  
* [Gocql](https://github.com/gocql/gocql)  
 

## <a name="cql-data-types"></a>Типы данных CQL 

API Cassandra для Azure Cosmos DB поддерживает следующие типы данных CQL:

|Тип  |Поддерживаются: |
|---------|---------|
| ascii  | Да |
| BIGINT  | Да |
| большой двоичный объект  | Да |
| Логическое  | Да |
| Счетчик  | Да |
| Дата  | Да |
| Decimal  | Да |
| double  | Да |
| FLOAT  | Да |
| frozen  | Да |
| inet  | Да |
| INT  | Да |
| list  | Да |
| set  | Да |
| smallint  | Да |
| text  | Да |
| time  | Да |
| TIMESTAMP  | Да |
| timeuuid  | Да |
| tinyint  | Да |
| tuple  | Да |
| uuid  | Да |
| varchar  | Да |
| varint  | Да |
| tuples | Да | 
| udts  | Да |
| карта | Да |

Static поддерживается для объявления типа данных.

## <a name="cql-functions"></a>Функции CQL

API Cassandra для Azure Cosmos DB поддерживает следующие функции CQL:

|Get-Help  |Поддерживаются: |
|---------|---------|
| Токен * | Да |
| ttl *** | Да |
| writetime *** | Да |
| cast ** | Да |

> [!NOTE] 
> \* API Cassandra поддерживает токен в качестве проекции или селектора и разрешает параметр token(pk) в левой части предложения WHERE. Например, `WHERE token(pk) > 1024` поддерживается, но `WHERE token(pk) > token(100)` — **нет**.  
> \*\* Функция `cast()` не может иметь вложенные элементы в API Cassandra. Например, `SELECT cast(count as double) FROM myTable` поддерживается, но `SELECT avg(cast(count as double)) FROM myTable` — **нет**.    
> \*\*\* Пользовательские метки времени и срок жизни, заданный с помощью параметра `USING`, применяются на уровне строк (а не ячейки).



Агрегатные функции:

|Get-Help  |Поддерживаются: |
|---------|---------|
| avg | Да |
| count | Да |
| Min | Да |
| max | Да |
| Sum | Да |

> [!NOTE]
> Агрегатная функция работает с обычными столбцами, но агрегирование для кластеризованных столбцов **не** поддерживается.


Функции преобразования больших двоичных объектов
 
|Get-Help  |Поддерживаются: |
|---------|---------|
| typeAsBlob(value)   | Да |
| blobAsType(value) | Да |


Функции UUID и timeuuid
 
|Get-Help  |Поддерживаются: |
|---------|---------|
| dateOf()  | Да |
| now()  | Да |
| minTimeuuid()  | Да |
| unixTimestampOf()  | Да |
| toDate(timeuuid)  | Да |
| toTimestamp(timeuuid)  | Да |
| toUnixTimestamp(timeuuid)  | Да |
| toDate(timestamp)  | Да |
| toUnixTimestamp(timestamp)  | Да |
| toTimestamp(date)  | Да |
| toUnixTimestamp(date) | Да |


  
## <a name="cql-commands"></a>Команды CQL

Azure Cosmos DB поддерживает следующие команды базы данных для учетных записей API Cassandra.

|Get-Help  |Поддерживаются: |
|---------|---------|
| ALLOW FILTERING | Да |
| ALTER KEYSPACE | Н/д (служба PaaS, внутреннее управление репликацией)|
| ALTER MATERIALIZED VIEW | Нет |
| ALTER_ROLE | Нет |
| ALTER TABLE | Да |
| ALTER TYPE | Нет |
| ALTER USER | Нет |
| BATCH | Да (только в незарегистрированном пакете)|
| COMPACT STORAGE | Н/д (служба PaaS) |
| CREATE AGGREGATE | Нет | 
| CREATE CUSTOM INDEX (SASI) | Нет |
| CREATE INDEX | Да (без [указания имени индекса](cassandra-secondary-index.md); индексы в ключах кластеризации или полные замороженные коллекции не поддерживаются) |
| CREATE FUNCTION | Нет |
| CREATE KEYSPACE (параметры репликации игнорируются) | Да |
| CREATE MATERIALIZED VIEW | Нет |
| CREATE TABLE | Да |
| CREATE TRIGGER | Нет |
| CREATE TYPE | Да |
| CREATE ROLE | Нет |
| CREATE USER (не рекомендуется во встроенном Cassandra Apache) | Нет |
| DELETE | Да |
| DISTINCT | Нет |
| DROP AGGREGATE | Нет |
| .DROP FUNCTION | Нет |
| DROP INDEX | Да |
| DROP KEYSPACE | Да |
| DROP MATERIALIZED VIEW | Нет |
| DROP ROLE | Нет |
| DROP TABLE | Да |
| DROP_TRIGGER | Нет | 
| DROP TYPE | Да |
| DROP USER (не рекомендуется во встроенном Cassandra Apache) | Нет |
| GRANT | Нет |
| INSERT | Да |
| LIST PERMISSIONS | Нет |
| LIST ROLES | Нет |
| LIST USERS (не рекомендуется во встроенном Cassandra Apache) | Нет |
| REVOKE | Нет |
| SELECT | Да |
| UPDATE | Да |
| TRUNCATE | Нет |
| USE | Да |

## <a name="lightweight-transactions-lwt"></a>Упрощенные транзакции (LWT)

| Компонент  |Поддерживается |
|---------|---------|
| DELETE IF EXISTS | Да |
| Условия DELETE | Нет |
| INSERT IF NOT EXISTS | Да |
| UPDATE IF EXISTS | Да |
| UPDATE IF NOT EXISTS | Да |
| Условия UPDATE | Нет |

## <a name="cql-shell-commands"></a>Команды оболочки CQL

Azure Cosmos DB поддерживает следующие команды базы данных для учетных записей API Cassandra.

|Get-Help  |Поддерживается |
|---------|---------|
| ЗАПИСАТЬ | Да |
| CLEAR | Да |
| CONSISTENCY * | Н/Д |
| КОПИРОВАТЬ | Нет |
| DESCRIBE | Да |
| cqlshExpand | Нет |
| EXIT | Да |
| Имя_для_входа | Н/П (функция CQL не поддерживается `USER`, поэтому `LOGIN` не нужно использовать) |
| PAGING | Да |
| SERIAL CONSISTENCY * | Н/Д |
| SHOW | Да |
| ИСТОЧНИК | Да |
| TRACING | Н/П (API Cassandra работает на основе Azure Cosmos DB — используйте [журнал ведения диагностики](cosmosdb-monitor-resource-logs.md) для устранения неполадок) |

> [!NOTE] 
> \* В Azure Cosmos DB согласованность действует иначе, дополнительные сведения см. [здесь](cassandra-consistency.md).  


## <a name="json-support"></a>Поддержка JSON
|Get-Help  |Поддерживаются: |
|---------|---------|
| SELECT JSON | Да |
| INSERT JSON | Да |
| fromJson() | Нет |
| toJson() | Нет |


## <a name="cassandra-api-limits"></a>Ограничения API Cassandra

API Cassandra для Azure Cosmos DB не имеет ограничений на размер данных, хранящихся в таблице. В таблицах могут храниться сотни терабайт или петабайт данных с соблюдением ограничений для ключей секций. Каждый эквивалент сущности или строки не имеет никаких ограничений на число столбцов. Однако общий размер сущности не должен превышать 2 МБ. Объем данных на ключ секции не может превышать 20 ГБ, как и во всех других API.

## <a name="tools"></a>Инструменты 

API Cassandra для Azure Cosmos DB — это платформа управляемой службы. Ей не требуются расходы на управление или такие служебные программы, как сборщик мусора, виртуальная машина Java (JVM) и Nodetool для управления кластером. Она поддерживает средства, такие как cqlsh, использующее совместимость с двоичным CQLv4. 

* Кроме того, для управления учетной записью применяются обозреватель данных на портале Azure, метрики, журналы диагностики, PowerShell и интерфейс командной строки.

## <a name="hosted-cql-shell-preview"></a>Размещенная оболочка CQL (предварительная версия)

Вы можете открыть размещенную собственную оболочку Cassandra (CQLSH версии 5.0.1) непосредственно из обозревателя данных на [портале Azure](data-explorer.md) или [Обозревателя Azure Cosmos DB](https://cosmos.azure.com/). Перед включением оболочки CQL необходимо включить в учетной записи компонент [Записные книжки](enable-notebooks.md) (если он еще не включен, при щелчке `Open Cassandra Shell` появится запрос). Дополнительные сведения см. в статье [Включение записных книжек для учетных записей Azure Cosmos DB](enable-notebooks.md#supported-regions).

:::image type="content" source="./media/cassandra-support/cqlsh.png" alt-text="Открытие CQLSH":::

С помощью установленной на локальном компьютере программы CQLSH можно также подключиться к API Cassandra в Azure Cosmos DB. Она входит в состав Apache Cassandra 3.1.1 и работает без дополнительной настройки, задавая переменные среды. В следующих разделах приведены инструкции по установке, настройке и подключению к API Cassandra в Azure Cosmos DB в системах с Windows или Linux с помощью CQLSH.

> [!NOTE]
> Подключения к API Cassandra для Azure Cosmos DB не будут работать с версиями DataStax Enterprise (DSE) для CQLSH. При подключении к API Cassandra убедитесь, что вы используете только версии с открытым кодом Apache Cassandra для CQLSH. 

**Windows:**

При использовании Windows рекомендуется включить [файловую систему Windows для Linux](/windows/wsl/install-win10#install-the-windows-subsystem-for-linux). Затем вы можете выполнять приведенные ниже команды Linux.

**UNIX/Linux и Mac:**

```bash
# Install default-jre and default-jdk
sudo apt install default-jre
sudo apt-get update
sudo apt install default-jdk

# Import the Baltimore CyberTrust root certificate:
curl https://cacert.omniroot.com/bc2025.crt > bc2025.crt
keytool -importcert -alias bc2025ca -file bc2025.crt

# Install the Cassandra libraries in order to get CQLSH:
echo "deb http://www.apache.org/dist/cassandra/debian 311x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
curl https://downloads.apache.org/cassandra/KEYS | sudo apt-key add -
sudo apt-get update
sudo apt-get install cassandra

# Export the SSL variables:
export SSL_VERSION=TLSv1_2
export SSL_VALIDATE=false

# Connect to Azure Cosmos DB API for Cassandra:
cqlsh <YOUR_ACCOUNT_NAME>.cassandra.cosmosdb.azure.com 10350 -u <YOUR_ACCOUNT_NAME> -p <YOUR_ACCOUNT_PASSWORD> --ssl

```

Все операции CRUD, выполняемые с помощью пакета SDK, совместимого с CQL версии 4, возвращают дополнительные сведения об ошибке и использованных единицах запроса. Команды DELETE и UPDATE должны обрабатываться с учетом управления ресурсами, чтобы обеспечить наиболее эффективное использование подготовленной пропускной способности.

* Примечание. Значение gc_grace_seconds должно равняться нулю, если оно указывается.

```csharp
var tableInsertStatement = table.Insert(sampleEntity); 
var insertResult = await tableInsertStatement.ExecuteAsync(); 
 
foreach (string key in insertResult.Info.IncomingPayload) 
        { 
            byte[] valueInBytes = customPayload[key]; 
            double value = Encoding.UTF8.GetString(valueInBytes); 
            Console.WriteLine($"CustomPayload:  {key}: {value}"); 
        } 
```

## <a name="consistency-mapping"></a>Сопоставление согласованности 

API Cassandra для Azure Cosmos DB предоставляет согласованные операции чтения.  Сопоставление согласованности подробно описано [здесь](./cassandra-consistency.md#mapping-consistency-levels).

## <a name="permission-and-role-management"></a>Управление разрешениями и ролями

Azure Cosmos DB поддерживает управление доступом на основе ролей Azure (Azure RBAC) для подготовки, смены ключей, просмотра метрик, а также пароли (ключи) для чтения и записи или только для чтения, которые можно получить с помощью [портала Azure](https://portal.azure.com). Azure Cosmos DB не поддерживает роли для действий CRUD.

## <a name="keyspace-and-table-options"></a>Параметры пространства ключей и таблицы

Параметры для имени региона, класса, replication_factor и центра обработки данных в команде "Создать пространство ключей" в настоящее время игнорируются. Для добавления регионов система использует базовый метод репликации [глобального распространения](global-dist-under-the-hood.md) Azure Cosmos DB. Если необходимо присутствие данных между регионами, вы можете включить их на уровне учетной записи с помощью PowerShell, интерфейса командной строки или портала. Дополнительные сведения см. в статье [Добавление регионов](how-to-manage-database-account.md#addremove-regions-from-your-database-account). Невозможно отключить Durable_writes, потому что Azure Cosmos DB гарантирует длительность каждой записи. В каждом регионе Azure Cosmos DB реплицирует данные по набору реплик, который состоит из 4 реплик. [Конфигурацию](global-dist-under-the-hood.md) этого набора реплик невозможно изменить.
 
При создании таблицы игнорируются все параметры, кроме gc_grace_seconds, для которого должно быть задано значение ноль.
Пространство ключей и таблица имеют дополнительную опцию с именем "cosmosdb_provisioned_throughput" с минимальным значением 400 единиц запроса/секунду. Пропускная способность пространства ключей позволяет разделять пропускную способность между несколькими таблицами, и это полезно для сценариев, когда не все таблицы используют предоставленную пропускную способность. Команда "Alter Table" позволяет изменить выделенную пропускную способность по регионам. 

```
CREATE  KEYSPACE  sampleks WITH REPLICATION = {  'class' : 'SimpleStrategy'}   AND cosmosdb_provisioned_throughput=2000;  

CREATE TABLE sampleks.t1(user_id int PRIMARY KEY, lastname text) WITH cosmosdb_provisioned_throughput=2000; 

ALTER TABLE gks1.t1 WITH cosmosdb_provisioned_throughput=10000 ;

```
## <a name="secondary-index"></a>Вторичный индекс
API Cassandra поддерживает вторичные индексы для всех типов данных, кроме типов замороженных коллекций, типов Decimal и Variant. 

## <a name="usage-of-cassandra-retry-connection-policy"></a>Использование политики повторных подключений Cassandra

Azure Cosmos DB — это система управления ресурсами. Это означает, что в данную секунду можно выполнять определенное количество операций на основе единиц запроса, потребляемых операциями. Если приложение превысит этот лимит в течение данной секунды, запросы будут ограничены по скорости, и будут выданы исключения. API Cassandra в Azure Cosmos DB преобразует эти исключения в ошибки перегрузки в собственном протоколе Cassandra. Чтобы приложение могло перехватывать и повторять запросы в случае ограничения скорости, предусмотрены расширения [spark](https://mvnrepository.com/artifact/com.microsoft.azure.cosmosdb/azure-cosmos-cassandra-spark-helper) и [Java](https://github.com/Azure/azure-cosmos-cassandra-extensions). См. также примеры кода Java для драйверов Datastax версии [3](https://github.com/Azure-Samples/azure-cosmos-cassandra-java-retry-sample) и [4](https://github.com/Azure-Samples/azure-cosmos-cassandra-java-retry-sample-v4) для подключения к API Cassandra в Azure Cosmos DB. Если вы используете другие пакеты SDK для доступа к API Cassandra в Azure Cosmos DB, создайте политику подключения для повторения этих исключений.

## <a name="next-steps"></a>Дальнейшие действия

- Начните с [создания учетной записи API Cassandra, базы данных и таблицы](create-cassandra-api-account-java.md) с помощью приложения Java.
