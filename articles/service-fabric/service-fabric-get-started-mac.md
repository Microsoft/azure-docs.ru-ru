---
title: Настройка среды разработки на macOS
description: Установите среду выполнения, пакет SDK и инструменты и создайте локальный кластер разработки. После завершения установки вы будете готовы к созданию приложений на macOS.
ms.topic: conceptual
ms.date: 10/16/2020
ms.custom: devx-track-js
ms.openlocfilehash: d08046c8f29901dd9650a1edc886efa2ff226e00
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93086783"
---
# <a name="set-up-your-development-environment-on-mac-os-x"></a>Настройка среды разработки для Mac OS X
> [!div class="op_single_selector"]
> * [Windows](service-fabric-get-started.md)
> * [Linux](service-fabric-get-started-linux.md)
> * [Mac OS X](service-fabric-get-started-mac.md)

Вы можете собирать приложения Azure Service Fabric на базе Mac OS X, которые работают в кластерах Linux. Из этого документа вы узнаете, как настроить систему Mac для разработки приложений.

## <a name="prerequisites"></a>Предварительные условия
Платформа Azure Service Fabric изначально не предназначена для работы в Mac OS X. Чтобы запустить локальный кластер Service Fabric, предоставляется предварительно настроенный образ контейнера Docker. Перед началом работы вам потребуются:

