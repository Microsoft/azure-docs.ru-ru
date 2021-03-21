---
title: Матрица поддержки для центра архивации
description: В этой статье перечислены сценарии, поддерживаемые центром архивации для каждого типа рабочей нагрузки.
ms.topic: conceptual
ms.date: 09/07/2020
ms.openlocfilehash: d6e5d34e201edda4fd1e9fda85f210fb88211e28
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102504513"
---
# <a name="support-matrix-for-backup-center"></a>Матрица поддержки для центра архивации

Центр архивации предоставляет единую панель для предприятий, которая позволяет [управлять, отслеживать и анализировать резервные копии в нужном масштабе](backup-center-overview.md). В этой статье перечислены сценарии, поддерживаемые центром архивации для каждого типа рабочей нагрузки.

## <a name="supported-scenarios"></a>Поддерживаемые сценарии

| **Категория** | **Сценарий**  | **Поддерживаемые рабочие нагрузки**  | **Ограничения** |
| -------------| ------------- | ----------------------- |------------|
| Мониторинг   | Просмотреть все задания | <li> Виртуальная машина Azure <br><br> <li> сервер базы данных Azure для PostgreSQL. <br><br> <li> SQL на виртуальной машине Azure <br><br> <li> SAP HANA на виртуальной машине Azure <br><br> <li> Файлы Azure<br/><br/> <li>Большие двоичные объекты Azure<br/><br/> <li>управляемые диски Azure. | <li> в течение 7 дней все задания доступны. <br> <li> Каждый фильтр/раскрывающийся список поддерживает не более 1000 элементов. Поэтому Центр архивации можно использовать для мониторинга максимум 1000 подписок и 1000 хранилищ между клиентами. |
| Мониторинг | Просмотреть все экземпляры резервного копирования | <li> Виртуальная машина Azure <br><br> <li> сервер базы данных Azure для PostgreSQL. <br><br> <li> SQL на виртуальной машине Azure <br><br> <li> SAP HANA на виртуальной машине Azure <br><br> <li> Файлы Azure<br/><br/> <li>Большие двоичные объекты Azure<br/><br/> <li>управляемые диски Azure. | То же, что и выше |
| Мониторинг | Просмотреть все политики архивации | <li> Виртуальная машина Azure <br><br> <li> сервер базы данных Azure для PostgreSQL. <br><br> <li> SQL на виртуальной машине Azure <br><br> <li> SAP HANA на виртуальной машине Azure <br><br> <li> Файлы Azure<br/><br/> <li>Большие двоичные объекты Azure<br/><br/> <li>управляемые диски Azure. | То же, что и выше |
| Мониторинг | Просмотреть все хранилища | <li> Виртуальная машина Azure <br><br> <li> сервер базы данных Azure для PostgreSQL. <br><br> <li> SQL на виртуальной машине Azure <br><br> <li> SAP HANA на виртуальной машине Azure <br><br> <li> Файлы Azure<br/><br/> <li>Большие двоичные объекты Azure<br/><br/> <li>управляемые диски Azure. | То же, что и выше |
| Действия | Настройка резервного копирования | <li> Виртуальная машина Azure <br><br> <li> сервер базы данных Azure для PostgreSQL. <br><br> <li> SQL на виртуальной машине Azure <br><br> <li> SAP HANA на виртуальной машине Azure <br><br> <li> Файлы Azure<br/><br/> <li>Большие двоичные объекты Azure<br/><br/> <li>управляемые диски Azure. | Дополнительные сведения см. в статье матрицы поддержки для [резервного копирования виртуальных машин Azure](./backup-support-matrix-iaas.md) и [базы данных Azure для PostgreSQL Server](backup-azure-database-postgresql.md#support-matrix) . |
| Действия | Восстановление экземпляра резервной копии | <li> Виртуальная машина Azure <br><br> <li> сервер базы данных Azure для PostgreSQL. <br><br> <li> SQL на виртуальной машине Azure <br><br> <li> SAP HANA на виртуальной машине Azure <br><br> <li> Файлы Azure<br/><br/> <li>Большие двоичные объекты Azure<br/><br/> <li>управляемые диски Azure. | Дополнительные сведения см. в статье матрицы поддержки для [резервного копирования виртуальных машин Azure](./backup-support-matrix-iaas.md) и [базы данных Azure для PostgreSQL Server](backup-azure-database-postgresql.md#support-matrix) . |
| Действия | Создать хранилище | <li> Виртуальная машина Azure <br><br> <li> сервер базы данных Azure для PostgreSQL. <br><br> <li> SQL на виртуальной машине Azure <br><br> <li> SAP HANA на виртуальной машине Azure <br><br> <li> Файлы Azure<br/><br/> <li>Большие двоичные объекты Azure<br/><br/> <li>управляемые диски Azure. | Дополнительные сведения см. в статье матрицы поддержки для [хранилища служб восстановления](./backup-support-matrix.md#vault-support) . |
| Действия | Создание политики архивации | <li> Виртуальная машина Azure <br><br> <li> сервер базы данных Azure для PostgreSQL. <br><br> <li> SQL на виртуальной машине Azure <br><br> <li> SAP HANA на виртуальной машине Azure <br><br> <li> Файлы Azure<br/><br/> <li>Большие двоичные объекты Azure<br/><br/> <li>управляемые диски Azure. | Дополнительные сведения см. в статье матрицы поддержки для [хранилища служб восстановления](./backup-support-matrix.md#vault-support) . |
| Действия | Выполнение резервного копирования по запросу для экземпляра резервной копии | <li> Виртуальная машина Azure <br><br> <li> сервер базы данных Azure для PostgreSQL. <br><br> <li> SQL на виртуальной машине Azure <br><br> <li> SAP HANA на виртуальной машине Azure <br><br> <li> Файлы Azure<br/><br/> <li>Большие двоичные объекты Azure<br/><br/> <li>управляемые диски Azure. | Дополнительные сведения см. в статье матрицы поддержки для [резервного копирования виртуальных машин Azure](./backup-support-matrix-iaas.md) и [базы данных Azure для PostgreSQL Server](backup-azure-database-postgresql.md#support-matrix) . |
| Действия | Завершение резервного копирования экземпляра резервной копии | <li> Виртуальная машина Azure <br><br> <li> сервер базы данных Azure для PostgreSQL. <br><br> <li> SQL на виртуальной машине Azure <br><br> <li> SAP HANA на виртуальной машине Azure <br><br> <li> Файлы Azure<br/><br/> <li>Большие двоичные объекты Azure<br/><br/> <li>управляемые диски Azure. | Дополнительные сведения см. в статье матрицы поддержки для [резервного копирования виртуальных машин Azure](./backup-support-matrix-iaas.md) и [базы данных Azure для PostgreSQL Server](backup-azure-database-postgresql.md#support-matrix) . |
| Аналитика | Просмотр отчетов по резервному копированию | <li> Виртуальная машина Azure <br><br> <li> SQL на виртуальной машине Azure <br><br> <li> SAP HANA на виртуальной машине Azure <br><br> <li> Файлы Azure <br><br> <li> System Center Data Protection Manager <br><br> <li> Агент Azure Backup (режим MARS) <br><br> <li> Azure Backup Server (MABS) | См. [Поддерживаемые сценарии резервного копирования отчетов](./configure-reports.md#supported-scenarios) |
| Система управления | Просмотр и назначение встроенных и пользовательских политик Azure в категории "Архивация" | Н/Д | Н/Д |
| Система управления | Просмотр источников данных, не настроенных для архивации | <li> Виртуальная машина Azure <br><br> <li> сервер базы данных Azure для PostgreSQL. | Н/Д |

## <a name="unsupported-scenarios"></a>Неподдерживаемые сценарии

| **Категория** | **Сценарий**  |
|--------------|---------------|
| Мониторинг | Просмотр предупреждений в масштабе |
| Действия | Настройка параметров хранилища в масштабе |
| Действия | Выполнение задания по восстановлению между регионами из центра архивации |

## <a name="next-steps"></a>Дальнейшие действия

* [Обзор матрицы поддержки для Azure Backup](./backup-support-matrix.md)
* [Обзор матрицы поддержки для резервного копирования виртуальных машин Azure](./backup-support-matrix-iaas.md)
* [Ознакомьтесь со сведениями в матрице поддержки для резервного копирования сервера базы данных Azure для PostgreSQL](backup-azure-database-postgresql.md#support-matrix)