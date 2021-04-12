---
title: Привязка экземпляра Базы данных Azure для MySQL к приложению Azure Spring Cloud
description: Сведения о том, как привязать экземпляр Базы данных Azure для MySQL к приложению Azure Spring Cloud.
author: bmitchell287
ms.service: spring-cloud
ms.topic: how-to
ms.date: 11/04/2019
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: cc1c186f9acb6d49314f5d581a4b51ffe49d6627
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104878370"
---
# <a name="bind-an-azure-database-for-mysql-instance-to-your-azure-spring-cloud-application"></a>привязке экземпляра Базы данных Azure для MySQL к приложению Azure Spring Cloud 

**Эта статья применима к:** ✔️ Java

С помощью Azure Spring Cloud можно автоматически привязать выбранные службы Azure к приложениям, не настраивая приложение Spring Boot вручную. Из этой статьи вы узнаете, как привязать приложение к экземпляру Базы данных Azure для MySQL.

## <a name="prerequisites"></a>Предварительные требования

* Развернутый облачный экземпляр Azure Spring Cloud
* Учетная запись базы данных Azure для MySQL
* Azure CLI

Если у вас нет развернутого экземпляра Azure Spring Cloud, выполните инструкции, приведенные в статье [Краткое руководство. Запуск приложения Azure Spring Cloud с помощью портала Azure](spring-cloud-quickstart.md), чтобы развернуть первое приложение Azure Spring Cloud.

## <a name="bind-your-app-to-your-azure-database-for-mysql-instance"></a>Привязка приложения к экземпляру Базы данных Azure для MySQL

1. Запишите имя и пароль администратора учетной записи Базы данных Azure для MySQL. 

1. Подключитесь к серверу, создайте базу данных с именем **testdb** из клиента MySQL, а затем создайте учетную запись без прав администратора.

1. Добавьте в файл проекта *pom.xml* следующую зависимость:

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    ```
1. В файле *application.properties* удалите свойство `spring.datasource.*`.

1. Обновите текущее развертывание, выполнив команду `az spring-cloud app update`, или создайте новое развертывание для этого изменения с помощью команды `az spring-cloud app deployment create`.  Эти команды либо обновляют, либо создают приложение с новой зависимостью.

1. На портале Azure на странице службы **Azure Spring Cloud** найдите **Панель мониторинга приложения** и выберите приложение, которое нужно привязать к экземпляру Базы данных Azure для MySQL.  Это то же приложение, которое вы обновили или развернули на предыдущем шаге. 

1. Выберите **Service binding** (Привязка службы), а затем нажмите кнопку **Создание привязки службы**. 

1. Заполните форму, выбрав **Azure MySQL** в качестве **типа привязки**. Укажите то же имя базы данных, которое вы использовали ранее, и те же имя пользователя и пароль, которые были записаны на первом шаге.

1. Перезапустите приложение, и эта привязка должна сразу начать работать.

1. Чтобы проверить правильность привязки службы, выберите имя привязки и проверьте сведения о ней. Поле `property` должно выглядеть следующим образом:
    ```
    spring.datasource.url=jdbc:mysql://some-server.mysql.database.azure.com:3306/testdb?useSSL=true&requireSSL=false&useLegacyDatetimeCode=false&serverTimezone=UTC
    spring.datasource.username=admin@some-server
    spring.datasource.password=abc******
    spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
    ```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье описывается, как привязать приложение Azure Spring Cloud к экземпляру Базы данных Azure для MySQL. Дополнительные сведения о привязке служб к приложению см. в статье [Привязка Базы данных Azure Cosmos DB к приложению Azure Spring Cloud](spring-cloud-howto-bind-cosmos.md).
