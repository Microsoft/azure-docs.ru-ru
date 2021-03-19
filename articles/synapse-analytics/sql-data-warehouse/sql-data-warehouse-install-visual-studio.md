---
title: Установка Visual Studio 2019.
description: Установка средств разработки Visual Studio и SQL Server (SSDT) для синапсе SQL
services: synapse-analytics
ms.custom: vs-azure, azure-synapse
ms.workload: azure-vs
author: gaursa
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 05/11/2020
ms.author: gaursa
ms.reviewer: igorstan
ms.openlocfilehash: ab70ec4f7f2c313c2ae4efcc6d1ad994fbec8b03
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104585576"
---
# <a name="getting-started-with-visual-studio-2019"></a>Начало работы с Visual Studio 2019

Visual Studio **2019** SQL Server Data Tools (SSDT) — это единый инструмент, позволяющий выполнять следующие действия:

- Подключение, запрос и разработка приложений
- Используйте обозреватель объектов, чтобы визуально исследовать все объекты в модели данных, включая таблицы, представления, хранимые процедуры и т. д.
- Создание скриптов языка описания данных (DDL) T-SQL для объектов
- Разработка хранилища данных с использованием подхода на основе состояния с проектами баз данных SSDT
- Интегрируйте проект базы данных с системами управления версиями, такими как Git, с Azure Repos
- Настройка конвейеров непрерывной интеграции и развертывания с помощью серверов автоматизации, таких как Azure DevOps

## <a name="install-visual-studio-2019"></a>Установка Visual Studio 2019.

См. статью [скачивание Visual studio 2019](https://visualstudio.microsoft.com/downloads/) для загрузки и установки visual Studio **16,3 и более поздних версий**. Во время установки выберите рабочую нагрузку "хранение и обработка данных". Автономная установка SSDT больше не требуется в Visual Studio 2019.

## <a name="unsupported-features-in-ssdt"></a>Неподдерживаемые функции в SSDT

Бывают случаи, когда выпуски компонентов для синапсе SQL могут не включать поддержку SSDT. Следующие функции в настоящее время не поддерживаются:


- [Управление рабочей нагрузкой](sql-data-warehouse-workload-management.md) — группы рабочей нагрузки и классификаторы
- [Безопасность на уровне строк](/sql/relational-databases/security/row-level-security?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) (включая функции, возвращающие табличные значения)
  - Отправьте запрос в [службу поддержки или голосование](https://feedback.azure.com/forums/307516-sql-data-warehouse/suggestions/39040057-ssdt-row-level-security) , чтобы получить поддерживаемую функцию.
  - Отправьте запрос в [службу поддержки или голосование](https://feedback.azure.com/forums/307516-sql-data-warehouse/suggestions/39040048-ssdt-support-dynamic-data-masking) , чтобы получить поддерживаемую функцию.
- Таблицы со [столбцом идентификаторов](/sql/t-sql/statements/create-table-transact-sql-identity-property?view=azure-sqldw-latest&preserve-view=true)
- Некоторые функции T-SQL, такие как:
   - В предложении *Group* в функции [STRING_AGG](/sql/t-sql/functions/string-agg-transact-sql) строк.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда у вас установлена последняя версия SSDT, вы можете [Подключиться](sql-data-warehouse-query-visual-studio.md) к пулу SQL.