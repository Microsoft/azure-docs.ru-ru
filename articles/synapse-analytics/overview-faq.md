---
title: Вопросы и ответы (Azure Synapse Analytics)
description: Вопросы и ответы по Azure Synapse Analytics
services: synapse-analytics
author: saveenr
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: overview
ms.date: 10/25/2020
ms.author: saveenr
ms.reviewer: jrasnick
ms.openlocfilehash: 9a0fb8ed8ac54fa866b6db7d8f808c011c0c6758
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101695827"
---
# <a name="azure-synapse-analytics-frequently-asked-questions"></a>Часто задаваемые вопросы по Azure Synapse Analytics

В этом руководстве вы найдете часто задаваемые вопросы по Azure Synapse Analytics.

## <a name="general"></a>Общие сведения

### <a name="q-how-can-i-use-rbac-roles-to-secure-my-workspace"></a>Вопрос. Как я могу использовать роли RBAC для защиты своей рабочей области?

A. Azure Synapse предоставляет ряд ролей и областей для их назначения, что позволяет упростить защиту рабочей области.

Роли RBAC в Synapse:
* администратор Synapse;
* администратор Synapse SQL;
* администратор Synapse Spark;
* участник Synapse (предварительная версия);
* издатель артефактов Synapse (предварительная версия);
* пользователь артефактов Synapse (предварительная версия);
* оператор вычислений Synapse (предварительная версия);
* пользователь учетных данных Synapse (предварительная версия).

Чтобы защитить свою рабочую область Synapse, назначьте роли RBAC следующим областям RBAC:
* Рабочие области
* Пулы Spark
* Среды выполнения интеграции
* Связанные службы
* Учетные данные

Кроме того, при использовании выделенных пулов SQL вы можете применять все известные и привычные функции безопасности.

### <a name="q-how-do-i-control-dedicated-sql-pools-serverless-sql-pools-and-serverless-spark-pools"></a>Вопрос. Как я могу управлять выделенными пулами SQL, бессерверными пулами SQL и бессерверными пулами Spark?

A. В качестве отправной точки Azure Synapse взаимодействует со встроенным механизмом анализа затрат и оповещений о затратах, который предоставляется на уровне подписки Azure.

- Выделенные пулы SQL предоставляют точные данные о затратах и контроль над расходами, поскольку вы создаете выделенные пулы SQL с заданным размером. Кроме того, вы можете управлять доступом конкретных пользователей к созданию или масштабированию выделенных пулов SQL с помощью ролей RBAC в Azure.

- Бессерверные пулы SQL предоставляют средства мониторинга и управления затратами, которые позволяют планировать ежедневные, еженедельные и ежемесячные расходы. Дополнительные сведения см. в статье [Управление затратами для бессерверного пула SQL](./sql/data-processed.md). 

- Бессерверные пулы Spark позволяют ограничить круг пользователей, которые могут создавать пулы Spark, с помощью ролей RBAC в Synapse.  

### <a name="q-will-synapse-workspace-support-folder-organization-of-objects-and-granularity-at-ga"></a>Вопрос. Будет ли рабочая область Synapse в общедоступной версии поддерживать упорядочение объектов по папкам и степень детализации?

A. Рабочие области Synapse поддерживают определяемые пользователем папки.

### <a name="q-can-i-link-more-than-one-power-bi-workspace-to-a-single-azure-synapse-workspace"></a>Вопрос. Можно ли связать с одной рабочей областью Azure Synapse несколько рабочих областей Power BI?
    
A. В настоящее время вы можете связать с рабочей областью Azure Synapse только одну рабочую область Power BI. 

### <a name="q-is-synapse-link-to-cosmos-db-ga"></a>Вопрос. Предоставляется ли Synapse Link для Cosmos DB в общедоступной версии?

A. Synapse Link для Apache Spark предоставляется в общедоступной версии. Synapse Link для бессерверного пула SQL предоставляется в режиме общедоступной предварительной версии.

### <a name="q-does-azure-synapse-workspace-support-cicd"></a>Вопрос. Поддерживает ли рабочая область Azure Synapse конвейеры CI/CD? 

A. Конечно! Все артефакты конвейера, записные книжки, скрипты SQL и определения заданий Spark будут размещаться в Git. Все определения пулов будут размещаться в Git как шаблоны ARM. Управление объектами выделенных пулов SQL (схемы, таблицы, представления и т. д.) будет осуществляться через проекты баз данных с поддержкой CI/CD.

## <a name="pipelines"></a>Pipelines

### <a name="q-how-do-i-ensure-i-know-what-credential-is-being-used-to-run-a-pipeline"></a>Вопрос. Как мне узнать, какие учетные данные используются для выполнения конвейера? 

A. Каждое действие в конвейере Synapse выполняется с учетными данными, которые указаны в связанной службе.

### <a name="q-are-ssis-irs-supported-in-synapse-integrate"></a>Вопрос. Поддерживаются ли среды выполнения интеграции SSIS в Synapse Integrate?

Ответ. Пока что нет. 

### <a name="q-how-do-i-migrate-existing-pipelines-from-azure-data-factory-to-an-azure-synapse-workspace"></a>Вопрос. Как мне перенести существующие конвейеры из Фабрики данных Azure в рабочую область Azure Synapse?

A. В настоящее время вы должны вручную повторно создать конвейеры Фабрики данных Azure и связанные артефакты, экспортировав JSON из исходного конвейера и импортировав его в рабочую область Synapse.

