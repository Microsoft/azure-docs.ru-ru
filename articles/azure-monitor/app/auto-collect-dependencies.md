---
title: Автоматический сбор зависимостей в Azure Application Insights | Документация Майкрософт
description: Автоматический сбор и визуализация зависимостей Application Insights
ms.topic: reference
ms.custom: devx-track-dotnet
ms.date: 05/06/2020
ms.openlocfilehash: 8a4d79e52465e93fb4db2625217cb37a06917218
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91930872"
---
# <a name="dependency-auto-collection"></a>Автоматический сбор зависимостей

Ниже приведен список поддерживаемых в настоящее время вызовов зависимостей, которые автоматически определяются как зависимости без каких-либо дополнительных изменений в код приложения. Эти зависимости отображаются в представлениях [схемы приложений](./app-map.md) и [диагностики транзакций](./transaction-diagnostics.md) Application Insights. Если зависимость отсутствует в списке ниже, можно по-прежнему отслеживать ее самостоятельно с помощью [отслеживания вызова зависимости](./api-custom-events-metrics.md#trackdependency).

## <a name="net"></a>.NET

| Платформы приложений| Версии |
| ------------------------|----------|
| Веб-формы ASP.NET | 4.5+ |
| ASP.NET MVC 3 | 4+ |
| ASP.NET WebAPI | 4.5+ |
| ASP.NET Core | 1.1+ |
| <b>Библиотеки обмена данными</b> |
| [HttpClient](https://www.microsoft.com/net/) | 4.5+, .NET Core 1.1+ |
| [SqlClient](https://www.nuget.org/packages/System.Data.SqlClient) | .NET Core 1.0+, NuGet 4.3.0 |
| [Microsoft.Data.SqlClient](https://www.nuget.org/packages/Microsoft.Data.SqlClient/1.1.2)| 1.1.0 — последний стабильный выпуск. (См. Примечание ниже.)
| [Пакет SDK для клиента концентраторов событий](https://www.nuget.org/packages/Microsoft.Azure.EventHubs) | 1.1.0 |
| [Пакет SDK для клиента служебной шины](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus) | 3.0.0 |
| <b>Клиенты хранилища</b>|  |
| ADO.NET | 4.5+ |

> [!NOTE]
> Существует [известная ошибка](https://github.com/microsoft/ApplicationInsights-dotnet/issues/1347) , связанная с более старыми версиями Microsoft. Data. SqlClient. Для устранения этой проблемы рекомендуется использовать 1.1.0 или более позднюю версию. Entity Framework Core не обязательно поставляется с последним стабильным выпуском Microsoft. Data. SqlClient, поэтому мы рекомендуем подтвердить, что вы по крайней мере 1.1.0, чтобы избежать этой проблемы.   


## <a name="java"></a>Java
| Серверы приложений | Версии |
|-------------|----------|
| [Tomcat](https://tomcat.apache.org/) | 7, 8 | 
| [JBoss EAP](https://developers.redhat.com/products/eap/download/) | 6, 7 |
| [Jetty](https://www.eclipse.org/jetty/) | 9 |
| <b>Платформы приложений</b> |  |
| [Spring](https://spring.io/) | 3.0 |
| [Spring Boot](https://spring.io/projects/spring-boot) | 1.5.9 +<sup>*</sup> |
| Сервлет Java | 3.1+ |
| <b>Библиотеки связи</b> |  |
| [HTTP-клиент Apache](https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient) | 4.3 +<sup>†</sup> |
| <b>Клиенты хранилища</b> | |
| [SQL Server]( https://mvnrepository.com/artifact/com.microsoft.sqlserver/mssql-jdbc) | 1 +<sup>†</sup> |
| [PostgreSQL (бета-поддержка)](https://github.com/Microsoft/ApplicationInsights-Java/blob/master/CHANGELOG.md#version-240-beta) | |
| [Oracle]( https://www.oracle.com/technetwork/database/application-development/jdbc/downloads/index.html) | 1 +<sup>†</sup> |
| [MySql]( https://mvnrepository.com/artifact/mysql/mysql-connector-java) | 1 +<sup>†</sup> |
| <b>Библиотеки ведения журналов</b> | |
| [Logback](https://logback.qos.ch/) | 1+ |
| [Log4j](https://logging.apache.org/log4j/) | 1.2+ |
| <b>Метрики библиотек</b> |  |
| JMX | 1.0+ |

> [!NOTE]
> *За исключением поддержки программирования.
> <br>†Требуется установка [агента виртуальной машины Java](./java-agent.md#install-the-application-insights-agent-for-java).

## <a name="nodejs"></a>Node.js

| Библиотеки обмена данными | Версии |
| ------------------------|----------|
| [http](https://nodejs.org/api/http.html), [HTTPS](https://nodejs.org/api/https.html) | 0.10+ |
| <b>Клиенты хранилища</b> | |
| [Redis](https://www.npmjs.com/package/redis) | 2.x |
| [MongoDb](https://www.npmjs.com/package/mongodb); [MongoDb Core](https://www.npmjs.com/package/mongodb-core) | 2. x-3. x |
| [MySQL](https://www.npmjs.com/package/mysql) | 2.0.0-2.16. x |
| [PostgreSQL](https://www.npmjs.com/package/pg); | 6. x-7. x |
| [PG-пул](https://www.npmjs.com/package/pg-pool) | 1. x-2. x |
| <b>Библиотеки ведения журналов</b> | |
| [Консоль](https://nodejs.org/api/console.html) | 0.10+ |
| [Bunyan](https://www.npmjs.com/package/bunyan) | 1.x |
| [Winston](https://www.npmjs.com/package/winston) | 2. x-3. x |

## <a name="javascript"></a>JavaScript

| Библиотеки обмена данными | Версии |
| ------------------------|----------|
| [XMLHttpRequest](https://developer.mozilla.org/docs/Web/API/XMLHttpRequest) | Все |

## <a name="next-steps"></a>Дальнейшие действия

- Настройка отслеживания зависимостей для платформы [.NET](./asp-net-dependencies.md).
- Настройка отслеживания зависимостей для [Java](./java-agent.md).
- Настройка пользовательского отслеживания зависимостей для [Опенценсус Python](./opencensus-python-dependency.md).
- [Создание телеметрии настраиваемых зависимостей](./api-custom-events-metrics.md#trackdependency)
- В [этой статье](./data-model.md) представлены типы данных и модель данных для Application Insights.
- Ознакомление с [платформами](./platforms.md), поддерживаемыми Application Insights.

