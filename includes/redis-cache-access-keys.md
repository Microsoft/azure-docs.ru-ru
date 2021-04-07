---
title: включить файл
description: включить файл
services: redis-cache
author: wesmc7777
ms.service: cache
ms.topic: include
ms.date: 11/05/2019
ms.author: wesmc
ms.custom: include file
ms.openlocfilehash: a737e130d616a67bab28c7c96c0372216a6707af
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96002474"
---
### <a name="retrieve-host-name-ports-and-access-keys-from-the-azure-portal"></a>Получение имени узла, портов и ключей доступа с помощью портала Azure

Чтобы подключаться к экземпляру Кэша Azure для Redis, клиентам кэша требуются имя узла, сведения о портах и ключ кэша. Некоторые клиенты могут ссылаться на эти элементы с помощью незначительно различающихся имен. Вы можете получить имя узла, порты и ключи на [портале Azure](https://portal.azure.com).

- Чтобы получить ключи доступа, в области навигации кэша слева щелкните **Ключи доступа**. 
  
  ![Ключи кэша Azure для Redis](media/redis-cache-access-keys/redis-cache-keys.png)

- Чтобы получить имя узла и порты, в области навигации кэша слева щелкните **Свойства**. Имя узла имеет вид *\<DNS name>.redis.cache.windows.net*.

  ![Свойства кэша Azure для Redis](media/redis-cache-access-keys/redis-cache-hostname-ports.png)

