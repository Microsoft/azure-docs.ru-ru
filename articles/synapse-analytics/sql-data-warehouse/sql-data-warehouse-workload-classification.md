---
title: Классификация рабочей нагрузки для выделенного пула SQL
description: Руководство по использованию классификации для управления параллелизмом запросов, важности и ресурсами вычислений для выделенного пула SQL в Azure синапсе Analytics.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 02/04/2020
ms.author: rortloff
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: 7cd3619aa60f1bd8ac13ff767857b44348989285
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98678429"
---
# <a name="workload-classification-for-dedicated-sql-pool-in-azure-synapse-analytics"></a>Классификация рабочей нагрузки для выделенного пула SQL в Azure синапсе Analytics

В этой статье описывается процесс классификации рабочей нагрузки по назначению группы рабочей нагрузки и важности для входящих запросов с выделенными пулами SQL в Azure синапсе.

## <a name="classification"></a>Классификация

> [!Video https://www.youtube.com/embed/QcCRBAhoXpM]

Классификация управления рабочей нагрузкой позволяет применять политики рабочей нагрузки к запросам путем назначения [классов ресурсов](resource-classes-for-workload-management.md#what-are-resource-classes) и [важности](sql-data-warehouse-workload-importance.md).

Существует множество способов классификации рабочих нагрузок хранилища данных, однако самая простая и наиболее распространенная классификация — это загрузка и запрос. Данные загружаются с помощью инструкций INSERT, UPDATE и DELETE.  Для запроса данных используются инструкции SELECT. Решение для хранения данных зачастую имеет политику рабочей нагрузки для действия загрузки, например назначение более высокого класса ресурсов с большим количеством ресурсов. К запросам может применяться другая политика рабочей нагрузки, например низкая важность по сравнению с действиями загрузки.

Также можно определить подклассы рабочих нагрузок загрузки и запросов. Это обеспечивает более полный контроль над рабочими нагрузками. Например, рабочие нагрузки запросов могут состоять из обновлений куба, запросов панели мониторинга или нерегламентированных запросов. Каждую из этих рабочих нагрузок запроса можно классифицировать с помощью различных классов ресурсов или параметров важности. Кроме того, использование подклассов позволяет повысить эффективность загрузки. Большие операции преобразования можно назначить более крупным классам ресурсов. Более высокий уровень важности можно использовать для обеспечения того, чтобы основные данные о продажах загружались перед данными о погоде или веб-каналом данных социального характера.

Однако не все инструкции классифицируются, поскольку для выполнения им не требуются ресурсы или указание уровня важности.  Команды консоли базы данных (DBCC), инструкции BEGIN, COMMIT и ROLLBACK TRANSACTION не классифицируются.

## <a name="classification-process"></a>Процесс классификации

Классификация для выделенного пула SQL достигается сегодня путем назначения пользователям роли, которой назначен соответствующий класс ресурсов, с помощью [sp_addrolemember](/sql/relational-databases/system-stored-procedures/sp-addrolemember-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true). Эта возможность позволяет определять запросы, которые выходят за пределы имени входа в класс ресурсов. Более мощный метод классификации теперь доступен с синтаксисом [классификатора рабочей нагрузки](/sql/t-sql/statements/create-workload-classifier-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) .  С помощью этого синтаксиса выделенные пользователи пула SQL могут назначать важность и количество системных ресурсов, назначенных запросу через `workload_group` параметр.

> [!NOTE]
> Классификация оценивается на основе каждого запроса. Несколько запросов в одном сеансе можно классифицировать по-разному.

## <a name="classification-weighting"></a>Весовые коэффициенты классификации

В рамках процесса классификации весовые коэффициенты определяют, какая группа рабочей нагрузки назначена.  Весовой коэффициент выглядит следующим образом.

|Параметр классификатора |Вес   |
|---------------------|---------|
|MEMBERNAME: ПОЛЬЗОВАТЕЛЬ      |64       |
|MEMBERNAME: ROLE      |32       |
|WLM_LABEL            |16       |
|WLM_CONTEXT          |8        |
|START_TIME/END_TIME  |4        |

`membername`Параметр является обязательным.  Однако, если указанное MemberName является пользователем базы данных, а не ролью базы данных, весовой коэффициент для пользователя больше, и поэтому выбран классификатор.

Если пользователь является членом нескольких ролей, которым назначены разные классы ресурсов или сопоставлены несколько классификаторов, пользователю назначается класс ресурсов наиболее высокого уровня.  Это поведение согласуется с существующим поведением при назначении классов ресурсов.

## <a name="system-classifiers"></a>Классификаторы систем

Классификация рабочей нагрузки содержит классификаторы системных нагрузок. Классификаторы систем сопоставляют существующие членства ролей класса ресурсов с выделением ресурсов класса ресурсов с обычной важностью. Не удается удалить классификаторы системы. Для просмотра классификаторов системы можно выполнить следующий запрос:

```sql
SELECT * FROM sys.workload_management_workload_classifiers where classifier_id <= 12
```

## <a name="mixing-resource-class-assignments-with-classifiers"></a>Смешивание назначений классов ресурсов с помощью классификаторов

Классификаторы систем, созданные от вашего имени, предоставляют простой путь для миграции на классификацию рабочих нагрузок. Использование сопоставления ролей класса ресурсов с приоритетом классификации может привести к неправильной классификации при начале создания новых классификаторов с важностью.

Рассмотрим следующий сценарий.

- Имеющееся хранилище данных имеет пользователя базы данных Дбаусер, назначенного роли класса ресурсов largerc. Назначение класса ресурсов было выполнено с sp_addrolemember.
- Теперь хранилище данных обновляется с помощью управления рабочей нагрузкой.
- Для проверки нового синтаксиса классификации роль базы данных Дбароле (которая является членом Дбаусер) имеет классификатор, созданный для их сопоставления с mediumrc и высокой важностью.
- Когда Дбаусер входит в систему и выполняет запрос, запрос будет назначен largerc. Так как пользователь имеет приоритет над членством в роли.

Чтобы упростить устранение неполадок классификации, рекомендуется удалить сопоставления ролей класса ресурсов при создании классификаторов рабочей нагрузки.  Приведенный ниже код возвращает членство в роли существующего класса ресурсов.  Запустите [sp_droprolemember](/sql/relational-databases/system-stored-procedures/sp-droprolemember-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) для каждого имени члена, возвращенного из соответствующего класса ресурсов.

```sql
SELECT  r.name AS [Resource Class]
,       m.name AS membername
FROM    sys.database_role_members rm
JOIN    sys.database_principals AS r ON rm.role_principal_id = r.principal_id
JOIN    sys.database_principals AS m ON rm.member_principal_id = m.principal_id
WHERE   r.name IN ('mediumrc','largerc','xlargerc','staticrc10','staticrc20','staticrc30','staticrc40','staticrc50','staticrc60','staticrc70','staticrc80');

--for each row returned run
sp_droprolemember '[Resource Class]', membername
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о создании классификатора см. в разделе [Создание классификатора рабочей нагрузки (Transact-SQL)](/sql/t-sql/statements/create-workload-classifier-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).  
- См. Краткое руководство по созданию классификатора рабочей нагрузки [Создание классификатора рабочей нагрузки](quickstart-create-a-workload-classifier-tsql.md).
- См. статьи с инструкциями по [настройке важности рабочей нагрузки](sql-data-warehouse-how-to-configure-workload-importance.md) , а также об [управлении рабочими нагрузками и наблюдении за](sql-data-warehouse-how-to-manage-and-monitor-workload-importance.md)ними.
- Запросы и назначенную важность см. в разделе [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).
