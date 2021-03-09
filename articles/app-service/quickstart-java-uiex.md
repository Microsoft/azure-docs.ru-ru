---
title: Краткое руководство. Создание приложения Java в Службе приложений Azure
description: Быстрое развертывание первого приложения Hello World на Java в Службе приложений Azure. Подключаемый модуль веб-приложения Azure для Maven упрощает развертывание приложений Java.
keywords: azure, служба приложений, веб-приложение, windows, linux, java, maven, краткое руководство
author: jasonfreeberg
ms.assetid: 582bb3c2-164b-42f5-b081-95bfcb7a502a
ms.devlang: Java
ms.topic: quickstart
ms.date: 08/01/2020
ms.author: jafreebe
ms.custom: mvc, seo-java-july2019, seo-java-august2019, seo-java-september2019
zone_pivot_groups: app-service-platform-windows-linux
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: 64e4c05e9439c164329dede5d714bec160bc5ae2
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102050381"
---
# <a name="quickstart-create-a-java-app-on-azure-app-service"></a>Краткое руководство. Создание приложения Java в Службе приложений Azure

[Служба приложений Azure](overview.md) — это служба веб-размещения с самостоятельной установкой исправлений и высоким уровнем масштабируемости.  В этом кратком руководстве показано, как использовать [Azure CLI](/cli/azure/get-started-with-azure-cli) с [подключаемым модулем веб-приложения Azure для Maven](https://github.com/Microsoft/azure-maven-plugins/tree/develop/azure-webapp-maven-plugin), чтобы развернуть JAR- или WAR-файл. 

Доступны также версии этой статьи для интегрированных сред разработки. См. подробнее об [Azure Toolkit for IntelliJ](/azure/developer/java/toolkit-for-intellij/create-hello-world-web-app) и [Azure Toolkit for Eclipse](/azure/developer/java/toolkit-for-eclipse/create-hello-world-web-app).

## <a name="1-prepare-your-environment"></a>1. Подготовка среды

Перед началом работы убедитесь, что у вас есть такие компоненты.

+ Обычного <abbr title="Профиль, который поддерживает данные для выставления счетов за использование Azure.">Учетная запись Azure</abbr> с активной <abbr title="Базовая организационная структура, в которой можно управлять ресурсами в Azure, обычно связанными с частным лицом или отделом в пределах организации.">Подписка</abbr>. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.

+ [Интерфейс командной строки Azure](/cli/azure/install-azure-cli).

+ [Java Developer Kit](/azure/developer/java/fundamentals/java-jdk-long-term-support) версии 8 или 11.

+ [Apache Maven](https://maven.apache.org) 3.0 или более поздней версии.

Подключаемый модуль Maven использует для развертывания в Службу приложений Azure данные учетной записи из Azure CLI. Прежде чем продолжить, [войдите с помощью Azure CLI](/cli/azure/authenticate-azure-cli). Дополнительные сведения см. в статье о [проверке подлинности с помощью подключаемых модулей Maven](https://github.com/microsoft/azure-maven-plugins/wiki/Authentication).

```azurecli
az login
```

<br>
<hr/>

## <a name="2-create-a-java-app"></a>2. Создание приложения Java

# <a name="java-se"></a>[Java SE](#tab/javase)

Клонируйте пример проекта [Spring Boot Getting Started](https://github.com/spring-guides/gs-spring-boot).

```bash
git clone https://github.com/spring-guides/gs-spring-boot
```

Перейдите в каталог готового проекта.

```bash
cd gs-spring-boot/complete
```

# <a name="tomcat"></a>[Tomcat](#tab/tomcat)

В приглашении Cloud Shell выполните следующую команду Maven, чтобы создать новое веб-приложение с именем `helloworld`.

```bash
mvn archetype:generate "-DgroupId=example.demo" "-DartifactId=helloworld" "-DarchetypeArtifactId=maven-archetype-webapp" "-Dversion=1.0-SNAPSHOT"
```

Затем измените рабочую папку на папку проекта:

```bash
cd helloworld
```

---

<br>
<hr/>

## <a name="3-configure-the-maven-plugin"></a>3. Настройка подключаемого модуля Maven

Воспользуйтесь приведенной ниже командой Maven для настройки развертывания. Эта команда поможет вам настроить операционную систему Службы приложений, версию Java и версию Tomcat.

```bash
mvn com.microsoft.azure:azure-webapp-maven-plugin:1.12.0:config
```

::: zone pivot="platform-windows"

# <a name="java-se"></a>[Java SE](#tab/javase)

1. При появлении запроса для параметра **Подписка** укажите правильную подписку (`Subscription`), введя соответствующий номер, указанный в начале строки.
1. При появлении запроса для параметра **Веб-приложение** примите параметр по умолчанию `<create>`, нажав клавишу ВВОД или выбрав существующее приложение.
1. При появлении запроса для параметра **Операционная система** выберите **Windows**, введя `3`.
1. При появлении запроса для параметра **Ценовая категория**, выберите **B2**, введя `2`.
1. Используйте версию Java по умолчанию (**Java 8**), нажав клавишу ВВОД.
1. Наконец, нажмите клавишу ВВОД в последнем запросе для подтверждения выбора.

    Сводные выходные данные будут выглядеть примерно так, как показано в следующем фрагменте кода.

    <pre class="is-monospace is-size-small has-padding-medium has-background-tertiary has-text-tertiary-invert">
    Please confirm webapp properties
    Subscription Id : ********-****-****-****-************
    AppName : spring-boot-1599007390755
    ResourceGroup : spring-boot-1599007390755-rg
    Region : westeurope
    PricingTier : Basic_B2
    OS : Windows
    Java : 1.8
    WebContainer : java 8
    Deploy to slot : false
    Confirm (Y/N)? : Y
    [INFO] Saving configuration to pom.
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 41.118 s
    [INFO] Finished at: 2020-09-01T17:43:45-07:00
    [INFO] ------------------------------------------------------------------------
    </pre>

# <a name="tomcat"></a>[Tomcat](#tab/tomcat)

1. При появлении запроса для параметра **Подписка** укажите правильную подписку (`Subscription`), введя соответствующий номер, указанный в начале строки.
1. При появлении запроса для параметра **Веб-приложение** примите параметр по умолчанию `<create>`, нажав клавишу ВВОД или выбрав существующее приложение.
1. При появлении запроса для параметра **Операционная система** выберите **Windows**, введя `3`.
1. При появлении запроса для параметра **Ценовая категория**, выберите **B2**, введя `2`.
1. Используйте версию Java по умолчанию (**Java 8**), нажав клавишу ВВОД.
1. Используйте веб-контейнер по умолчанию (**Tomcat 8.5**), нажав клавишу ВВОД.
1. Наконец, нажмите клавишу ВВОД в последнем запросе для подтверждения выбора.

    Сводные выходные данные будут выглядеть примерно так, как показано в следующем фрагменте кода.

    <pre class="is-monospace is-size-small has-padding-medium has-background-tertiary has-text-tertiary-invert">
    Please confirm webapp properties
    Subscription Id : ********-****-****-****-************
    AppName : helloworld-1599003152123
    ResourceGroup : helloworld-1599003152123-rg
    Region : westeurope
    PricingTier : Basic_B2
    OS : Windows
    Java : 1.8
    WebContainer : tomcat 8.5
    Deploy to slot : false
    Confirm (Y/N)? : Y
    [INFO] Saving configuration to pom.
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 03:03 min
    [INFO] Finished at: 2020-09-01T16:35:30-07:00
    [INFO] ------------------------------------------------------------------------
    </pre>

---

::: zone-end
::: zone pivot="platform-linux"

### <a name="java-se"></a>[Java SE](#tab/javase)

1. При появлении запроса для параметра **Подписка** укажите правильную подписку (`Subscription`), введя соответствующий номер, указанный в начале строки.
1. При появлении запроса для параметра **Веб-приложение** примите параметр по умолчанию `<create>`, нажав клавишу ВВОД или выбрав существующее приложение.
1. При появлении запроса для параметра **Операционная система** выберите **Linux**, нажав клавишу ВВОД.
1. При появлении запроса для параметра **Ценовая категория**, выберите **B2**, введя `2`.
1. Используйте версию Java по умолчанию (**Java 8**), нажав клавишу ВВОД.
1. Наконец, нажмите клавишу ВВОД в последнем запросе для подтверждения выбора.

    <pre class="is-monospace is-size-small has-padding-medium has-background-tertiary has-text-tertiary-invert">
    Please confirm webapp properties
    Subscription Id : ********-****-****-****-************
    AppName : spring-boot-1599007116351
    ResourceGroup : spring-boot-1599007116351-rg
    Region : westeurope
    PricingTier : Basic_B2
    OS : Linux
    RuntimeStack : JAVA 8-jre8
    Deploy to slot : false
    Confirm (Y/N)? : Y
    [INFO] Saving configuration to pom.
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 20.925 s
    [INFO] Finished at: 2020-09-01T17:38:51-07:00
    [INFO] ------------------------------------------------------------------------
    </pre>

### <a name="tomcat"></a>[Tomcat](#tab/tomcat)

1. При появлении запроса для параметра **Подписка** укажите правильную подписку (`Subscription`), введя соответствующий номер, указанный в начале строки.
1. При появлении запроса для параметра **Веб-приложение** примите параметр по умолчанию `<create>`, нажав клавишу ВВОД или выбрав существующее приложение.
1. При появлении запроса для параметра **Операционная система** выберите **Linux**, нажав клавишу ВВОД.
1. При появлении запроса для параметра **Ценовая категория**, выберите **B2**, введя `2`.
1. Используйте версию Java по умолчанию (**Java 8**), нажав клавишу ВВОД.
1. Используйте веб-контейнер по умолчанию (**Tomcat 8.5**), нажав клавишу ВВОД.
1. Наконец, нажмите клавишу ВВОД в последнем запросе для подтверждения выбора.

    <pre class="is-monospace is-size-small has-padding-medium has-background-tertiary has-text-tertiary-invert">
    Please confirm webapp properties
    Subscription Id : ********-****-****-****-************
    AppName : helloworld-1599003744223
    ResourceGroup : helloworld-1599003744223-rg
    Region : westeurope
    PricingTier : Basic_B2
    OS : Linux
    RuntimeStack : TOMCAT 8.5-jre8
    Deploy to slot : false
    Confirm (Y/N)? : Y
    [INFO] Saving configuration to pom.
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 50.785 s
    [INFO] Finished at: 2020-09-01T16:43:09-07:00
    [INFO] ------------------------------------------------------------------------
    </pre>

---

::: zone-end

При необходимости вы можете изменить другие параметры Службы приложений непосредственно в `pom.xml`. Некоторые из наиболее распространенных перечислены ниже.

Свойство | Обязательно | Описание | Версия
---|---|---|---
`<schemaVersion>` | false | Указывает версию схемы конфигурации. Поддерживаемые значения: `v1` и `v2`. | 1.5.2
`<subscriptionId>` | false | Укажите идентификатор подписки. | Версия 0.1.0 и выше
`<resourceGroup>` | Да | Azure <abbr title="Логический контейнер для связанных ресурсов Azure, которыми можно управлять как единым целым.">resource group</abbr> для вашего веб-приложения. | Версия 0.1.0 и выше
`<appName>` | Да | Название вашего веб-приложения. | Версия 0.1.0 и выше
`<region>` | Да | Указывает регион, в котором будет размещено ваше веб-приложение (значение по умолчанию: **westeurope**). Определить допустимые регионы можно в разделе [Поддерживаемые регионы](https://azure.microsoft.com/global-infrastructure/services/?products=app-service). | Версия 0.1.0 и выше
`<pricingTier>` | false | Ценовая категория веб-приложения. Значение по умолчанию — **P1V2** для производственной рабочей нагрузки. Вариант **B2** является рекомендуемым минимумом для разработки и тестирования в среде Java. [Дополнительные сведения](https://azure.microsoft.com/pricing/details/app-service/linux/)| Версия 0.1.0 и выше
`<runtime>` | Да | Конфигурация среды выполнения. Дополнительные сведения см. [здесь](https://github.com/microsoft/azure-maven-plugins/wiki/Azure-Web-App:-Configuration-Details). | Версия 0.1.0 и выше
`<deployment>` | Да | Конфигурация развертывания. Дополнительные сведения см. [здесь](https://github.com/microsoft/azure-maven-plugins/wiki/Azure-Web-App:-Configuration-Details). | Версия 0.1.0 и выше

Запишите значения `<appName>` и `<resourceGroup>`(`helloworld-1590394316693` и `helloworld-1590394316693-rg` соответственно в выходных данных примера), так как они понадобятся позже.

[Сообщите о проблеме](https://www.research.net/r/javae2e?tutorial=quickstart-java&step=config)

<br>
<hr/>

## <a name="4-deploy-the-app"></a>4. Развертывание приложения

Разверните приложение Java в Azure, используя команду приведенную ниже.

```bash
mvn package azure-webapp:deploy
```

После завершения развертывания ваше приложение будет готово к работе по адресу `http://<appName>.azurewebsites.net/`. Откройте URL-адрес в своем локальном веб-браузере. Вы должны увидеть следующее:

![Приложение, работающее в Службе приложений Azure](./media/quickstart-java/java-hello-world-in-browser-azure-app-service.png)

**Поздравляем!** Вы развернули свое первое приложение Java в службе приложений.

[Сообщите о проблеме](https://www.research.net/r/javae2e?tutorial=app-service-linux-quickstart&step=deploy)

<br>
<hr/>

## <a name="5-clean-up-resources"></a>5. Очистка ресурсов

Чтобы избежать дополнительных расходов, удалите приложение Java и связанные ресурсы.

```azurecli
az group delete --name <your resource group name; for example: helloworld-1558400876966-rg> --yes
```

Ее выполнение может занять до минуты.

<br>
<hr/>

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Подключение к Базе данных Azure для PostgreSQL на Java](../postgresql/connect-java.md)

> [!div class="nextstepaction"]
> [Настройка процесса непрерывной интеграции и доставки](deploy-continuous-deployment.md)

> [!div class="nextstepaction"]
> [Сведения о ценах](https://azure.microsoft.com/pricing/details/app-service/linux/)

> [!div class="nextstepaction"]
> [Агрегирование журналов и метрик](troubleshoot-diagnostic-logs.md)

> [!div class="nextstepaction"]
> [Увеличение масштаба](manage-scale-up.md)

> [!div class="nextstepaction"]
> [Ресурсы Azure для разработчиков Java](/java/azure/)

> [!div class="nextstepaction"]
> [Настройка приложения Java](configure-language-java.md)
