---
title: Службы машинного обучения в управляемом экземпляре SQL Azure
description: В этой статье содержится обзор или Службы машинного обучения в Azure SQL Управляемый экземпляр.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: machine-learning
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: garyericson
ms.author: garye
ms.reviewer: sstein, davidph
manager: cgronlun
ms.date: 03/17/2021
ms.openlocfilehash: 94495144c64b3770995a5f67e9129b3ba86e741e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104599567"
---
# <a name="machine-learning-services-in-azure-sql-managed-instance"></a>Службы машинного обучения в управляемом экземпляре SQL Azure

Службы машинного обучения — это функция Управляемый экземпляр SQL Azure, которая обеспечивает машинное обучение в базе данных, поддерживающее скрипты Python и R. Эта функция включает в себя пакеты Microsoft Python и R для высокопроизводительной прогнозной аналитики и машинного обучения. Реляционные данные можно использовать в скриптах с помощью хранимых процедур, скрипта T-SQL, содержащего инструкции Python или R, или кода Python или R, содержащего T-SQL.

## <a name="what-is-machine-learning-services"></a>Что такое Службы машинного обучения?

Службы машинного обучения в Azure SQL Управляемый экземпляр позволяет выполнять скрипты Python и R в базе данных. С их помощью можно подготавливать и очищать данные, выполнять проектирование признаков, а также обучать, оценивать и развертывать модели машинного обучения в базе данных. Этот компонент выполняет скрипты там, где хранятся данные, и устраняет необходимость перемещения данных по сети на другой сервер.

Используйте Службы машинного обучения с поддержкой R/Python в Управляемый экземпляр SQL Azure:

- **Выполнение скриптов r и Python для подготовки данных и обработки данных общего назначения** . Теперь вы можете перенести сценарии r/Python в Azure SQL управляемый экземпляр, где хранятся ваши данные, вместо того чтобы перемещать данные на другой сервер для выполнения скриптов R и Python. Можно исключить необходимость в перемещении данных и связанных с ними проблемах, связанных с задержкой, безопасностью и соответствием.

- **Обучение моделей машинного обучения в базе данных** . Вы можете обучить модели с помощью любых алгоритмов с открытым исходным кодом. Вы можете легко масштабировать обучение по всему набору данных, не полагаясь на образцы наборов данных, извлеченных из базы.

- **Развертывание моделей и скриптов в рабочей среде в хранимых процедурах** . скрипты и обученные модели могут работать просто путем их внедрения в хранимые процедуры T-SQL. Приложения, подключающиеся к Azure SQL Управляемый экземпляр могут воспользоваться преимуществами прогнозов и аналитики в этих моделях, просто вызвав хранимую процедуру. Можно также использовать собственную функцию ПРОГНОЗИРОВАНИя T-SQL, чтобы эксплуатацию модели для быстрой оценки в сценариях оценки с высокой степенью актуальности в реальном времени.

Базовые распределения Python и R включены в Службы машинного обучения. Вы можете установить и использовать платформы и пакеты с открытым исходным кодом, такие как PyTorch, TensorFlow и scikit-learn, в дополнение к пакетам Microsoft [revoscalepy](/sql/machine-learning/python/ref-py-revoscalepy) и [microsoftml](/sql/machine-learning/python/ref-py-microsoftml) для Python и [RevoScaleR](/sql/machine-learning/r/ref-r-revoscaler), [MicrosoftML](/sql/machine-learning/r/ref-r-microsoftml), [OLAP](/sql/machine-learning/r/ref-r-olapr) и [sqlrutils](/sql/machine-learning/r/ref-r-sqlrutils) для R.

## <a name="how-to-enable-machine-learning-services"></a>Включение Службы машинного обучения

Вы можете включить Службы машинного обучения в Управляемый экземпляр SQL Azure, включив расширяемость с помощью следующих команд SQL (SQL Управляемый экземпляр будет перезапущен и будет недоступен в течение нескольких секунд):

```sql
sp_configure 'external scripts enabled', 1;
RECONFIGURE WITH OVERRIDE;
```

Дополнительные сведения о влиянии этой команды на ресурсы SQL Управляемый экземпляр см. в разделе [Управление ресурсами](machine-learning-services-differences.md#resource-governance).

### <a name="enable-machine-learning-services-in-a-failover-group"></a>Включение Службы машинного обучения в группе отработки отказа

В [группе отработки отказа](failover-group-add-instance-tutorial.md)системные базы данных не реплицируются на дополнительный экземпляр (Дополнительные сведения см. в разделе [ограничения групп отработки отказа](../database/auto-failover-group-overview.md#limitations-of-failover-groups) ).

Если Управляемый экземпляр, который вы используете, является частью группы отработки отказа, выполните следующие действия.

- Выполните `sp_configure` команды и `RECONFIGURE` на каждом экземпляре группы отработки отказа, чтобы включить службы машинного обучения.

- Установите библиотеки R/Python в пользовательской базе данных, а не в базе данных master.

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с [основными отличиями от SQL Server службы машинного обучения](machine-learning-services-differences.md).
- Сведения об использовании Python в Службы машинного обучения см. в разделе [выполнение скриптов Python](/sql/machine-learning/tutorials/quickstart-python-create-script?context=/azure/azure-sql/managed-instance/context/ml-context&view=azuresqldb-mi-current&preserve-view=true).
- Сведения об использовании R в Службы машинного обучения см. в разделе [выполнение скриптов r](/sql/machine-learning/tutorials/quickstart-r-create-script?context=/azure/azure-sql/managed-instance/context/ml-context&view=azuresqldb-mi-current&preserve-view=true).
- Дополнительные сведения о машинном обучении на других платформах SQL см. в [документации по машинному обучению SQL](/sql/machine-learning/index).
