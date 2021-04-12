---
title: Использование динамической конфигурации в приложении Spring Boot
titleSuffix: Azure App Configuration
description: Сведения о динамическом обновлении данных конфигурации для приложений Spring Boot.
services: azure-app-configuration
author: mrm9084
ms.service: azure-app-configuration
ms.topic: tutorial
ms.date: 12/09/2020
ms.custom: devx-track-java
ms.author: mametcal
ms.openlocfilehash: 590f221b0a4980d462267dd8c3a73ca7d02583fd
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105625523"
---
# <a name="tutorial-use-dynamic-configuration-in-a-java-spring-app"></a>Руководство по использованию динамической конфигурации в приложении Java Spring

Конфигурация приложения включает две библиотеки для Spring. Для `spring-cloud-azure-appconfiguration-config` требуется Spring Boot и зависимость от `spring-cloud-context`. Для `spring-cloud-azure-appconfiguration-config-web` требуется Spring Web и Spring Boot. Обе библиотеки поддерживают активацию вручную для проверки обновленных значений конфигурации. `spring-cloud-azure-appconfiguration-config-web` также предусматривает поддержку автоматической проверки обновления конфигурации.

Вы можете обновить значения конфигурации без перезапуска приложения, хотя это приведет к повторному созданию всех объектов Bean в `@RefreshScope`. Клиентская библиотека кэширует идентификатор хэша текущих загруженных конфигураций, чтобы избежать слишком большого количества вызовов к хранилищу конфигураций. До истечения срока действия кэшированного значения параметра операция обновления не приводит к обновлению значения, даже если оно изменилось в хранилище конфигураций. Срок действия по умолчанию для каждого запроса составляет 30 секунд. При необходимости это значение можно переопределить.

Автоматическое обновление `spring-cloud-azure-appconfiguration-config-web` активируется на основе действия, а именно `ServletRequestHandledEvent` Spring Web. Если `ServletRequestHandledEvent` не активируется, автоматическое обновление `spring-cloud-azure-appconfiguration-config-web` не активирует обновление, даже если срок действия кэша истек.

## <a name="use-manual-refresh"></a>Выполнение обновления вручную

Служба "Конфигурация приложений" предоставляет `AppConfigurationRefresh`, чтобы можно было проверить истечение срока действия кэша. В таком случае активируется обновление.

```java
import com.microsoft.azure.spring.cloud.config.AppConfigurationRefresh;

...

@Autowired
private AppConfigurationRefresh appConfigurationRefresh;

...

public void myConfigurationRefreshCheck() {
    Future<Boolean> triggeredRefresh = appConfigurationRefresh.refreshConfigurations();
}
```

`refreshConfigurations()` `AppConfigurationRefresh` возвращает значение `Future`, которое равно true, если обновление активировано, или false, если нет. False означает, что срок действия кэша не истек, изменения отсутствуют или другой поток в настоящий момент проверяет наличие обновления.

## <a name="use-automated-refresh"></a>Использовать автоматическое обновление

Чтобы использовать автоматическое обновление, запустите приложение Spring Boot, использующее Конфигурацию приложений, например приложение, созданное с помощью [краткого руководства по созданию приложения Spring Boot для службы "Конфигурация приложений"](quickstart-java-spring-app.md).

Затем откройте файл *pom.xml* в текстовом редакторе и добавьте `<dependency>` для `spring-cloud-azure-appconfiguration-config-web`.

**Spring Cloud 1.1.x**

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>spring-cloud-azure-appconfiguration-config-web</artifactId>
    <version>1.1.5</version>
</dependency>
```

**Spring Cloud 1.2.x**

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>spring-cloud-azure-appconfiguration-config-web</artifactId>
    <version>1.2.7</version>
</dependency>
```

## <a name="run-and-test-the-app-locally"></a>Локальный запуск и проверка приложения

1. Создайте приложение Spring Boot с помощью Maven и запустите его.

    ```shell
    mvn clean package
    mvn spring-boot:run
    ```

1. В браузере перейдите по адресу `http://localhost:8080`.  Вы увидите сообщение, связанное с ключом.

    Вы также можете использовать инструмент *curl* для проверки приложения, например следующим образом: 
    
    ```cmd
    curl -X GET http://localhost:8080/
    ```

1. Чтобы проверить динамическую конфигурацию, откройте портал Конфигурации приложений Azure, связанный с вашим приложением. Выберите **Обозреватель конфигураций** и обновите значение отображаемого ключа, например следующим образом:

    | Клавиши | Значение |
    |---|---|
    | application/config.message | Hello - Updated |

1. Обновите страницу браузера, чтобы просмотреть новое отображаемое сообщение.

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого руководства вы включили в приложении Spring Boot динамическое обновление параметров конфигурации из службы "Конфигурация приложения". Чтобы узнать, как с помощью удостоверения, управляемого Azure, упростить доступ к службе "Конфигурация приложений Azure", перейдите к следующему учебнику.

> [!div class="nextstepaction"]
> [Руководство по интеграции с управляемыми удостоверениями Azure](./howto-integrate-azure-managed-service-identity.md)
