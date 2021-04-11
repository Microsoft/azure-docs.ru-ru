---
title: Общие сведения об Azure Spring Cloud
description: Узнайте о преимуществах и возможностях развертывания приложений Java Spring и управления ими в Azure с помощью Azure Spring Cloud.
author: bmitchell287
ms.service: spring-cloud
ms.topic: overview
ms.date: 12/02/2020
ms.author: brendm
ms.custom: devx-track-java, contperf-fy21q2
customer intent: As an Azure Cloud user, I want to deploy, run, and monitor Spring Boot microservices.
ms.openlocfilehash: a92e535cb1edeb0eeaa285e442b4b24766e20ead
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104878855"
---
# <a name="what-is-azure-spring-cloud"></a>Что представляет собой Azure Spring Cloud?

Azure Spring Cloud позволяет легко развертывать в Azure приложения для микрослужб на основе Spring Boot без изменений кода.  Эта служба управляет инфраструктурой приложений Spring Cloud, благодаря чему разработчики могут сосредоточиться на коде.  Azure Spring Cloud обеспечивает управление жизненным циклом за счет комплексного мониторинга и диагностики, управления конфигурацией, обнаружения служб, интеграции CI/CD, выполнения развертываний по схеме Blue-Green и прочего.

## <a name="why-use-azure-spring-cloud"></a>Зачем использовать Azure Spring Cloud?

Развертывание приложений в Azure Spring Cloud имеет много преимуществ.  Можно сделать следующее:
* Эффективная миграция существующих приложений Spring и управление масштабированием и затратами в облаке.
* Модернизировать приложения с шаблонами Spring Cloud для повышения гибкости и скорости доставки.
* Запуск Java на уровне облака и более интенсивное использование без сложной инфраструктуры.
* Быстрая разработка и развертывание без зависимостей при контейнеризации.
* Эффективное отслеживание рабочих нагрузок.

Azure Spring Cloud поддерживает приложения Java [Spring Boot](https://spring.io/projects/spring-boot) и ASP.NET Core [Steeltoe](https://steeltoe.io/). Поддержка Steeltoe в настоящее время предлагается в общедоступной предварительной версии. Предложения в общедоступной предварительной версии позволяют экспериментировать с новыми функциями до официального выпуска. Общедоступные предварительные версии функций и служб не предназначены для использования в рабочей среде. Дополнительные сведения см. на странице [Часто задаваемые вопросы о поддержке Azure](https://azure.microsoft.com/support/faq/) или отправьте [запрос на поддержку](../azure-portal/supportability/how-to-create-azure-support-request.md).

## <a name="service-overview"></a>Обзор службы

Azure Spring Cloud, как часть экосистемы Azure, позволяет легко добавлять привязки к другим службам Azure, включая хранилище, базы данных, службы мониторинга и многое другое.  

  ![Обзор Azure Spring Cloud](media/spring-cloud-principles/azure-spring-cloud-overview.png)

* Azure Spring Cloud — это полностью управляемая служба для приложений Spring Boot, которая позволяет сосредоточиться на создании и запуске приложений, не заботясь об управлении инфраструктурой.

* Просто разверните свои файлы JAR или код, и Azure Spring Cloud автоматически подключит ваши приложения со средой выполнения службы Spring и встроенным жизненным циклом приложения.

* Мониторинг выполняется просто. После развертывания можно отслеживать производительность приложений, устранять ошибки и быстро улучшать приложения. 

* Полная интеграция с экосистемами и службами Azure.

* Решение Azure Spring Cloud готово к работе благодаря полностью управляемой инфраструктурой, встроенным управлением жизненным циклом и простым мониторингом.

## <a name="documentation-overview"></a>Обзор документации
В этой документации объясняется, как приступить к работе и использовать службы Azure Spring Cloud.

* Начало работы
    * [Запуск первого приложения](spring-cloud-quickstart.md)
    * [Подготовка службы Azure Spring Cloud](spring-cloud-quickstart-provision-service-instance.md)
    * [Настройка сервера конфигурации]()
    * [Сборка и развертывание приложений](spring-cloud-quickstart-deploy-apps.md)
    * [Использование журналов, метрик и трассировки](spring-cloud-quickstart-logs-metrics-tracing.md)
* Практические советы
    * [Разработка](how-to-prepare-app-deployment.md). Подготовка имеющегося приложения Java Spring к развертыванию в Azure Spring Cloud. При правильной настройке Azure Spring Cloud предоставляет надежные службы для мониторинга, масштабирования и обновления приложений Java Spring Cloud.
    * [Развертывание](spring-cloud-howto-staging-environment.md). Настройка промежуточного развертывания с помощью стратегии сине-зеленого развертывания в Azure Spring Cloud. Сине-зеленое развертывание — это шаблон непрерывной поставки Azure DevOps, который предполагает сохранение существующей (синей) версии при развертывании новой (зеленой).
    * [Настройка приложений](spring-cloud-howto-start-stop-delete.md).  Запуск, приостановка или удаление приложений Azure Spring Cloud. Изменение состояния приложения в Azure Spring Cloud с помощью портала Azure или Azure CLI.
    * [Масштабируемость.](spring-cloud-howto-scale-manual.md) Масштабирование любых приложений микрослужб с помощью панели мониторинга Azure Spring Cloud на портале Azure или с помощью параметров автомасштабирования. Общедоступные IP-адреса доступны для взаимодействия с внешними ресурсами, такими как базы данных, хранилище и хранилища ключей.
    * [Мониторинг приложений](spring-cloud-howto-distributed-tracing.md). Распределенные средства трассировки для упрощения отладки и отслеживания сложных проблем. Azure Spring Cloud интегрирует Spring Cloud Sleuth с Azure Application Insights. Эта интеграция предоставляет мощные возможности распределенной трассировки через портал Azure.
    * [Безопасные приложения](spring-cloud-howto-enable-system-assigned-managed-identity.md). Ресурсы Azure предоставляют автоматически управляемое удостоверение в Azure Active Directory. Это удостоверение можно использовать для аутентификации в любой службе, которая поддерживает аутентификацию Azure AD, не храня какие-либо учетные данные в коде.
    * [Интеграция с другими службами Azure](spring-cloud-howto-bind-cosmos.md). Вместо ручной настройки приложений с пружинной загрузкой можно автоматически привязать выбранные службы Azure к приложениям, например привязать приложения к базе данных Azure Cosmos DB.
    * [Автоматизация](spring-cloud-howto-cicd.md). Средства непрерывной интеграции и непрерывной поставки позволяют быстро развертывать обновления для существующих приложений с минимальными усилиями и рисками. Azure DevOps помогает организовывать и контролировать эти ключевые задачи. 
    * [Устранение неполадок](spring-cloud-howto-self-diagnose-solve.md). Служба диагностики Azure Spring Cloud предоставляет интерактивный интерфейс для устранения неполадок в приложениях. Настройка не требуется. При обнаружении проблем система диагностики Azure Spring Cloud выявляет проблемы и предоставляет сведения, помогающие устранить неполадки и решить проблемы.
    * [Миграция.](/azure/developer/java/migration/migrate-spring-boot-to-azure-spring-cloud) Как перенести существующее приложение Spring Cloud или приложение Spring Boot для запуска в Azure Spring Cloud.

 Чтобы начать работу, см. следующие статьи.

> [!div class="nextstepaction"]
> [Краткое руководство по Spring Cloud](spring-cloud-quickstart.md)

Примеры можно найти на сайте GitHub. [Примеры для Azure Spring Cloud](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/).