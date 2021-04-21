---
title: Краткое руководство. Создание приложения Spring Boot в Azure Service Fabric
description: В этом кратком руководстве вы развернете приложение Spring Boot для Azure Service Fabric, используя пример приложения Spring Boot.
ms.date: 01/29/2019
ms.topic: quickstart
ms.custom:
- mvc
- devcenter
- seo-java-august2019
- seo-java-september2019
- devx-track-java
- mode-api
ms.openlocfilehash: ad01697349eab20aa55fc6f6b59bc0d317c5a7e2
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107530863"
---
# <a name="quickstart-deploy-a-java-spring-boot-app-on-azure-service-fabric"></a>Краткое руководство. Развертывание приложения Java Spring Boot в Azure Service Fabric

В этом кратком руководстве мы развернем приложение Java Spring Boot в Azure Service Fabric, используя уже знакомые нам средства командной строки в Linux и MacOS. Azure Service Fabric — это платформа распределенных систем для развертывания микрослужб и контейнеров и управления ими. 

## <a name="prerequisites"></a>Предварительные требования

#### <a name="linux"></a>[Linux](#tab/linux)

- [Среда Java](./service-fabric-get-started-linux.md#set-up-java-development) и [Yeoman](./service-fabric-get-started-linux.md#set-up-yeoman-generators-for-containers-and-guest-executables).
- [Пакет SDK и интерфейс командной строки (CLI) для Service Fabric](./service-fabric-get-started-linux.md#installation-methods).
- [Git](https://git-scm.com/downloads);

#### <a name="macos"></a>[MacOS](#tab/macos)

- [Среда Java и Yeoman](./service-fabric-get-started-mac.md#create-your-application-on-your-mac-by-using-yeoman).
- [Пакет SDK и интерфейс командной строки (CLI) для Service Fabric](./service-fabric-cli.md#cli-mac).
- [Git](https://git-scm.com/downloads);

--- 

## <a name="download-the-sample"></a>Скачивание примера приложения

В окне терминала выполните следующую команду, чтобы клонировать пример [начального приложения](https://github.com/spring-guides/gs-spring-boot) Spring Boot на локальный компьютер.

```bash
git clone https://github.com/spring-guides/gs-spring-boot.git
```

## <a name="build-the-spring-boot-application"></a>Создание приложения Spring Boot 
В каталоге *gs-spring-boot/complete* выполните указанную ниже команду, чтобы скомпилировать приложение. 

```bash
./gradlew build
``` 

## <a name="package-the-spring-boot-application"></a>Упаковка приложения Spring Boot 
1. В каталоге *gs-spring-boot* клонированного приложения выполните команду `yo azuresfguest`. 

1. Введите следующие сведения для каждого запроса.

    ![Записи Spring Boot Yeoman](./media/service-fabric-quickstart-java-spring-boot/yeoman-entries-spring-boot.png)

1. В папке *SpringServiceFabric/SpringServiceFabric/SpringGettingStartedPkg/code* создайте файл с именем *entryPoint.sh*. Добавьте в файл *entryPoint.sh* следующий код: 

    ```bash
    #!/bin/bash
    BASEDIR=$(dirname $0)
    cd $BASEDIR
    java -jar *spring-boot*.jar
    ```

1. Добавьте ресурс **Endpoints** в файл *gs-spring-boot/SpringServiceFabric/SpringServiceFabric/SpringGettingStartedPkg/ServiceManifest.xml*.

    ```xml 
        <Resources>
          <Endpoints>
            <Endpoint Name="WebEndpoint" Protocol="http" Port="8080" />
          </Endpoints>
       </Resources>
    ```

    Файл *ServiceManifest.xml* теперь выглядит следующим образом: 

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <ServiceManifest Name="SpringGettingStartedPkg" Version="1.0.0"
                     xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" >

       <ServiceTypes>
          <StatelessServiceType ServiceTypeName="SpringGettingStartedType" UseImplicitHost="true">
       </StatelessServiceType>
       </ServiceTypes>

       <CodePackage Name="code" Version="1.0.0">
          <EntryPoint>
             <ExeHost>
                <Program>entryPoint.sh</Program>
                <Arguments></Arguments>
                <WorkingFolder>CodePackage</WorkingFolder>
             </ExeHost>
          </EntryPoint>
       </CodePackage>
        <Resources>
          <Endpoints>
            <Endpoint Name="WebEndpoint" Protocol="http" Port="8080" />
          </Endpoints>
       </Resources>
     </ServiceManifest>
    ```

Итак, на данный момент у вас есть приложение Service Fabric, созданное на основе примера "Начало работы" для Spring Boot, которое теперь можно развернуть в Service Fabric.

## <a name="run-the-application-locally"></a>Локальный запуск приложения

1. Запустите локальный кластер на компьютерах Ubuntu, выполнив следующую команду:

    ```bash
    sudo /opt/microsoft/sdk/servicefabric/common/clustersetup/devclustersetup.sh
    ```

    Если вы пользуетесь компьютером Mac, запустите локальный кластер из образа Docker (при условии, что выполнены [предварительные требования](./service-fabric-get-started-mac.md#create-a-local-container-and-set-up-service-fabric) по настройке локального кластера для Mac). 

    ```bash
    docker run --name sftestcluster -d -p 19080:19080 -p 19000:19000 -p 25100-25200:25100-25200 -p 8080:8080 mysfcluster
    ```

    Запуск локального кластера занимает некоторое время. Чтобы убедиться, что кластер работает, откройте Service Fabric Explorer по адресу `http://localhost:19080`. Наличие пяти работоспособных узлов означает, что локальный кластер запущен и работает. 
    
    ![В Service Fabric Explorer отображаются работоспособные узлы](./media/service-fabric-quickstart-java-spring-boot/service-fabric-explorer-healthy-nodes.png)

1. Откройте папку *gs-spring-boot/SpringServiceFabric*.
1. Выполните следующую команду, чтобы подключиться к локальному кластеру.

    ```bash
    sfctl cluster select --endpoint http://localhost:19080
    ```
1. Выполните сценарий *install.sh*.

    ```bash
    ./install.sh
    ```

1. Откройте любой веб-браузер и перейдите к приложению по адресу `http://localhost:8080`.

    ![Пример Spring Boot Service Fabric](./media/service-fabric-quickstart-java-spring-boot/spring-boot-service-fabric-sample.png)

Теперь вы можете получить доступ к приложению Spring Boot, которое развернуто в кластере Service Fabric.

Дополнительные сведения см. на странице [начального приложения](https://spring.io/guides/gs/spring-boot/) Spring Boot.

## <a name="scale-applications-and-services-in-a-cluster"></a>Масштабирование приложений и служб в кластере

Службы могут легко масштабироваться в кластере с учетом изменения нагрузки на них. Масштабирование службы осуществляется путем изменения числа экземпляров, запущенных в кластере. Существует множество способов масштабирования служб. Например, можно использовать скрипты или команды интерфейса командной строки Service Fabric (sfctl). В следующих шагах используется Service Fabric Explorer.

Средство Service Fabric Explorer работает во всех кластерах Service Fabric. Чтобы его открыть, укажите адрес кластера и порт управления HTTP (19080) для кластера в адресной строке браузера, например `http://localhost:19080`.

Для масштабирования службы веб-интерфейса сделайте следующее:

1. Откройте Service Fabric Explorer в своем кластере (например, по ссылке `http://localhost:19080`).
1. Щелкните многоточие (**...**) рядом с узлом **fabric:/SpringServiceFabric/SpringGettingStarted** в представлении в виде дерева и выберите **Масштабировать службу**.

    ![Пример масштабирования службы Service Fabric Explorer](./media/service-fabric-quickstart-java-spring-boot/service-fabric-explorer-scale-sample.png)

    Теперь вы можете выбрать нужное количество экземпляров службы.

1. Измените количество на **3** и щелкните **Масштабировать службу**.

    Масштабирование службы можно настроить и с помощью командной строки, как описано ниже.

    ```bash
    # Connect to your local cluster
    sfctl cluster select --endpoint https://<ConnectionIPOrURL>:19080 --pem <path_to_certificate> --no-verify

    # Run Bash command to scale instance count for your service
    sfctl service update --service-id 'SpringServiceFabric~SpringGettingStarted' --instance-count 3 --stateless 
    ``` 

1. Выберите узел **fabric:/SpringServiceFabric/SpringGettingStarted** в представлении в виде дерева и разверните узел раздела (здесь отображается его идентификатор GUID).

    ![Масштабирование службы в Service Fabric Explorer завершено](./media/service-fabric-quickstart-java-spring-boot/service-fabric-explorer-partition-node.png)

    У службы есть три экземпляра. Представление в виде дерева демонстрирует, на каких узлах они запущены.

С помощью этой простой задачи управления вы удвоили количество ресурсов для обработки пользовательской нагрузки для службы веб-интерфейса. Важно понимать, что для надежной работы службы не требуется запускать несколько экземпляров службы. При сбое в работе службы Service Fabric запускает новый экземпляр службы в кластере.

## <a name="fail-over-services-in-a-cluster"></a>Службы отработки отказа в кластере

Чтобы продемонстрировать отработку отказа службы, с помощью Service Fabric Explorer моделируется перезагрузка узла. Убедитесь, что запущен только один экземпляр службы.

1. Откройте Service Fabric Explorer в своем кластере (например, по ссылке `http://localhost:19080`).
1. Щелкните многоточие (**...**) рядом с узлом, на котором запущен экземпляр этой службы, и перезапустите узел.

    ![Service Fabric Explorer с командой перезапуска узла](./media/service-fabric-quickstart-java-spring-boot/service=fabric-explorer-restart=node.png)
1. Этот экземпляр службы перемещается в другой узел без простоев в работе приложения.

    ![Service Fabric Explorer после успешного перезапуска узла](./media/service-fabric-quickstart-java-spring-boot/service-fabric-explorer-service-moved.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве рассматривались следующие темы:

* Развертывание приложения Spring Boot в Service Fabric
* Развертывание приложения в локальном кластере.
* Масштабирование приложения на несколько узлов
* Отработка отказа службы без ущерба для ее доступности

Дополнительные сведения о работе с приложениями Java в Service Fabric см. в руководстве по приложениям Java.

> [!div class="nextstepaction"]
> [Развертывание приложения Java](./service-fabric-tutorial-create-java-app.md)
