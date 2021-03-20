---
title: Автоматизация реестра и обнаружения служб
description: Узнайте, как автоматизировать обнаружение и регистрацию служб с помощью реестра пружины облачной службы
author: bmitchell287
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 09/08/2020
ms.author: brendm
ms.custom: devx-track-java
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: 1e60799878cc30b729344c03df36a4c5e4f4a199
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90904221"
---
# <a name="discover-and-register-your-spring-cloud-services"></a>Обнаружение и регистрация облачных служб весны

Обнаружение службы является ключевым требованием для архитектуры на основе микрослужб.  Настройка каждого клиента вручную занимает некоторое время и дает возможность человеческим ошибкам.  Эта проблема решается в реестре облачной службы Azure весны.  После настройки сервер реестра службы будет управлять регистрацией и обнаружением служб для микрослужб вашего приложения. Сервер реестра служб поддерживает реестр развернутых микрослужб, обеспечивает балансировку нагрузки на стороне клиента и отделяет поставщиков услуг от клиентов, не полагаясь на DNS.

::: zone pivot="programming-language-csharp"
Сведения о настройке регистрации службы для приложения Стилтое см. [в статье Подготовка приложения Java весны для развертывания в Azure веснного облака](spring-cloud-tutorial-prepare-app-deployment.md).
::: zone-end
::: zone pivot="programming-language-java"

## <a name="register-your-application-using-spring-cloud-service-registry"></a>Регистрация приложения с помощью реестра пружины облачной службы

Прежде чем приложение сможет управлять регистрацией и обнаружением служб с помощью реестра пружины облачной службы, в файл *pom.xml* приложения необходимо добавить несколько зависимостей.
Включите в *pom.xml* зависимости для *пружины-Cloud-Starter-Netflix-Еурека-Client* и *пружины-Cloud-Starter-Azure-пружины-Cloud-Client* .

```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>spring-cloud-starter-azure-spring-cloud-client</artifactId>
        <version>2.1.0</version>
    </dependency>
```

## <a name="update-the-top-level-class"></a>Обновление класса верхнего уровня

Наконец, мы добавим аннотацию к классу верхнего уровня приложения.

 ```java
    package foo.bar;

    import org.springframework.boot.SpringApplication;
    import org.springframework.cloud.client.SpringCloudApplication;

    @SpringCloudApplication
    public class DemoApplication {
        public static void main(String... args) {
            SpringApplication.run(DemoApplication.class, args);
        }
    }
 ```

Конечная точка сервера реестра пружины облачной службы будет внедрена в приложение в качестве переменной среды.  Микрослужбы теперь смогут регистрироваться на сервере реестра службы и обнаруживать другие зависимые микрослужбы.

Обратите внимание, что для распространения изменений с сервера на все микрослужбы может потребоваться несколько минут.
::: zone-end
