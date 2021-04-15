---
author: wesmc7777
ms.service: redis-cache
ms.topic: include
ms.date: 11/09/2018
ms.author: wesmc
ms.openlocfilehash: 1ab6243be39bf30bc060ed5745fbf600924743a9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "71839230"
---
| Ресурс | Ограничение |
| --- | --- |
| Объем кэша |1,2 ТБ |
| Базы данных |64 |
| Максимальное количество подключенных клиентов |40 000 |
| Реплики Кэша Redis для Azure для обеспечения высокого уровня доступности |1 |
| Сегменты кэша уровня "Премиум" с включенной кластеризацией |10 |

Ограничения и размер кэша Redis для Azure отличаются для каждой ценовой категории. Информацию о ценовых категориях и соответствующих им размерах см. на странице [цен на Кэш Redis для Azure](https://azure.microsoft.com/pricing/details/cache/).

Дополнительные сведения об ограничениях конфигурации кэша Redis для Azure см. в разделе о [конфигурации сервера Redis по умолчанию](../articles/azure-cache-for-redis/cache-configure.md#default-redis-server-configuration).

Так как настройка экземпляров кэша Redis для Azure и управление ими осуществляется корпорацией Майкрософт, кэш Redis для Azure поддерживает не все команды Redis. Дополнительные сведения см. в разделе о [командах Redis, неподдерживаемых в кэше Redis для Azure](../articles/azure-cache-for-redis/cache-configure.md#redis-commands-not-supported-in-azure-cache-for-redis).

