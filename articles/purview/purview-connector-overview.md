---
title: Общие сведения о соединителе зрения
description: В этой статье описаны различные хранилища данных и возможности, поддерживаемые в зрения
author: chandrakavya
ms.author: kchandra
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 11/13/2020
ms.openlocfilehash: 08b22af8743082bab1d547205e51917cb9d92a11
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101695776"
---
# <a name="supported-data-stores"></a>Поддерживаемые хранилища данных

Зрения поддерживает следующие хранилища данных. Щелкните каждое хранилище данных, чтобы узнать о поддерживаемых возможностях и соответствующих конфигурациях.

## <a name="purview-data-sources"></a>Источники данных зрения

|**Категория**|  **Хранилище данных**  |**Извлечение метаданных**|**Полная проверка**|**Добавочное сканирование**|**Просмотр с заданной областью**|**Классификация**|**Журнал преобразований**|
|---|---|---|---|---|---|---|---|
| Azure | [Хранилище BLOB-объектов Azure](register-scan-azure-blob-storage-source.md)| Да| Да| Да| Да| Да| Да|
||[Azure Cosmos DB](register-scan-azure-cosmos-database.md)|Да| Да| Да| Да| Да| Да|
||[Azure Data Explorer](register-scan-azure-data-explorer.md)|Да| Да| Да| Да| Да| Да|
||[Azure Data Lake Storage 1-го поколения](register-scan-adls-gen1.md)|Да| Да| Да| Да| Да| Да|
||[Azure Data Lake Storage 2-го поколения](register-scan-adls-gen2.md)|Да| Да| Да| Да| Да| Да|
||[База данных SQL Azure](register-scan-azure-sql-database.md)|Да| Да| Нет| Да| Да| Да|
||[Управляемый экземпляр Базы данных SQL Azure](register-scan-azure-sql-database-managed-instance.md)|Да| Да| Нет| Да| Да| Да|
||[Azure Synapse Analytics (ранее — Хранилище данных SQL)](register-scan-azure-synapse-analytics.md)|Да| Да| Нет| Да| Да| Да|
|База данных|[Oracle DB](register-scan-oracle-source.md)|Да| Да| Нет| Нет| Нет| Да|
||[SQL Server](register-scan-on-premises-sql-server.md)|Да| Да| Нет| Да| Да| Да|
||[Teradata](register-scan-teradata-source.md)|Да| Да| Нет| Нет| Нет| Да|
|Power BI|[Power BI](register-scan-power-bi-tenant.md)|Да| Да| Нет| Нет| Нет| Да|
|Службы и приложения|[SAP ECC](register-scan-sapecc-source.md)|Да| Да| Нет| Да| Да| Да|
||[S4HANA SAP](register-scan-saps4hana-source.md)|Да| Да| Нет| Да| Да| Да|

## <a name="next-steps"></a>Дальнейшие действия

- [Регистрация и проверка источника хранилища BLOB-объектов Azure](register-scan-azure-blob-storage-source.md)