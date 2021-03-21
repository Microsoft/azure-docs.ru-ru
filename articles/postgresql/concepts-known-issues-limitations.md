---
title: 'Известные проблемы и ограничения: база данных Azure для PostgreSQL — один сервер и гибкий сервер (Предварительная версия)'
description: Список известных проблем, о которых следует знать клиентам.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: conceptual
ms.date: 02/05/2020
ms.openlocfilehash: 3c26368e6281ad74d617891ba15c980117b25a6e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100106427"
---
# <a name="azure-database-for-postgresql---known-issues-and-limitations"></a>База данных Azure для PostgreSQL — известные проблемы и ограничения

На этой странице приведен список известных проблем в базе данных Azure для PostgreSQL, которые могут повлиять на работу приложения. В нем также перечислены все меры и рекомендации по решению проблемы.

## <a name="intelligent-performance---query-store"></a>Интеллектуальное хранилище запросов с интеллектуальной производительностью

Применимо к базе данных Azure для PostgreSQL — Single Server.

| Применимо | Причина | Серверы|
| ----- | ------ | ---- | 
| PostgreSQL 9,6, 10, 11 | Включение параметра сервера `pg_qs.replace_parameter_placeholders` может привести к завершению работы сервера в некоторых редких ситуациях. | С помощью портала Azure, раздела Параметры сервера включите значение параметра `pg_qs.replace_parameter_placeholders` в `OFF` и сохраните.   | 

## <a name="server-parameters"></a>Параметры сервера

Применимо к базе данных Azure для PostgreSQL — один сервер и гибкий сервер

| Применимо | Причина | Серверы| 
| ----- | ------ | ---- | 
| PostgreSQL 9,6, 10, 11 | Изменение параметра сервера `max_locks_per_transaction` на большее значение, чем [рекомендовано](https://www.postgresql.org/docs/11/kernel-resources.html) , может привести к тому, что сервер не сможет запуститься после перезагрузки. | Оставьте значение по умолчанию (32 или 64) или задайте разумное значение для каждой PostgreSQL [документации](https://www.postgresql.org/docs/11/kernel-resources.html). <br> <br> Со стороны службы это работает, чтобы ограничить высокую ценность в зависимости от номера SKU.  | 

## <a name="next-steps"></a>Дальнейшие действия
- См. [рекомендации](./concepts-query-store-best-practices.md) по хранилищу запросов
