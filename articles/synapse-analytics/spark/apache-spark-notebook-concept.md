---
title: Обзор записных книжек Azure синапсе Analytics
description: В этой статье представлен обзор возможностей, доступных в записных книжках Azure синапсе Analytics.
services: synapse-analytics
author: midesa
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 11/18/2020
ms.author: midesa
ms.reviewer: jrasnick
ms.subservice: spark
ms.openlocfilehash: 5efdfcea7b6d727038bad124368179b9395bfafd
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97093283"
---
# <a name="azure-synapse-analytics-notebooks"></a>Записные книжки Azure синапсе Analytics

Записная книжка синапсе Studio — это веб-интерфейс для создания файлов, содержащих код в реальном времени, визуализации и текст описания. Записные книжки отлично подходят для проверки идей и использования быстрых экспериментов, чтобы получить аналитические сведения по данным. 

С помощью записной книжки Azure Synapse Studio можно выполнять следующие задачи.

* Начните работу, потратив минимум усилий на настройку.
* Обеспечьте безопасность данных благодаря встроенным функциям безопасности предприятия.
* Анализируйте данные в необработанных форматах (CSV, txt, JSON и т. д.), обработанных форматах файлов (parquet, Delta Lake, ORC и т. д.) и файлах табличных данных SQL с использованием Spark и SQL.
* Повысьте продуктивность труда с расширенными возможностями разработки и встроенной визуализацией данных.

В этом разделе содержатся статьи о смешении языков, создании визуализаций данных, параметризации записных книжек, построении конвейеров и многом другое. Здесь также содержатся ссылки и учебники, в которых можно приступить к разработке записной книжки.

## <a name="create-manage-and-use-notebooks"></a>Создание, управление и использование записных книжек
Вы можете управлять записными книжками с помощью пользовательского интерфейса синапсе Studio. 

Дополнительные сведения о создании записных книжек и управлении ими см. в следующих статьях:
  - Управление записными книжками
    - [Создание записных книжек](./spark/../apache-spark-development-using-notebooks.md#create-a-notebook)
    - [Разработка записных книжек](./spark/../apache-spark-development-using-notebooks.md#develop-notebooks)
    - [Перенесение данных в записную книжку](./spark/../apache-spark-development-using-notebooks.md#bring-data-to-a-notebook)
    - [Использование нескольких языков с помощью команд Magic и временных таблиц](./spark/../apache-spark-development-using-notebooks.md#integrate-a-notebook)
    - [Использование команд Magic Cell](./spark/../apache-spark-development-using-notebooks.md#magic-commands)
  - Разработка
    - [Настройка параметров сеанса Spark](./spark/../apache-spark-development-using-notebooks.md#spark-session-config)
    - [Использование служебных программ Microsoft Spark](./spark/../microsoft-spark-utilities.md)
    - [Визуализация данных с помощью записных книжек и библиотек](./spark/../apache-spark-data-visualization.md)
    - [Интеграция записной книжки в конвейеры](./spark/../apache-spark-development-using-notebooks.md#integrate-a-notebook)


## <a name="next-steps"></a>Дальнейшие действия
Записные книжки также широко используются при подготовке данных, визуализации данных, машинном обучении и других сценариях обработки больших данных. Дополнительные сведения о том, как можно использовать записные книжки для анализа данных и сценариев обработки больших данных, см. в следующих руководствах:
  - [Создание записной книжки](./spark/../../quickstart-apache-spark-notebook.md)
  - [Создание визуализаций с помощью записных книжек синапсе Studio](./spark/../apache-spark-data-visualization-tutorial.md)
  - [Создание моделей машинного обучения с помощью Apache Spark MLlib](./spark/../apache-spark-machine-learning-mllib-notebook.md)
  - [Создание моделей машинного обучения с помощью автоматического машинного обучения Azure](./spark/../apache-spark-azure-machine-learning-tutorial.md)