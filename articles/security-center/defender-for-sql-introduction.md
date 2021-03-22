---
title: Преимущества и функции Azure Defender для SQL
description: Сведения о преимуществах и функциях Azure Defender для SQL.
author: memildin
ms.author: memildin
ms.date: 12/13/2020
ms.topic: overview
ms.service: security-center
ms.custom: references_regions
manager: rkarlin
ms.openlocfilehash: 532c46c50d0b422946af649801e43904b4b6ed7d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102096876"
---
# <a name="introduction-to-azure-defender-for-sql"></a>Общие сведения об Azure Defender для SQL

Azure Defender для SQL содержит два плана Azure Defender, которые расширяют [пакет безопасности данных](../azure-sql/database/azure-defender-for-sql.md) Центра безопасности Azure и позволяют защитить сами базы данных и размешенные в них данные в любом расположении. 

> [!VIDEO https://www.youtube.com/embed/V7RdB6RSVpc]

## <a name="availability"></a>Доступность

|Аспект|Сведения|
|----|:----|
|Состояние выпуска:|**Azure Defender для серверов баз данных SQL Azure** — общедоступная версия<br>**Azure Defender для серверов SQL на компьютерах**. Общедоступная версия (GA) |
|Цены|Для двух планов, которые входят в предложение **Azure Defender для SQL**, плата взимается по тарифам, приведенным на странице [Цены на Центр безопасности](https://azure.microsoft.com/pricing/details/security-center/).|
|Защищаемые версии SQL|[SQL на виртуальных машинах Azure](../azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md)<br>[Серверы SQL с поддержкой Azure Arc](/sql/sql-server/azure-arc/overview)<br>Локальные серверы SQL на компьютерах Windows без поддержки Azure Arc<br>[Отдельные базы данных](../azure-sql/database/single-database-overview.md) и [эластичные пулы](../azure-sql/database/elastic-pool-overview.md) Azure SQL<br>[Управляемый экземпляр SQL Azure](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md)<br>[Выделенный пул SQL в Azure Synapse Analytics (ранее — Хранилище данных SQL)](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md)|
|Облако.|![Да](./media/icons/yes-icon.png) Коммерческие облака<br>![Да](./media/icons/yes-icon.png) US Gov<br>![Yes](./media/icons/yes-icon.png) China Gov (**частично**: набор оповещений и оценка уязвимости для серверов SQL Server; защита от угроз поведения недоступна)|
|||

## <a name="what-does-azure-defender-for-sql-protect"></a>На что распространяется защита Azure Defender для SQL?

**Azure Defender для SQL** состоит из двух отдельных планов Azure Defender.

- **Azure Defender для серверов баз данных SQL Azure** защищает следующие компоненты:
    - [База данных SQL Azure](../azure-sql/database/sql-database-paas-overview.md)
    - [Управляемый экземпляр SQL Azure](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md)
    - [Выделенный пул SQL в Azure Synapse](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md)

- **Azure Defender для серверов SQL на компьютерах** расширяет возможности защиты для собственных серверов SQL Azure, предоставляя полную поддержку гибридных сред, то есть обеспечивает защиту серверов SQL (все поддерживаемые версии) не только в Azure, но и в других облачных средах и даже на локальных компьютерах.
    - [SQL Server на виртуальных машинах](https://azure.microsoft.com/services/virtual-machines/sql-server/)
    - Локальные серверы SQL.
        - [SQL Server с поддержкой Azure Arc (предварительная версия)](/sql/sql-server/azure-arc/overview)
        - [SQL Server, работающий на компьютерах Windows, без поддержки Azure Arc](../azure-monitor/agents/agent-windows.md)


## <a name="what-are-the-benefits-of-azure-defender-for-sql"></a>Каковы преимущества Azure Defender для SQL?

Два предлагаемых плана включают возможности обнаружения и устранения потенциальных уязвимостей базы данных и обнаружения аномальных действий, которые могут указывать на угрозу для вашей базы данных.

- [Оценка уязвимостей](../azure-sql/database/sql-vulnerability-assessment.md) — служба проверки, которая поможет обнаруживать, отслеживать и устранять потенциальные уязвимости базы данных. Оценочная проверка предоставит общие сведения о состоянии безопасности для компьютеров SQL и подробные сведения о результатах проверки безопасности.

- [Расширенная защита от угроз](../azure-sql/database/threat-detection-overview.md) — служба обнаружения, которая постоянно проверяет наличие на серверах SQL таких угроз, как внедрение кода SQL, атаки методом перебора и злоупотребление привилегиями. Эта служба передает в Центр безопасности Azure оповещения безопасности, которые содержат описания подозрительных действий, рекомендации по устранению этих угроз и варианты для продолжения исследований с помощью Azure Sentinel. 
    > [!TIP]
    > Просмотрите список предупреждений системы безопасности для серверов SQL Server на [странице справки по оповещениям](alerts-reference.md#alerts-sql-db-and-warehouse).


## <a name="what-kind-of-alerts-does-azure-defender-for-sql-provide"></a>Какие типы оповещений предоставляет Azure Defender для SQL?

Оповещения системы безопасности, усовершенствованные анализом угроз, активируются, если существуют:

- **Потенциальные атаки путем внедрения кода SQL**, включая обнаружения уязвимостей, связанные с созданием приложениями ошибочных инструкций SQL в базе данных.
- **Аномальные шаблоны доступа и запросов к базе данных**, например необычно высокая доля неудачных попыток входа с разными учетными данными (как при атаках методом перебора).
- **Подозрительная активность базы данных**, например, допустимый пользователь обращается к SQL Server с поврежденного компьютера, который обменивается данными с сервером контроля и управления для добычи криптовалюты

Оповещения содержат сведения об инциденте, который привел к их активации, а также рекомендации по определению причины угроз и их устранению.



## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали об Azure Defender для SQL. Чтобы использовать описанные службы, выполните следующие действия:

- Используйте Azure Defender для серверов SQL на компьютерах для [проверки SQL серверов на наличие уязвимостей](defender-for-sql-usage.md).