* Не менее 4 ГБ ОЗУ.
* Последняя версия [Docker](https://www.docker.com/).

>[!TIP]
>
>Чтобы установить Docker на компьютере Mac, выполните инструкции, приведенные в [документации Docker](https://docs.docker.com/docker-for-mac/install/#what-to-know-before-you-install). Затем [проверьте правильность установки](https://docs.docker.com/docker-for-mac/#check-versions-of-docker-engine-compose-and-machine).
>

## <a name="create-a-local-container-and-set-up-service-fabric"></a>Создание локального контейнера и настройка Service Fabric
Чтобы настроить локальный контейнер Docker и запустить его в кластере Service Fabric, выполните следующие действия:

1. Обновите конфигурацию управляющей программы Docker в узле с помощью следующих параметров и перезапустите управляющую программу Docker: 

    ```json
    {
        "ipv6": true,
        "fixed-cidr-v6": "fd00::/64"
    }
    ```
    Вы можете обновить эти параметры непосредственно в файле daemon.json, который находится в папке установки Docker. Вы можете непосредственно изменить параметры конфигурации управляющей программы в Docker. Нажмите **значок Docker**, а затем последовательно выберите **Параметры** > **Управляющая программа** > **Дополнительно**.
    
    >[!NOTE]
    >
    >Рекомендуется изменить управляющую программу непосредственно в DOCKER, так как расположение daemon.jsв файле может отличаться от компьютера к компьютеру. Например, ~/Library/Containers/com.docker.docker/Data/database/com.docker.driver.amd64-linux/etc/docker/daemon.json.
    >

    >[!TIP]
    >При тестировании больших приложений мы рекомендуем увеличить число ресурсов, выделенных для Docker. Для этого щелкните **значок Docker**, выберите **Дополнительно** и настройте число ядер и объем памяти.

2. Запустите кластер.<br/>
    <b>Ubuntu 18,04 LTS:</b>
    ```bash
    docker run --name sftestcluster -d -v /var/run/docker.sock:/var/run/docker.sock -p 19080:19080 -p 19000:19000 -p 25100-25200:25100-25200 mcr.microsoft.com/service-fabric/onebox:u18
    ```

    <b>Ubuntu 16,04 LTS:</b>
    ```bash
    docker run --name sftestcluster -d -v /var/run/docker.sock:/var/run/docker.sock -p 19080:19080 -p 19000:19000 -p 25100-25200:25100-25200 mcr.microsoft.com/service-fabric/onebox:u16
    ```

    >[!TIP]
    > По умолчанию вы получите образ с последней версией Service Fabric. Конкретные редакции см. на странице [Service Fabric OneBox](https://hub.docker.com/_/microsoft-service-fabric-onebox) в центре DOCKER.



3. (Необязательно.) создайте расширенный образ Service Fabric.

    В новом каталоге создайте файл `Dockerfile` с именем для создания настраиваемого образа:

    >[!NOTE]
    >Вы можете адаптировать изображение выше с помощью Dockerfile, чтобы добавить в контейнер дополнительные программы или зависимости.
    >Например, добавленный элемент `RUN apt-get install nodejs -y` разрешает поддержку приложений `nodejs` в качестве гостевых исполняемых файлов.
    ```Dockerfile
    FROM mcr.microsoft.com/service-fabric/onebox:u18
    RUN apt-get install nodejs -y
    EXPOSE 19080 19000 80 443
    WORKDIR /home/ClusterDeployer
    CMD ["./ClusterDeployer.sh"]
    ```
    
    >[!TIP]
    > По умолчанию вы получите образ с последней версией Service Fabric. Конкретные редакции см. на странице [центра DOCKER](https://hub.docker.com/r/microsoft/service-fabric-onebox/) .

    Чтобы создать повторно используемое изображение из `Dockerfile` , откройте терминал, а затем запустите его, а `cd` `Dockerfile` затем выполните команду:

    ```bash 
    docker build -t mysfcluster .
    ```
    
    >[!NOTE]
    >Для выполнения этой операции потребуется некоторое время, но она выполняется только один раз.

    Теперь можно быстро запустить локальную копию Service Fabric при необходимости, запустив:

    ```bash 
    docker run --name sftestcluster -d -v /var/run/docker.sock:/var/run/docker.sock -p 19080:19080 -p 19000:19000 -p 25100-25200:25100-25200 mysfcluster
    ```

    >[!TIP]
    >Укажите имя экземпляра контейнера, которое будет более удобным для чтения. 
    >
    >Если приложение ожидает передачи данных через определенные порты, их необходимо указать с помощью дополнительных тегов `-p`. Например, если приложение ожидает передачи данных через порт 8080, добавьте следующий тег `-p`:
    >
    >`docker run -itd -p 19000:19000 -p 19080:19080 -p 8080:8080 --name sfonebox mcr.microsoft.com/service-fabric/onebox:u18`
    >

4. Для запуска кластера потребуется некоторое время. При запуске можно просмотреть журналы с помощью следующей команды или перейти на панель мониторинга, чтобы просмотреть работоспособность кластеров: `http://localhost:19080`

    ```bash 
    docker logs sftestcluster
    ```


5. Чтобы отключить и очистить контейнер, используйте следующую команду. Тем не менее мы будем использовать этот контейнер на следующем шаге.

    ```bash 
    docker rm -f sftestcluster
    ```

### <a name="known-limitations"></a>Известные ограничения 
 
 Ниже перечислены известные ограничения для локального кластера, запущенного в контейнере для Mac: 
 
 * Служба DNS не запускается и в настоящее время не поддерживается в контейнере. [#132 проблемы](https://github.com/Microsoft/service-fabric/issues/132)
 * Для выполнения приложений на основе контейнеров необходимо запустить на узле Linux. Вложенные приложения контейнера в настоящее время не поддерживаются.

## <a name="set-up-the-service-fabric-cli-sfctl-on-your-mac"></a>Настройка интерфейса командной строки Service Fabric (sfctl) на компьютере Mac

Чтобы установить интерфейс командной строки Service Fabric (`sfctl`) на компьютере Mac, см. инструкции в [этом разделе](service-fabric-cli.md#cli-mac).
Команды интерфейса командной строки поддерживают взаимодействие с сущностями Service Fabric, включая кластеры, приложения и службы.

1. Чтобы подключиться к кластеру перед развертыванием приложений, выполните следующую команду: 

```bash
sfctl cluster select --endpoint http://localhost:19080
```

## <a name="create-your-application-on-your-mac-by-using-yeoman"></a>Создание приложения на компьютере Mac с помощью Yeoman

Service Fabric предоставляет средства формирования шаблонов, которые позволяют создать приложение Service Fabric из терминала с помощью генератора шаблонов Yeoman. Чтобы установить генератор шаблонов Yeoman для Service Fabric на компьютере, выполните следующие действия:

1. Установите Node.js и диспетчер пакетов Node (NPM) на компьютере Mac. Это можно сделать с помощью [HomeBrew](https://brew.sh/), как показано ниже:

    ```bash
    brew install node
    node -v
    npm -v
    ```
2. Установите на компьютере генератор шаблонов [Yeoman](https://yeoman.io/) из NPM.

    ```bash
    npm install -g yo
    ```
3. Установите генератор Yeoman, следуя инструкциям из [документации](service-fabric-get-started-linux.md#set-up-yeoman-generators-for-containers-and-guest-executables) по началу работы. Чтобы создавать приложения Service Fabric с помощью Yeoman, сделайте следующее:

    ```bash
    npm install -g generator-azuresfjava       # for Service Fabric Java Applications
    npm install -g generator-azuresfguest      # for Service Fabric Guest executables
    npm install -g generator-azuresfcontainer  # for Service Fabric Container Applications
    ```
4. Установив генераторы, создайте службы гостевых исполняемых файлов или службы контейнеров с помощью команды `yo azuresfguest` или `yo azuresfcontainer` соответственно.

5. Чтобы создать приложение Service Fabric Java на компьютере Mac, установите JDK 1.8 и Gradle на хост-компьютере. Это можно сделать с помощью [HomeBrew](https://brew.sh/), как показано ниже: 

    ```bash
    brew update
    brew cask install java
    brew install gradle
    ```

    > [!IMPORTANT]
    > В текущих версиях `brew cask install java` может устанавливаться более новая версия JDK.
    > Не забудьте установить JDK 8.

## <a name="deploy-your-application-on-your-mac-from-the-terminal"></a>Развертывание приложения на компьютере Mac из терминала

Когда вы создадите и соберете приложение Service Fabric, можно развернуть его с помощью [интерфейса командной строки Service Fabric](service-fabric-cli.md#cli-mac).

1. Подключитесь к кластеру Service Fabric, в котором выполняется экземпляр контейнера, на компьютере Mac.

    ```bash
    sfctl cluster select --endpoint http://localhost:19080
    ```

2. Перейдите к каталогу проекта и запустите скрипт установки.

    ```bash
    cd MyProject
    bash install.sh
    ```

## <a name="set-up-net-core-31-development"></a>Настройка разработки .NET Core 3,1

Установите [пакет SDK для .NET Core 3,1 для Mac](https://www.microsoft.com/net/core#macos) , чтобы начать [Создание приложений Service Fabric C#](service-fabric-create-your-first-linux-application-with-csharp.md). Пакеты для Service Fabric приложений .NET Core размещаются в NuGet.org.

## <a name="install-the-service-fabric-plug-in-for-eclipse-on-your-mac"></a>Установка подключаемого модуля Service Fabric для Eclipse на компьютере Mac

Azure Service Fabric предоставляет подключаемый модуль для Eclipse Neon (или более поздней версии) для интегрированной среды разработки Java. Он упрощает процесс создания и развертывания служб Java. Чтобы установить или обновить подключаемый модуль Service Fabric для Eclipse до последней версии, выполните [эти действия](service-fabric-get-started-eclipse.md#install-or-update-the-service-fabric-plug-in-in-eclipse). Другие шаги в [документации Service Fabric для Eclipse](service-fabric-get-started-eclipse.md) также применимы: сборка приложения, добавление службы в приложение, удаление приложения и т. д.

На последнем шаге создается экземпляр контейнера с тем же путем, что и для узла. Для подключаемого модуля требуется, чтобы экземпляр этого типа работал в контейнере Docker на компьютере Mac. Пример:

```bash
docker run -itd -p 19080:19080 -v /Users/sayantan/work/workspaces/mySFWorkspace:/tmp/mySFWorkspace --name sfonebox mcr.microsoft.com/service-fabric/onebox:latest
```

Атрибуты определяются следующим образом:
* `/Users/sayantan/work/workspaces/mySFWorkspace` — полный путь к рабочей области на компьютере Mac.
* `/tmp/mySFWorkspace` — это путь внутри контейнера. С ним следует сопоставить рабочую область.

>[!NOTE]
> 
>Если вы используете другие путь и имя рабочей области, обновите эти значения в команде `docker run`.
> 
>Если вы запускаете контейнер с именем, отличающимся от `sfonebox`, обновите его в файле testclient.sh в приложении Java субъекта Service Fabric.
>

## <a name="next-steps"></a>Дальнейшие действия
<!-- Links -->
* [Создание первого приложения Azure Service Fabric](service-fabric-create-your-first-linux-application-with-java.md)
* [Создание и развертывание первого приложения Service Fabric Java в Linux с использованием подключаемого модуля Service Fabric для Eclipse](service-fabric-get-started-eclipse.md).
* [Создание кластера Service Fabric в Azure с помощью портала Azure](service-fabric-cluster-creation-via-portal.md)
* [Создание кластера Service Fabric в Azure с помощью Azure Resource Manager](service-fabric-cluster-creation-via-arm.md).
* [Моделирование приложения в Service Fabric](service-fabric-application-model.md)
* [Manage an Azure Service Fabric application by using Azure Service Fabric CLI](service-fabric-application-lifecycle-sfctl.md) (Управление приложением Azure Service Fabric с помощью интерфейса командной строки Azure Service Fabric)
* [Подготовка среды разработки Linux в Windows](service-fabric-local-linux-cluster-windows.md)

<!-- Images -->
[cluster-setup-script]: ./media/service-fabric-get-started-mac/cluster-setup-mac.png
[sfx-mac]: ./media/service-fabric-get-started-mac/sfx-mac.png
[sf-eclipse-plugin-install]: ./media/service-fabric-get-started-mac/sf-eclipse-plugin-install.png
[buildship-update]: https://projects.eclipse.org/projects/tools.buildship
