---
title: Загрузка данных в среды службы хранилища Azure. процесс обработки и анализа данных группы
description: Сведения о приеме данных в различных целевых средах, где хранятся и обрабатываются данные.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 46e911ad6bdf5a478d46e425f8700740ece56c6e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96452668"
---
# <a name="load-data-into-storage-environments-for-analytics"></a>Загрузка данных в среды хранения для аналитики

Процесс обработки и анализа данных группы требует, чтобы данные были приняты или загружены наиболее подходящим образом на каждом этапе. Назначения данных могут включать хранилище BLOB-объектов Azure, базы данных SQL Azure, SQL Server на виртуальной машине Azure, HDInsight (Hadoop), Azure синапсе Analytics и Машинное обучение Azure. 

В следующих статьях описывается, как выполняется прием данных в различных целевых средах, где данные могут храниться и обрабатываться:

* в [хранилище BLOB-объектов Azure](move-azure-blob.md) или из него;
* в [SQL Server на виртуальной машине Azure](move-sql-server-virtual-machine.md);
* В [базу данных SQL Azure](move-sql-azure.md)
* в [таблице Hive](move-hive-tables.md);
* в [секционированных таблицах SQL](parallel-load-sql-partitioned-tables.md);
* из [локальной базы данных SQL Server](move-sql-azure-adf.md).

Лучшие планы приема данных определяют технические и бизнес-потребности, а также начальное расположение, формат и размер данных. Нередко оптимальным планом является наличие нескольких шагов. Эта последовательность задач может включать в себя, например, просмотр данных, предварительную обработку, очистку, понижение частоты выборки и обучение модели.  Фабрика данных Azure — это рекомендуемый ресурс Azure для управления перемещением и преобразованием данных.
