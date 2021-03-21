---
title: Сравнение с Фабрикой данных Azure
description: Узнайте, как возможности интеграции данных Azure синапсе Analytics отличаются от функций фабрики данных Azure.
services: synapse-analytics
author: kromerm
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 12/10/2020
ms.author: makromer
ms.reviewer: jrasnick
ms.openlocfilehash: 9cd3fc9353c684ec6617761616b958477ca83ee3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102183647"
---
# <a name="data-integration-in-azure-synapse-analytics-versus-azure-data-factory"></a>Интеграция данных в Azure синапсе Analytics и фабрика данных Azure

В Azure синапсе Analytics возможности интеграции данных, такие как конвейеры синапсе и потоки данных, основаны на этих возможностях фабрики данных Azure. Дополнительные сведения см. в статье [что такое фабрика данных Azure](../../data-factory/introduction.md).


## <a name="available-features-in-adf--azure-synapse-analytics"></a>Доступные функции в ADF & Azure синапсе Analytics

Ниже приведена таблица со сведениями о доступности функций.

| Category                 | Компонент    |  Фабрика данных Azure  | Azure Synapse Analytics |
| ------------------------ | ---------- | :------------------: | :---------------------: |
| **Integration Runtime**  | Использование служб SSIS и служб SSIS Integration Runtime | ✓ | ✗ |
|                          | Поддержка Integration Runtime в разных регионах (потоки данных) | ✓ | ✗ |
|                          | Integration Runtime общий доступ | ✓<br><small>*Можно совместно использовать в разных фабриках данных* | ✗ |
|                          | Срок жизни | ✓ | ✗ |
| **Действия конвейеров** | Действие пакета служб SSIS | ✓ | ✗ |
|                          | Поддержка действий Power Query | ✓ | ✓ |
| **Коллекция шаблонов и центр знаний** | Шаблоны решений | ✓<br><small>*Коллекция шаблонов фабрики данных Azure* | ✓<br><small>*Центр знаний рабочей области синапсе* |
| **Интеграция репозитория GIT** | Интеграция с GIT | ✓ | ✓ |
| **Мониторинг**           | Мониторинг заданий Spark для потока данных | ✗ | ✓<br><small>*Использование пулов синапсе Spark* |
|                          | Интеграция с Azure Monitor | ✓ | ✗ |
| **Журнал преобразований** | Поддерживает публикацию данных журнала преобразований конвейера в зрения  | ✓ | ✗ |  

> [!Note]
> **Срок жизни** — это параметр Azure Integration Runtime, который позволяет кластеру Spark *оставаться тепло* в течение определенного периода времени после выполнения потока данных.
>


## <a name="next-steps"></a>Дальнейшие действия

Приступите к работе с интеграцией данных в рабочей области синапсе, изучите прием [данных в учетную запись gen2 Azure Data Lake Storage](data-integration-data-lake.md).
