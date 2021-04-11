---
title: Привязка Кэша Azure для Redis к приложению Azure Spring Cloud
description: Узнайте, как привязать кэш Azure для Redis к приложению Azure Spring Cloud
author: bmitchell287
ms.service: spring-cloud
ms.topic: how-to
ms.date: 10/31/2019
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 414f7dc64027b286d8a963452d2b86126b3c5818
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104878552"
---
# <a name="bind-azure-cache-for-redis-to-your-azure-spring-cloud-application"></a>Привязка Кэша Azure для Redis к приложению Azure Spring Cloud 

**Эта статья применима к:** ✔️ Java

Чтобы не настраивать приложения Spring Boot вручную, вы можете автоматически привязать выбранные службы Azure к приложениям с помощью Azure Spring Cloud. В этой статье показано, как привязать приложение к Кэшу Azure для Redis.

## <a name="prerequisites"></a>Предварительные требования

* Развернутый облачный экземпляр Azure Spring Cloud
* Экземпляр службы кэша Azure для Redis
* Расширение Azure Spring Cloud для Azure CLI.

Если у вас еще не развернут экземпляр Azure Spring Cloud, выполните процедуру в [этом кратком руководстве](spring-cloud-quickstart.md), чтобы развернуть приложение Azure Spring Cloud.

## <a name="bind-azure-cache-for-redis"></a>Привязка кэша Azure для Redis

1. Добавьте следующую зависимость в файл pom.xml. проекта:

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
    </dependency>
    ```
1. Удалите все свойства `spring.redis.*` из файла `application.properties`.

1. Обновите текущую развернутую службу с помощью `az spring-cloud app update` или создайте новое развертывание с помощью `az spring-cloud app deployment create`.

1. Перейдите к странице своей службы Azure Spring Cloud на портале Azure. Перейдите на **панель мониторинга приложений** и выберите приложение для привязки к Кэшу Azure для Redis. Это то же приложение, которое вы обновили или развернули на предыдущем шаге.

1. Выберите **Service binding** (Привязка службы), а затем — **Создание привязки службы**. Заполняя форму, не забудьте выбрать в качестве **типа привязки** значение **Кэш Azure для Redis**, а также сервер Кэша Azure для Redis и **первичный** ключ.

1. Перезапустите приложение. Теперь привязка должна работать.

1. Чтобы обеспечить правильную привязку службы, выберите имя привязки и проверьте сведения о ней. Поле `property` должно выглядеть следующим образом:
    ```
    spring.redis.host=some-redis.redis.cache.windows.net
    spring.redis.port=6380
    spring.redis.password=abc******
    spring.redis.ssl=true
    ```

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали, как привязать приложение Azure Spring Cloud к Кэшу Azure для Redis. Дополнительные сведения о привязке служб к приложению см. в статье [Привязка к экземпляру Базы данных Azure для MySQL](spring-cloud-howto-bind-mysql.md).
