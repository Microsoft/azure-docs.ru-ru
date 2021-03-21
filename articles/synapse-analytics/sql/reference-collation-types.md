---
title: Поддержка параметров сортировки
description: Поддержка типов параметров сортировки для синапсе SQL в Azure синапсе Analytics
author: filippopovic
ms.service: synapse-analytics
ms.topic: reference
ms.subservice: sql
ms.date: 04/15/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.openlocfilehash: 436dbac814197556385a33d956928f97fd4716bf
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93311914"
---
# <a name="database-collation-support-for-synapse-sql-in-azure-synapse-analytics"></a>Поддержка параметров сортировки базы данных для синапсе SQL в Azure синапсе Analytics 

Параметры сортировки обеспечивают языковой стандарт, кодовую страницу, порядок сортировки и правила учета чувствительности к символам для символьных типов данных. После выбора все столбцы и выражения, для которых требуются сведения о параметрах сортировки, наследуют выбранные параметры сортировки из параметра базы данных. Наследование по умолчанию может быть переопределено явным образом, указывая другие параметры сортировки для символьного типа данных.

Параметры сортировки базы данных по умолчанию можно изменить из портал Azure при создании новой выделенной базы данных пула SQL. Эта возможность упрощает создание новой базы данных, используя один из поддерживаемых параметров сортировки базы данных 3800.

Можно указать параметры сортировки базы данных SQL Server без сервера по умолчанию во время создания с помощью инструкции CREATE DATABASE.

## <a name="change-collation"></a>Изменить параметры сортировки
Чтобы изменить параметры сортировки по умолчанию для выделенной базы данных пула SQL, обновите поле Параметры сортировки в процессе подготовки. Например, если вы хотите изменить параметры сортировки по умолчанию на с учетом регистра, измените параметры сортировки с SQL_Latin1_General_CP1_CI_AS на SQL_Latin1_General_CP1_CS_AS. 

Чтобы изменить параметры сортировки по умолчанию для базы данных пула SQL без сервера, можно использовать инструкцию ALTER DATABASE.

## <a name="list-of-unsupported-collation-types"></a>Список неподдерживаемых типов параметров сортировки
*    Japanese_Bushu_Kakusu_140_BIN
*    Japanese_Bushu_Kakusu_140_BIN2
*    Japanese_Bushu_Kakusu_140_CI_AI_VSS
*    Japanese_Bushu_Kakusu_140_CI_AI_WS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AI_KS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AI_KS_WS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AS_WS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AS_KS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AS_KS_WS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AI_VSS
*    Japanese_Bushu_Kakusu_140_CS_AI_WS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AI_KS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AI_KS_WS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AS_WS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AS_KS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AS_KS_WS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AI
*    Japanese_Bushu_Kakusu_140_CI_AI_WS
*    Japanese_Bushu_Kakusu_140_CI_AI_KS
*    Japanese_Bushu_Kakusu_140_CI_AI_KS_WS
*    Japanese_Bushu_Kakusu_140_CI_AS
*    Japanese_Bushu_Kakusu_140_CI_AS_WS
*    Japanese_Bushu_Kakusu_140_CI_AS_KS
*    Japanese_Bushu_Kakusu_140_CI_AS_KS_WS
*    Japanese_Bushu_Kakusu_140_CS_AI
*    Japanese_Bushu_Kakusu_140_CS_AI_WS
*    Japanese_Bushu_Kakusu_140_CS_AI_KS
*    Japanese_Bushu_Kakusu_140_CS_AI_KS_WS
*    Japanese_Bushu_Kakusu_140_CS_AS
*    Japanese_Bushu_Kakusu_140_CS_AS_WS
*    Japanese_Bushu_Kakusu_140_CS_AS_KS
*    Japanese_Bushu_Kakusu_140_CS_AS_KS_WS
*    Japanese_XJIS_140_BIN
*    Japanese_XJIS_140_BIN2
*    Japanese_XJIS_140_CI_AI_VSS
*    Japanese_XJIS_140_CI_AI_WS_VSS
*    Japanese_XJIS_140_CI_AI_KS_VSS
*    Japanese_XJIS_140_CI_AI_KS_WS_VSS
*    Japanese_XJIS_140_CI_AS_VSS
*    Japanese_XJIS_140_CI_AS_WS_VSS
*    Japanese_XJIS_140_CI_AS_KS_VSS
*    Japanese_XJIS_140_CI_AS_KS_WS_VSS
*    Japanese_XJIS_140_CS_AI_VSS
*    Japanese_XJIS_140_CS_AI_WS_VSS
*    Japanese_XJIS_140_CS_AI_KS_VSS
*    Japanese_XJIS_140_CS_AI_KS_WS_VSS
*    Japanese_XJIS_140_CS_AS_VSS
*    Japanese_XJIS_140_CS_AS_WS_VSS
*    Japanese_XJIS_140_CS_AS_KS_VSS
*    Japanese_XJIS_140_CS_AS_KS_WS_VSS
*    Japanese_XJIS_140_CI_AI
*    Japanese_XJIS_140_CI_AI_WS
*    Japanese_XJIS_140_CI_AI_KS
*    Japanese_XJIS_140_CI_AI_KS_WS
*    Japanese_XJIS_140_CI_AS
*    Japanese_XJIS_140_CI_AS_WS
*    Japanese_XJIS_140_CI_AS_KS
*    Japanese_XJIS_140_CI_AS_KS_WS
*    Japanese_XJIS_140_CS_AI
*    Japanese_XJIS_140_CS_AI_WS
*    Japanese_XJIS_140_CS_AI_KS
*    Japanese_XJIS_140_CS_AI_KS_WS
*    Japanese_XJIS_140_CS_AS
*    Japanese_XJIS_140_CS_AS_WS
*    Japanese_XJIS_140_CS_AS_KS
*    Japanese_XJIS_140_CS_AS_KS_WS

Кроме того, выделенный пул SQL не поддерживает следующие типы параметров сортировки:

*    SQL_EBCDIC1141_CP1_CS_AS
*    SQL_EBCDIC277_2_CP1_CS_AS
*    UTF-8

## <a name="check-the-current-collation"></a>Проверить текущие параметры сортировки
Чтобы проверить текущие параметры сортировки для базы данных, можно выполнить следующий фрагмент кода T-SQL:
```sql
SELECT DATABASEPROPERTYEX(DB_NAME(), 'Collation') AS Collation;
```
Если в качестве параметра свойства передан параметр "collation", функция DatabasePropertyEx возвращает текущие параметры сортировки для указанной базы данных. Дополнительные сведения о функции DatabasePropertyEx см. на сайте MSDN.

## <a name="next-steps"></a>Дальнейшие действия

Рекомендации по работе с выделенным и бессерверным пулами SQL см. в следующих статьях:

- [Рекомендации по использованию выделенного пула SQL](best-practices-sql-pool.md)
- [Рекомендации по использованию бессерверного пула SQL](best-practices-sql-on-demand.md)