## <a name="apache-spark"></a>Apache Spark

### <a name="q-what-is-the-difference-between-apache-spark-for-synapse-and-apache-spark"></a>Вопрос. В чем разница между Apache Spark для Synapse и Apache Spark?

A. Apache Spark для Synapse — это и есть Apache Spark, но с дополнительной поддержкой интеграции с другими службами (Azure AD, Машинное обучение Azure и т. д.), дополнительными библиотеками (mssparktuils, Hummingbird) и предварительно настроенными конфигурациями производительности.

Любая рабочая нагрузка, которая сейчас выполняется в Apache Spark, будет без изменений работать и в Apache Spark для Azure Synapse. 

### <a name="q-what-versions-of-spark-are-available"></a>Вопрос. Какие версии Spark доступны?

A. Azure Synapse Apache Spark полностью поддерживает Spark 2.4. Полный список основных компонентов и поддерживаемых в настоящее время версий Apache Spark см. [здесь](./spark/apache-spark-version-support.md).

### <a name="q-is-there-an-equivalent-of-dbutils-in-azure-synapse-spark"></a>Вопрос. Существует ли в Azure Synapse Spark эквивалент DButils?

A. Да, Azure Synapse Apache Spark предоставляет библиотеку **mssparkutils**. Полную документацию по ней см. в статье [Введение в служебные программы Microsoft Spark](./spark/microsoft-spark-utilities.md).

### <a name="q-how-do-i-set-session-parameters-in-apache-spark"></a>Вопрос. Как правильно задать параметры сеанса в Apache Spark?

A. Чтобы задать параметры сеанса, используйте магическую команду %%configure. Чтобы параметры вступили в силу, необходимо перезапустить сеанс. 

### <a name="q-how-do-i-set-cluster-level-parameters-in-a-serverless-spark-pool"></a>Вопрос. Как правильно задать параметры на уровне кластера в бессерверном пуле Spark?

A. Чтобы задать параметры на уровне кластера, предоставьте файл spark.conf для пула Spark. С этого момента в пуле будут применяться параметры, заданные в файле конфигурации. 

### <a name="q-can-i-run-a-multi-user-spark-cluster-in-azure-synapse-analytics"></a>Вопрос. Можно ли запустить в Azure Synapse Analytics кластер Spark с несколькими пользователями?
 
A. Azure Synapse предоставляет специализированные механизмы для конкретных вариантов использования. Apache Spark для Synapse разработана как служба заданий, а не модель кластера. Существует два сценария, в которых пользователи хотят использовать модель кластера с несколькими пользователями.

**Сценарий 1. К кластеру за данными для бизнес-аналитики обращаются множество пользователей.**

Чтобы выполнить эту задачу, можно просто подготовить данные с помощью Spark и применить средства Synapse SQL для предоставления пользователям возможности подключить Power BI к полученным наборам данных.

**Сценарий 2. Для экономии средств в одном кластере работают несколько разработчиков.**
 
Чтобы реализовать такой сценарий, следует предоставить каждому разработчику отдельный бессерверный пул Spark с малым объемом ресурсов Spark. Бессерверные пулы Spark не нужно оплачивать, пока они не используются активно, а значит затраты на работу нескольких разработчиков будут минимальными. Эти пулы могут иметь общие метаданные (таблицы Spark), что позволяет легко организовать взаимодействие между ними.

### <a name="q-how-do-i-include-manage-and-install-libraries"></a>Вопрос. Как правильно включать, устанавливать библиотеки и управлять ими?

A.  Внешние пакеты можно установить с помощью файла requirements.txt при создании пула Spark, из рабочей области Synapse или на портале Azure. Подробные сведения см. в статье [Управление библиотеками для Apache Spark в Azure Synapse Analytics](./spark/apache-spark-azure-portal-add-libraries.md).

## <a name="dedicated-sql-pools"></a>Выделенные пулы SQL

### <a name="q-what-are-the-functional-differences-between-dedicated-sql-pools-and-serverless-pools"></a>Вопрос. Каковы функциональные различия между выделенными пулами и бессерверными пулами SQL?

A. Полный список таких различий см. в статье [о различиях в функциях T-SQL в Synapse SQL](./sql/overview-features.md).

### <a name="q-now-that-azure-synapse-is-ga-how-do-i-move-my-dedicated-sql-pools-that-were-previously-standalone-into-azure-synapse"></a>Вопрос. Теперь, после выхода общедоступной версии Azure Synapse, как я могу переместить в Azure Synapse выделенные пулы SQL, которые ранее были автономными? 

A. "Перемещение" или "миграция" в этом сценарии не имеют смысла. Вы можете просто включить новые возможности рабочей области для существующих пулов. Это действие не приводит к критическим изменениям, вам просто станут доступны все новые возможности, включая Synapse Studio, Spark и бессерверные пулы SQL.

### <a name="q-what-is-the-default-deployment-of-dedicated-sql-pools-now"></a>Вопрос. Какое развертывание теперь применяется для выделенных пулов SQL по умолчанию? 

A. По умолчанию все новые выделенные пулы SQL будут развертываться в рабочей области, но вы можете создать выделенный пул SQL (ранее Хранилище данных SQL) в режиме автономного развертывания. 

## <a name="next-steps"></a>Дальнейшие действия

* [Начало работы с Azure Synapse Analytics](get-started.md)
* [Создание рабочей области](quickstart-create-workspace.md)
* [Использование бессерверного пула SQL](quickstart-sql-on-demand.md)
