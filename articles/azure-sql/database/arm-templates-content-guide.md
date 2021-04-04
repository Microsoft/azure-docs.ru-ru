---
title: Шаблоны Azure Resource Manager — База данных SQL Azure и Управляемый экземпляр SQL
description: Использование шаблонов Azure Resource Manager для создания и настройки Базы данных SQL Azure и Управляемого экземпляра SQL Azure.
services: sql-database
ms.service: sql-db-mi
ms.subservice: service
ms.custom: overview-samples sqldbrb=2
ms.devlang: ''
ms.topic: guide
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein
ms.date: 02/04/2019
ms.openlocfilehash: 5202145af6f60f34bdce57aa94f4fec92a8b227f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92677256"
---
# <a name="azure-resource-manager-templates-for-azure-sql-database--sql-managed-instance"></a>Шаблоны Azure Resource Manager для Базы данных SQL Azure и Управляемого экземпляра SQL
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Шаблоны Azure Resource Manager позволяют определять инфраструктуру как код и развертывать решения в облаке Azure для Базы данных SQL и Управляемого экземпляра SQL Azure.

## <a name="azure-sql-database"></a>[База данных SQL Azure](#tab/single-database)

В следующей таблице представлены ссылки на шаблоны Azure Resource Manager для Базы данных SQL Azure.

|Ссылка |Описание|
|---|---|
| [База данных SQL](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sql-database-transparent-encryption-create) | Этот шаблон Azure Resource Manager создает отдельную базу данных в службе "База данных SQL Azure" и настраивает правила брандмауэра для IP-адресов на уровне сервера. |
| [Server](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sql-logical-server) | Этот шаблон Azure Resource Manager создает логический сервер Базы данных SQL Azure. |
| [Эластичный пул](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sql-elastic-pool-create) | Этот шаблон позволяет развернуть эластичный пул и назначить ему базы данных. |
| [Группы отработки отказа](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sql-with-failover-group) | Этот шаблон создает два сервера, отдельную базу данных и группу отработки отказа в Базе данных SQL Azure.|
| [Обнаружение угроз](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sql-threat-detection-db-policy-multiple-databases) | Этот шаблон позволяет развернуть сервер и набор баз данных с включенным обнаружением угроз и адресом электронной почты для оповещений для каждой базы данных. Обнаружение угроз является частью предложения Расширенной защиты от угроз (ATP) SQL и предоставляет слой безопасности, который отвечает на потенциальные угрозы через серверы и базы данных.|
| [Аудит в Хранилище BLOB-объектов Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sql-auditing-server-policy-to-blob-storage) | Этот шаблон позволяет развернуть сервер с включенным аудитом для записи журналов аудита в Хранилище BLOB-объектов. Аудит базы данных SQL Azure отслеживает события базы данных и записывает их в журнал аудита, который можно разместить в учетной записи хранения Azure, рабочей области OMS или Центрах событий.|
| [Аудит в концентраторе событий Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sql-auditing-server-policy-to-eventhub) | Этот шаблон позволяет развернуть сервер с включенным аудитом для записи журналов аудита в существующий концентратор событий. Чтобы отправить события аудита в Центры событий, настройте параметры аудита (`Enabled` `State`) и задайте для параметра `IsAzureMonitorTargetEnabled` значение `true`. Кроме того, настройте параметры диагностики, указав категорию журналов диагностики `SQLSecurityAuditEvents` в базе данных `master` (для аудита на уровне сервера). Аудит отслеживает события базы данных и записывает их в журнал аудита, который можно разместить в учетной записи хранения Azure, рабочей области OMS или концентраторах событий.|
| [Веб-приложение Azure с Базой данных SQL](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-sql-database) | Этот пример создает бесплатное веб-приложение Azure и базу данных в Базе данных SQL Azure с уровнем обслуживания "Базовый".|
| [Веб-приложение Azure и Кэш Redis с Базой данных SQL](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-redis-cache-sql-database) | Этот шаблон создает веб-приложение, Redis Cache и базу данных в той же группе ресурсов, а также две строки подключения в веб-приложении для базы данных и Redis Cache.|
| [Импорт данных из Хранилища BLOB-объектов с помощью ADF версии 2](https://github.com/Azure/azure-quickstart-templates/tree/master/101-data-factory-v2-blob-to-sql-copy) | Этот шаблон Azure Resource Manager создает экземпляр Фабрики данных Azure версии 2, который копирует данные из Хранилища BLOB-объектов Azure в Базу данных SQL.|
| [Кластер HDInsight с базой данных](https://github.com/Azure/azure-quickstart-templates/tree/master/101-hdinsight-linux-with-sql-database) | Этот шаблон создает кластер HDInsight, логический сервер SQL, базу данных и две таблицы. Этот шаблон применяется в статье [Использование Apache Sqoop с Hadoop в HDInsight](../../hdinsight/hadoop/hdinsight-use-sqoop.md). |
| [Приложение логики Azure, выполняющее хранимую процедуру SQL по расписанию](https://github.com/Azure/azure-quickstart-templates/tree/master/101-logic-app-sql-proc) | Этот шаблон создает приложение логики, которое будет запускать хранимую процедуру SQL по расписанию. Все аргументы для процедуры можно поместить в раздел body шаблона.|

## <a name="azure-sql-managed-instance"></a>[Управляемый экземпляр SQL Azure](#tab/managed-instance)

В следующей таблице представлены ссылки на шаблоны Azure Resource Manager для Управляемого экземпляра SQL Azure.

|Ссылка|Описание|
|---|---|
| [Управляемый экземпляр SQL в новой виртуальной сети](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sqlmi-new-vnet) | Этот шаблон Azure Resource Manager создает настроенную виртуальную сеть Azure и управляемый экземпляр в виртуальной сети. |
| [Сетевая среда для Управляемого экземпляра SQL](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sql-managed-instance-azure-environment) | Это развертывание создаст настроенную виртуальную сеть Azure с двумя подсетями: одна будет выделена для управляемых экземпляров, а другая — для других ресурсов (например, виртуальных машин, сред Службы приложений и т. д.). Этот шаблон создает правильно настроенные сетевые среды, в которых можно развернуть управляемые экземпляры. |
| [Управляемый экземпляр SQL с подключением "точка — сеть"](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sqlmi-new-vnet-w-point-to-site-vpn) | Это развертывание создает виртуальную сеть Azure с двумя подсетями: `ManagedInstance` и `GatewaySubnet`. Управляемый экземпляр SQL развертывается в подсети ManagedInstance. В подсети `GatewaySubnet` будет создается шлюз виртуальной сети, настроенный для VPN-подключения "точка — сеть". |
| [Управляемый экземпляр SQL с виртуальной машиной](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sqlmi-new-vnet-w-jumpbox) | Это развертывание создает виртуальную сеть Azure с двумя подсетями: `ManagedInstance` и `Management`. Управляемый экземпляр SQL развертывается в подсети `ManagedInstance`. Виртуальная машина с последней версией SQL Server Management Studio (SSMS) развертывается в подсети `Management`. |

---