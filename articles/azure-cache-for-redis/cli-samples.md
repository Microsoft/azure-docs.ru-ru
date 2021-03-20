---
title: Управление кэшем Azure для Redis с помощью Azure CLI
description: 'Azure CLI примеры управления кэшем Azure для Redis: создание кэша, удаление кэша, получение сведений о кэше, имя узла, порты и ключи, подключение веб-приложения.'
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.devlang: azurecli
ms.topic: conceptual
ms.date: 04/14/2017
ms.custom: devx-track-azurecli
ms.openlocfilehash: 32530982bc2a9d1b5deb31b3bc71460462352258
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92536425"
---
# <a name="manage-azure-cache-for-redis-with-azure-cli"></a>Управление кэшем Azure для Redis с помощью Azure CLI

В следующей таблице содержатся ссылки на скрипты Bash, созданные с помощью Azure CLI.

| Создание кэша | Описание |
| ------------ | ----------- |
| [Создание кэша](./scripts/create-cache.md) | Создает группу ресурсов и базовый уровень кэша Azure для Redis. |
| [Создание кэша уровня "Премиум" с кластеризацией](./scripts/create-premium-cache-cluster.md) | Создает группу ресурсов и кэш уровня "Премиум" с включенной кластеризацией.|
| [Получение сведений о кэше](./scripts/show-cache.md) | Возвращает сведения об экземпляре кэша Azure для Redis, включая состояние подготовки. |
| [Получение имени узла, портов и ключей](./scripts/cache-keys-ports.md) | Возвращает имя узла, порты и ключи для экземпляра кэша Azure для Redis. |
|**Веб-приложение и кэш**| **Описание**|
| [Подключение веб-приложения к кэшу Azure для Redis](./../app-service/scripts/cli-connect-to-redis.md) | Создает веб-приложение Azure и кэш Azure для Redis, а затем добавляет сведения о подключении Redis в параметры приложения. |
|**Удалить кэш**| **Описание** |
| [Удаление кэша](./scripts/delete-cache.md) | Удаляет экземпляр кэша Azure для Redis.  |

Дополнительные сведения об Azure CLI см. в статьях [Установка интерфейса командной строки Azure](/cli/azure/install-azure-cli) и [Приступая к работе с Azure CLI](/cli/azure/get-started-with-azure-cli).