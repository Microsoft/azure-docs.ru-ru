---
title: Краткое руководство. Сборка и отправка образов контейнера Java в Реестр контейнеров Azure с помощью Maven и Jib
description: Создайте контейнерное приложение Java и отправьте его в Реестр контейнеров Azure с помощью подключаемого модуля Maven Jib.
author: KarlErickson
ms.author: karler
ms.date: 02/26/2020
ms.topic: quickstart
ms.custom:
- devx-track-java
- devx-track-azurecli
- mode-api
ms.openlocfilehash: 4d805458d90c73de879a9b87d5b08c98a8f1a250
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107537302"
---
# <a name="quickstart-build-and-push-java-container-images-to-azure-container-registry"></a>Краткое руководство. Сборка и отправка образов контейнера Java в Реестр контейнеров Azure

В этом кратком руководстве показано, как создать контейнерное приложение Java и отправить его в Реестр контейнеров Azure с помощью подключаемого модуля Maven Jib. Использование Maven и Jib — это один из примеров работы со средствами разработчика по взаимодействию с Реестром контейнеров Azure.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. Если у вас ее еще нет, вы можете активировать [Преимущества для подписчиков MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) или зарегистрироваться для получения [бесплатной учетной записи Azure](https://azure.microsoft.com/pricing/free-trial).
* [Интерфейс командной строки Azure (CLI)](/cli/azure/overview).
* Поддерживаемая версия Java Development Kit (JDK). Дополнительные сведения о версиях JDK, доступных для разработки в Azure, см. в статье <https://aka.ms/azure-jdks>.
* Средство сборки [Maven](http://maven.apache.org) Apache (версии 3 или более поздней).
* Клиент [Git](https://git-scm.com).
* Клиент [Docker](https://www.docker.com).
* [Вспомогательное приложение для учетных данных ACR Docker](https://github.com/Azure/acr-docker-credential-helper).

## <a name="create-the-spring-boot-on-docker-getting-started-web-app"></a>Создание веб-приложения Spring Boot в Docker

Ниже представлены инструкции по созданию веб-приложения Spring Boot и его локальному тестированию.

1. В командной строке выполните следующую команду, чтобы клонировать пример проекта по [началу работы с приложением Spring Boot в Docker](https://github.com/spring-guides/gs-spring-boot-docker).

   ```bash
   git clone https://github.com/spring-guides/gs-spring-boot-docker.git
   ```

1. Перейдите в каталог готового проекта.

   ```bash
   cd gs-spring-boot-docker/complete
   ```

1. Используйте Maven для создания и запуска примера приложения.

   ```bash
   mvn package spring-boot:run
   ```

1. Чтобы протестировать веб-приложение, перейдите по адресу `http://localhost:8080` или введите такую команду `curl`:

   ```bash
   curl http://localhost:8080
   ```

Должно появиться следующее сообщение: **Hello Docker World**.

## <a name="create-an-azure-container-registry-using-the-azure-cli"></a>Создание реестра контейнеров Azure с помощью Azure CLI

Далее предстоит создать группу ресурсов Azure и ACR, выполнив следующие действия.

1. Войдите в свою учетную запись Azure с помощью следующей команды:

   ```azurecli
   az login
   ```

1. Укажите подписку Azure, которую нужно использовать:

   ```azurecli
   az account set -s <subscription ID>
   ```

1. Создайте группу ресурсов Azure, используемых в этом руководстве. В следующей команде обязательно замените заполнители именем и расположением своего ресурса, например `eastus`.

   ```azurecli
   az group create \
       --name=<your resource group name> \
       --location=<location>
   ```

1. Создайте частный реестр контейнеров Azure в группе ресурсов с помощью следующей команды. Обязательно замените заполнители на фактические значения. Позднее в руководстве пример принудительно отправляется в этот реестр как образ Docker.

   ```azurecli
   az acr create \
       --resource-group <your resource group name> \
       --location <location> \
       --name <your registry name> \
       --sku Basic
   ```

## <a name="push-your-app-to-the-container-registry-via-jib"></a>Принудительная отправка приложения в реестр контейнеров с использованием Jib

Теперь вы можете обновить конфигурацию проекта и использовать командную строку для сборки и развертывания образа.

> [!NOTE]
> Чтобы войти в только что созданный реестр контейнеров Azure, необходимо запустить управляющую программу Docker. Чтобы установить Docker на компьютер, [ознакомьтесь с официальной документацией по Docker](https://docs.docker.com/install/).

1. Войдите в Реестр контейнеров с помощью Azure CLI, используя следующую команду. Обязательно замените заполнитель именем своего реестра.

   ```azurecli
   az configure --defaults acr=<your registry name>
   az acr login
   ```

   Команда `az configure` задает имя реестра по умолчанию для использования в командах `az acr`.

1. Перейдите в каталог завершенного проекта для приложения Spring Boot (например, *C:\SpringBoot\gs-spring-boot-docker\complete* или */users/robert/SpringBoot/gs-spring-boot-docker/complete*) и откройте файл *pom.xml* в текстовом редакторе.

1. Обновите коллекцию `<properties>` в файле *pom.xml* следующим кодом XML. Замените заполнитель именем своего реестра, а также добавьте свойство `<jib-maven-plugin.version>` со значением `2.2.0` или обновленную версию [jib-maven-plugin](https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin).

   ```xml
   <properties>
      <docker.image.prefix><your registry name>.azurecr.io</docker.image.prefix>
      <java.version>1.8</java.version>
      <jib-maven-plugin.version>2.2.0</jib-maven-plugin.version>
   </properties>
   ```

1. Обновите коллекцию `<plugins>` в файле *pom.xml* таким образом, чтобы элемент `<plugin>` содержал запись `jib-maven-plugin`, как показано в следующем примере. Обратите внимание, что мы используем базовый образ из Реестра контейнеров Майкрософт (MCR): `mcr.microsoft.com/java/jdk:8-zulu-alpine`, который содержит официально поддерживаемый пакет JDK для Azure. Другие базовые образы MCR с официально поддерживаемыми пакетами JDK см. в статьях [Java SE JDK](https://hub.docker.com/_/microsoft-java-jdk), [Java SE JRE](https://hub.docker.com/_/microsoft-java-jre), [Java SE Headless JRE](https://hub.docker.com/_/microsoft-java-jre-headless) и [Java SE JDK и Maven](https://hub.docker.com/_/microsoft-java-maven).

   ```xml
   <plugin>
     <artifactId>jib-maven-plugin</artifactId>
     <groupId>com.google.cloud.tools</groupId>
     <version>${jib-maven-plugin.version}</version>
     <configuration>
        <from>
            <image>mcr.microsoft.com/java/jdk:8-zulu-alpine</image>
        </from>
        <to>
            <image>${docker.image.prefix}/${project.artifactId}</image>
        </to>
     </configuration>
   </plugin>
   ```

1. Перейдите в каталог завершенного проекта для приложения Spring Boot и выполните указанную ниже команду для создания образа и его отправки в реестр:

   ```bash
   az acr login && mvn compile jib:build
   ```

> [!NOTE]
>
> По соображениям безопасности учетные данные, созданные `az acr login`, действительны только в течение 1 часа. При получении сообщения об ошибке *401 Не авторизовано* можно выполнить команду `az acr login -n <your registry name>` еще раз для повторной проверки подлинности.

## <a name="verify-your-container-image"></a>Проверка образа контейнера

Поздравляем! Теперь у вас есть отправленная в ACR контейнерная сборка приложения Java на основе JDK, поддерживаемого Azure. Вы можете протестировать образ, для этого разверните образ в Службе приложений Azure или извлеките его локально с помощью команды ниже (замените заполнитель):

```bash
docker pull <your registry name>.azurecr.io/gs-spring-boot-docker
```

## <a name="next-steps"></a>Дальнейшие действия

Другие версии официальных базовых образов Java, поддерживаемых корпорацией Майкрософт, см. в следующих статьях:

* [Пакет JDK для Java SE](https://hub.docker.com/_/microsoft-java-jdk);
* [Среда выполнения Java SE](https://hub.docker.com/_/microsoft-java-jre);
* [Среда выполнения Java SE для удаленного управления](https://hub.docker.com/_/microsoft-java-jre-headless);
* [Пакет JDK для Java SE и Maven](https://hub.docker.com/_/microsoft-java-maven)

Дополнительные сведения о Spring и Azure см. в центре документации об использовании Spring в Azure.

> [!div class="nextstepaction"]
> [Spring в Azure](/azure/developer/java/spring-framework)

### <a name="additional-resources"></a>Дополнительные ресурсы

Для получения дополнительных сведений см. следующие ресурсы:

* [Azure для разработчиков Java](/azure/java)
* [Working with Azure DevOps and Java](/azure/devops/java) (Работа с Azure DevOps и Java)
* [Spring Boot on Docker Getting Started](https://spring.io/guides/gs/spring-boot-docker)
* [Spring Initializr](https://start.spring.io)
* [Развертывание приложения Spring Boot Application в службе приложений Azure](/azure/developer/java/spring-framework/deploy-spring-boot-java-app-on-linux#configure-maven-to-build-image-to-your-azure-container-registry)
* [Применение пользовательского образа Docker для веб-приложения Azure на платформе Linux](../app-service/tutorial-custom-container.md)
