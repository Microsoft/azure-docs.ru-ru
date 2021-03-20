---
title: Клиентские API-интерфейсы Java в Azure Service Fabric
description: Создание и применение клиентских API-интерфейсов Java в Azure Service Fabric с использованием спецификации клиентских REST API в Service Fabric
author: rapatchi
ms.topic: conceptual
ms.date: 11/27/2017
ms.custom: devx-track-java
ms.author: rapatchi
ms.openlocfilehash: 24ee4a3d8109626bf93e01fbd10c00140762c9c6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87324630"
---
# <a name="azure-service-fabric-java-client-apis"></a>Клиентские API-интерфейсы Java в Azure Service Fabric

Клиентские API-интерфейсы Service Fabric позволяют развертывать приложения и контейнеры, основанные на микрослужбах, и управлять ими в кластере Service Fabric в Azure (локально, на локальном компьютере разработки или в другом облаке). В этой статье описывается создание и использование клиентских API Java Service Fabric поверх клиентских REST API Service Fabric.

## <a name="generate-the-client-code-using-autorest"></a>Создание кода клиента с использованием AutoRest

[AutoRest](https://github.com/Azure/autorest) — средство, создающее клиентские библиотеки для получения доступа к веб-службам RESTful. Входные данные для AutoRest — спецификация, описывающая REST API в формате спецификации OpenAPI. [Клиентские REST API в Service Fabric](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/servicefabric/data-plane) следуют данной спецификации.

Выполните действия, описанные ниже, чтобы создать клиентский код Java Service Fabric с помощью средства AutoRest.

1. Установите Node.js и NPM на компьютере.

    При использовании Linux выполните следующее:
    ```bash
    sudo apt-get install npm
    sudo apt install nodejs
    ```
    При использовании Mac OS X выполните следующее:
    ```bash
    brew install node
    ```

2. Установите AutoRest с помощью NPM.
    ```bash
    npm install -g autorest
    ```

3. Создайте вилку и клонируйте репозиторий [azure-rest-api-specs](https://github.com/Azure/azure-rest-api-specs) на локальном компьютере, а затем перейдите в расположение клонированного репозитория из терминала на вашем компьютере.


4. Перейдите в указанное ниже расположение в клонированном репозитории.
    ```bash
    cd specification\servicefabric\data-plane\Microsoft.ServiceFabric\stable\6.0
    ```

    > [!NOTE]
    > Если версия кластера не 6.0. *, перейдите в соответствующий каталог в стабильной папке.
    >   

5. Выполните следующую команду autorest, чтобы создать клиентский код Java.
    
    ```bash
    autorest --input-file= servicefabric.json --java --output-folder=[output-folder-name] --namespace=[namespace-of-generated-client]
    ```
   Ниже приведен пример, демонстрирующий использование команды autorest.
   
    ```bash
    autorest --input-file=servicefabric.json --java --output-folder=java-rest-api-code --namespace=servicefabricrest
    ```
   
   Следующая команда принимает файл спецификации ``servicefabric.json`` в качестве входных данных и создает клиентский код Java в папке ``java-rest-api-     code``, а затем включает этот код в пространство имен ``servicefabricrest``. После этого у вас будет две папки (``models`` и ``implementation``) и два файла (``ServiceFabricClientAPIs.java`` и ``package-info.java``), созданные в папке ``java-rest-api-code``.


## <a name="include-and-use-the-generated-client-in-your-project"></a>Добавление созданного клиента в проект и его использование

1. Добавьте созданный код в проект соответствующим образом. Мы советуем создать библиотеку с помощью сгенерированного кода и добавить ее в проект.
2. При создании библиотеки рекомендуется добавить следующие зависимости в проект библиотеки. Если вы используете другой подход, добавьте зависимость другим образом.

    ```
        GroupId:  com.microsoft.rest
        Artifactid: client-runtime
        Version: 1.2.1
    ```
    Например, если вы используете систему сборки Maven, добавьте в свой файл ``pom.xml`` следующее:

    ```xml
        <dependency>
          <groupId>com.microsoft.rest</groupId>
          <artifactId>client-runtime</artifactId>
          <version>1.2.1</version>
        </dependency>
    ```

3. Создайте RestClient, используя следующий код:

    ```java
        RestClient simpleClient = new RestClient.Builder()
            .withBaseUrl("http://<cluster-ip or name:port>")
            .withResponseBuilderFactory(new ServiceResponseBuilder.Factory())
            .withSerializerAdapter(new JacksonAdapter())
            .build();
        ServiceFabricClientAPIs client = new ServiceFabricClientAPIsImpl(simpleClient);
    ```
4. Используйте клиентский объект и при необходимости выполните соответствующие вызовы. Ниже приведены некоторые примеры, демонстрирующие использование клиентского объекта. Предполагается, что перед использованием следующих API вы создали и отправили пакет приложения в хранилище образов.
    * Подготовка приложения
    
        ```java
            ApplicationTypeImageStorePath imageStorePath = new ApplicationTypeImageStorePath();
            imageStorePath.withApplicationTypeBuildPath("<application-path-in-image-store>");
            client.provisionApplicationType(imageStorePath);
        ```
    * Создание приложения

        ```java
            ApplicationDescription applicationDescription = new ApplicationDescription();
            applicationDescription.withName("<application-uri>");
            applicationDescription.withTypeName("<application-type>");
            applicationDescription.withTypeVersion("<application-version>");
            client.createApplication(applicationDescription);
        ```

## <a name="understanding-the-generated-code"></a>Основные сведения о созданном коде
Для каждого API имеется четыре метода перегрузки реализации. Если имеются необязательные параметры, вы увидите еще четыре дополнительных варианта (включая эти необязательные параметры). Рассмотрим для примера API ``removeReplica``.
 1. **public void removeReplica (строка nodeName, UUID partitionId, строка replicaId, логическое значение forceRemove, долгое время ожидания)**
    * Это синхронный вариант вызова API removeReplica.
 2. **public ServiceFuture\<Void> removeReplicaAsync (строка nodeName, UUID partitionId, строка replicaId, логическое значение forceRemove, долгое время ожидания, итоговый ServiceCallback \<Void> serviceCallback)**
    * Этот вариант вызова API можно применять, если необходимо использовать обратные вызовы и асинхронное программирование на основе событий в будущем.
 3. **public Observable\<Void> removeReplicaAsync (строка nodeName, UUID partitionId, строка replicaId)**
    * Этот вариант вызова API можно применять, если необходимо использовать реактивное асинхронное программирование.
 4. **общедоступный наблюдаемый \<ServiceResponse\<Void>> ремоверепликависсервицереспонсеасинк (строка nodeName, UUID PartitionID, строка replicaId)**
    * Этот вариант вызова API можно применять, если необходимо использовать реактивное асинхронное программирование и работать с ответом REST без форматирования.

## <a name="next-steps"></a>Дальнейшие действия
* Дополнительные сведения см. в статье [REST API Service Fabric](/rest/api/servicefabric/).
