---
title: Основные отличия для Службы машинного обучения
description: В этой статье описываются основные различия между Службы машинного обучения в Управляемый экземпляр SQL Azure и SQL Server Службы машинного обучения.
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
ms.openlocfilehash: b5ad439a8e10fa9aa44e477ca35f45d65ae40803
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104599550"
---
# <a name="key-differences-between-machine-learning-services-in-azure-sql-managed-instance-and-sql-server"></a>Основные различия между Службами машинного обучения в Управляемом экземпляре SQL Azure и SQL Server

В этой статье описаны некоторые из основных отличий функциональности [службы машинного обучения в управляемый экземпляр SQL Azure](machine-learning-services-overview.md) и [SQL Server службы машинного обучения](https://docs.microsoft.com/sql/advanced-analytics/what-is-sql-server-machine-learning).

## <a name="language-support"></a>Поддержка языков

Службы машинного обучения как в SQL Управляемый экземпляр, так и в SQL Server поддерживают [платформу расширяемости](/sql/machine-learning/concepts/extensibility-framework)Python и R. Основные отличия в SQL Управляемый экземпляр:

- Поддерживаются только Python и R. Внешние языки, такие как Java, добавить нельзя.

- Первоначальные версии Python и R отличаются:

  | Платформа                   | Версия среды выполнения Python           | Версии среды выполнения R                   |
  |----------------------------|----------------------------------|--------------------------------------|
  | Управляемый экземпляр SQL Azure | 3.7.2                            | 3.5.2                                |
  | SQL Server 2019            | 3.7.1                            | 3.5.2                                |
  | SQL Server 2017            | 3.5.2 и 3.7.2 (CU22 и более поздние версии) | 3.3.3 и 3.5.2 (CU22 и более поздние версии)     |
  | SQL Server 2016            | Недоступно                    | 3.2.2 и 3.5.2 (SP2 CU14 и более поздние версии) |

## <a name="python-and-r-packages"></a>Пакеты Python и R

В SQL Управляемый экземпляр нет поддержки для пакетов, зависящих от внешних сред выполнения (например, Java) или для установки или использования доступа к API ОС.

Дополнительные сведения об управлении пакетами Python и R см. в следующих статьях:

- [Получение сведений о пакете Python](https://docs.microsoft.com/sql/machine-learning/package-management/python-package-information?context=/azure/azure-sql/managed-instance/context/ml-context&view=azuresqldb-mi-current&preserve-view=true)
- [Получение сведений о пакете R](https://docs.microsoft.com/sql/machine-learning/package-management/r-package-information?context=/azure/azure-sql/managed-instance/context/ml-context&view=azuresqldb-mi-current&preserve-view=true)

## <a name="resource-governance"></a>Управление ресурсами

В Управляемый экземпляр SQL невозможно ограничить ресурсы R с помощью [Resource Governor](/sql/relational-databases/resource-governor/resource-governor?view=azuresqldb-mi-current&preserve-view=true), а внешние пулы ресурсов не поддерживаются.

По умолчанию для ресурсов R задано не более 20% доступных ресурсов SQL Управляемый экземпляр, если включена расширяемость. Чтобы изменить этот процент по умолчанию, создайте запрос в службу поддержки Azure по адресу [https://azure.microsoft.com/support/create-ticket/](https://azure.microsoft.com/support/create-ticket/) .

Расширяемость включена с помощью следующих команд SQL (SQL Управляемый экземпляр будет перезапущен и будет недоступен в течение нескольких секунд):

```sql
sp_configure 'external scripts enabled', 1;
RECONFIGURE WITH OVERRIDE;
```

Чтобы отключить расширяемость и восстановить 100% ресурсов памяти и ЦП для SQL Server, используйте следующие команды:

```sql
sp_configure 'external scripts enabled', 0;
RECONFIGURE WITH OVERRIDE;
```

Общее количество ресурсов, доступных Управляемый экземпляр SQL, зависит от выбранного уровня служб. Дополнительные сведения см. в статье [Модели приобретения Базы данных SQL Azure](/azure/sql-database/sql-database-service-tiers).

### <a name="insufficient-memory-error"></a>Ошибка "недостаточно памяти"

Использование памяти зависит от объема памяти, используемого скриптами R, и от количества выполняемых параллельных запросов. Если объем доступной памяти для R недостаточен, вы получите сообщение об ошибке. Распространенные сообщения об ошибках:

- `Unable to communicate with the runtime for 'R' script for request id: *******. Please check the requirements of 'R' runtime`
- `'R' script error occurred during execution of 'sp_execute_external_script' with HRESULT 0x80004004. ...an external script error occurred: "..could not allocate memory (0 Mb) in C function 'R_AllocStringBuffer'"`
- `An external script error occurred: Error: cannot allocate vector of size.`

Если вы получаете одну из этих ошибок, ее можно устранить, увеличив масштаб базы данных до более высокого уровня служб.

## <a name="sql-managed-instance-pools"></a>Пулы Управляемый экземпляр SQL

Службы машинного обучения в настоящее время не поддерживается в [пулах управляемый экземпляр SQL Azure (Предварительная версия)](instance-pools-overview.md).

## <a name="next-steps"></a>Дальнейшие действия

- См. Обзор [службы машинного обучения в управляемый экземпляр SQL Azure](machine-learning-services-overview.md).
- Сведения об использовании Python в Службы машинного обучения см. в разделе [выполнение скриптов Python](/sql/machine-learning/tutorials/quickstart-python-create-script?context=/azure/azure-sql/managed-instance/context/ml-context&view=azuresqldb-mi-current&preserve-view=true).
- Сведения об использовании R в Службы машинного обучения см. в разделе [выполнение скриптов r](/sql/machine-learning/tutorials/quickstart-r-create-script?context=/azure/azure-sql/managed-instance/context/ml-context&view=azuresqldb-mi-current&preserve-view=true).
