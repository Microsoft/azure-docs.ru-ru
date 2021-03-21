---
title: Настройка среды разработки в Linux
description: Установите среду выполнения и пакет SDK, а затем создайте локальный кластер разработки в Linux. По завершении установки вы сможете создавать приложения.
ms.topic: conceptual
ms.date: 10/16/2020
ms.custom: devx-track-js
ms.openlocfilehash: 14b8a278605a908b4182c724831b2e42de54a753
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93086896"
---
# <a name="prepare-your-development-environment-on-linux"></a>Подготовка среды разработки в Linux
> [!div class="op_single_selector"]
> * [Windows](service-fabric-get-started.md)
> * [Linux](service-fabric-get-started-linux.md)
> * [Mac OS X](service-fabric-get-started-mac.md)

Чтобы развертывать и запускать [приложения Azure Service Fabric](service-fabric-application-model.md) на компьютере для разработки под управлением Linux, установите среду выполнения и стандартный пакет SDK. Вы также можете установить дополнительные пакеты SDK для разработки Java и .NET Core. 

В этом разделе предполагается, что вы установили изначально в Linux или используете [образ контейнера Service Fabric OneBox](https://hub.docker.com/_/microsoft-service-fabric-onebox), `mcr.microsoft.com/service-fabric/onebox:u18` т. е.

Вы можете управлять Service Fabric сущностями, размещенными в облаке или локальной среде, с помощью интерфейса командной строки (CLI) Service Fabric Azure. Дополнительные сведения об установке интерфейса командной строки см. в [этой статье](./service-fabric-cli.md).


## <a name="prerequisites"></a>Предварительные условия

Для разработки поддерживаются следующие операционные системы.

* Ubuntu 16,04 ( `Xenial Xerus` ), 18,04 ( `Bionic Beaver` )

    Убедитесь, что пакет `apt-transport-https` установлен.
         
    ```bash
    sudo apt-get install apt-transport-https
    ```
* Red Hat Enterprise Linux 7.4 (поддержка предварительной версии Service Fabric)


## <a name="installation-methods"></a>Методы установки

<!-- markdownlint-disable MD025 -->
<!-- markdownlint-disable MD024 -->

# <a name="ubuntu"></a>[Ubuntu](#tab/sdksetupubuntu)

## <a name="update-your-apt-sources"></a>Обновление списка источников APT
Прежде чем перейти к установке пакета SDK и связанного пакета среды выполнения с помощью средства командной строки apt-get, необходимо обновить список источников APT.

## <a name="script-installation"></a>Установка с помощью скрипта

Для удобства предоставляется сценарий для установки среды выполнения Service Fabric и Service Fabric общего пакета SDK вместе с интерфейсом [командной строки **sfctl**](service-fabric-cli.md). При запуске скрипта предполагается, что вы принимаете условия использования лицензий на все устанавливаемое программное обеспечение. Кроме того, вы можете выполнить действия по [установке вручную](#manual-installation) в следующем разделе, в котором будут представлены связанные лицензии, а также устанавливаемые компоненты.

После успешного выполнения скрипта можно сразу перейти к разделу [Настройка локального кластера](#set-up-a-local-cluster).

```bash
sudo curl -s https://raw.githubusercontent.com/Azure/service-fabric-scripts-and-templates/master/scripts/SetupServiceFabric/SetupServiceFabric.sh | sudo bash
```

## <a name="manual-installation"></a>Установка вручную
Чтобы установить среду выполнения Service Fabric и общий пакет SDK вручную, ознакомьтесь с остальными разделами руководства.

1. Откройте окно терминала.

2. Добавьте `dotnet` репозиторий в список источников, соответствующий дистрибутиву.

    ```bash
    wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb
    sudo dpkg -i packages-microsoft-prod.deb
    ```

3. Добавьте новый ключ защиты конфиденциальности Microsoft Open Tech GNU (GnuPG или GPG) в набор ключей APT.

    ```bash
    sudo curl -fsSL https://packages.microsoft.com/keys/msopentech.asc | sudo apt-key add -
    ```

4. Добавьте официальный ключ Docker GPG в набор ключей APT.

    ```bash
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```

5. Настройте репозиторий Docker.

    ```bash
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    ```

6. Добавьте ключ JDK Azul в набор ключей APT и настройте его репозиторий.

    ```bash
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 0xB1998361219BD9C9
    sudo apt-add-repository "deb http://repos.azul.com/azure-only/zulu/apt stable main"
    ```

7. Обновите списки пакетов, добавив в них новые репозитории.

    ```bash
    sudo apt-get update
    ```

## <a name="install-and-set-up-the-service-fabric-sdk-for-a-local-cluster"></a>Установка и настройка пакета SDK для Service Fabric для локального кластера

После обновления источников пакетов можно установить пакет SDK. Установите пакет SDK для Service Fabric, подтвердите установку и примите условия лицензионного соглашения.

### <a name="ubuntu"></a>Ubuntu

```bash
sudo apt-get install servicefabricsdkcommon
```

> [!TIP]
>   Следующие команды автоматизируют принятие лицензии для пакетов Service Fabric:
>   ```bash
>   echo "servicefabric servicefabric/accepted-eula-ga select true" | sudo debconf-set-selections
>   echo "servicefabricsdkcommon servicefabricsdkcommon/accepted-eula-ga select true" | sudo debconf-set-selections
>   ```

# <a name="red-hat-enterprise-linux-74"></a>[Red Hat Enterprise Linux 7.4](#tab/sdksetuprhel74)

## <a name="update-your-yum-repositories"></a>Обновление репозиториев Yum
Чтобы установить пакет SDK и связанный пакет среды выполнения с помощью программы командной строки yum, необходимо сначала обновить источники пакетов.

## <a name="manual-installation-rhel"></a>Установка вручную (RHEL)
Чтобы установить среду выполнения Service Fabric и общий пакет SDK вручную, ознакомьтесь с остальными разделами руководства.

1. Откройте окно терминала.
2. Скачайте и установите дополнительные пакеты для Enterprise Linux (EPEL).

    ```bash
    wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    sudo yum install epel-release-latest-7.noarch.rpm
    ```
3. Добавьте пакет репозитория EfficiOS RHEL7 в систему.

    ```bash
    sudo wget -P /etc/yum.repos.d/ https://packages.efficios.com/repo.files/EfficiOS-RHEL7-x86-64.repo
    ```

4. Импортируйте ключ подписывания пакета EfficiOS в локальный набор ключей GPG.

    ```bash
    sudo rpmkeys --import https://packages.efficios.com/rhel/repo.key
    ```

5. Добавьте репозиторий Microsoft RHEL в систему.

    ```bash
    curl https://packages.microsoft.com/config/rhel/7.4/prod.repo > ./microsoft-prod.repo
    sudo cp ./microsoft-prod.repo /etc/yum.repos.d/
    ```

## <a name="install-and-set-up-the-service-fabric-sdk-for-a-local-cluster-rhel"></a>Установка и настройка пакета SDK Service Fabric для локального кластера (RHEL)

После обновления источников пакетов можно установить пакет SDK. Установите пакет SDK для Service Fabric, подтвердите установку и примите условия лицензионного соглашения.

```bash
sudo yum install servicefabricsdkcommon
```

---

## <a name="included-packages"></a>Добавленные пакеты
Среда выполнения Service Fabric, которая поставляется с установкой пакета SDK, содержит пакеты, перечисленные в следующей таблице. 

 | | DotNetCore | Java | Python | Node.js | 
--- | --- | --- | --- |---
**Ubuntu** | 2.0.7 | AzulJDK 1.8 | Неявно с использованием npm | последняя |
**RHEL** | - | OpenJDK 1.8 | Неявно с использованием npm | последняя |

## <a name="set-up-a-local-cluster"></a>Настройка локального кластера
1. Запустите локальный кластер Service Fabric для разработки.

# <a name="container-based-local-cluster"></a>[Локальный кластер на основе контейнера](#tab/localclusteroneboxcontainer)

Запустите кластер [Service Fabric OneBox](https://hub.docker.com/_/microsoft-service-fabric-onebox) на основе контейнера.

1. Установите значок Кита, чтобы иметь возможность развертывать контейнеры DOCKER.
    ```bash
    sudo apt-get install moby-engine moby-cli -y
    ```
2. Обновите конфигурацию управляющей программы DOCKER на узле со следующими параметрами и перезапустите управляющую программу DOCKER. Сведения: [Включение поддержки IPv6](https://docs.docker.com/config/daemon/ipv6/)

    ```json
    {
        "ipv6": true,
        "fixed-cidr-v6": "fd00::/64"
    }
    ```

3. Запустите кластер.<br/>
    <b>Ubuntu 18,04 LTS:</b>
    ```bash
    docker run --name sftestcluster -d -v /var/run/docker.sock:/var/run/docker.sock -p 19080:19080 -p 19000:19000 -p 25100-25200:25100-25200 mcr.microsoft.com/service-fabric/onebox:u18
    ```

    <b>Ubuntu 16,04 LTS:</b>
    ```bash
    docker run --name sftestcluster -d -v /var/run/docker.sock:/var/run/docker.sock -p 19080:19080 -p 19000:19000 -p 25100-25200:25100-25200 mcr.microsoft.com/service-fabric/onebox:u16
    ```

    >[!TIP]
    > По умолчанию вы получите образ с последней версией Service Fabric. Конкретные редакции см. на странице [центра DOCKER](https://hub.docker.com/r/microsoft/service-fabric-onebox/) .

# <a name="local-cluster"></a>[Локальный кластер](#tab/localcluster)

После установки пакета SDK с помощью описанных выше действий запустите локальный кластер.

1. Выполните сценарий настройки кластера.

    ```bash
    sudo /opt/microsoft/sdk/servicefabric/common/clustersetup/devclustersetup.sh
    ```

---

2. Откройте веб-браузер и перейдите в **Service Fabric Explorer** ( `http://localhost:19080/Explorer` ). Когда кластер запустится, вы увидите панель мониторинга Service Fabric Explorer. Полная настройка кластера может занять несколько минут. Если ваш браузер не может открыть URL-адрес или Service Fabric Explorer указывает на неготовность системы, подождите несколько минут и повторите попытку.

    ![Обозреватель Service Fabric Explorer в Linux][sfx-linux]

    Теперь можно развертывать готовые пакеты приложений Service Fabric или новые приложения на базе гостевых контейнеров или гостевых исполняемых файлов. Чтобы создать службы с помощью пакетов SDK для Java или .NET Core, выполните этапы установки, указанные в следующих разделах.


> [!NOTE]
> Автономные кластеры не поддерживаются в Linux.


> [!TIP]
> При наличии доступного диска SSD мы рекомендуем передать путь к папке SSD с помощью `--clusterdataroot` и devclustersetup.sh для более высокой производительности.

## <a name="set-up-the-service-fabric-cli"></a>Настройка интерфейса командной строки Service Fabric

[Интерфейс командной строки Service Fabric](service-fabric-cli.md) поддерживает команды для взаимодействия с сущностями Service Fabric, включая кластеры и приложения. Чтобы установить CLI, см. инструкции по использованию [Service Fabric CLI](service-fabric-cli.md).


## <a name="set-up-yeoman-generators-for-containers-and-guest-executables"></a>Настройка генераторов Yeoman для контейнеров и гостевых исполняемых файлов
Service Fabric предоставляет средства формирования шаблонов для создания приложений Service Fabric из терминала с помощью генератора шаблонов Yeoman. Чтобы настроить генераторы шаблонов Yeoman Service Fabric, сделайте следующее:

1. Установите Node.js и NPM на компьютере.

    ```bash
    sudo add-apt-repository "deb https://deb.nodesource.com/node_8.x $(lsb_release -s -c) main"
    sudo apt-get update
    sudo apt-get install nodejs
    ```
2. Установите на компьютере генератор шаблонов [Yeoman](https://yeoman.io/) из NPM.

    ```bash
    sudo npm install -g yo
    ```
3. Установите генератор контейнеров Yeo для Service Fabric и генератор гостевых исполняемых файлов из NPM.

    ```bash
    sudo npm install -g generator-azuresfcontainer  # for Service Fabric container application
    sudo npm install -g generator-azuresfguest      # for Service Fabric guest executable application
    ```

Установив генераторы, создайте службы гостевых исполняемых файлов или службы контейнеров с помощью команды `yo azuresfguest` или `yo azuresfcontainer` соответственно.

## <a name="set-up-net-core-31-development"></a>Настройка разработки .NET Core 3,1

Установите [пакет SDK для .NET Core 3,1 для Ubuntu](https://www.microsoft.com/net/core#linuxubuntu) , чтобы начать [Создание приложений Service Fabric C#](service-fabric-create-your-first-linux-application-with-csharp.md). Пакеты для Service Fabric приложений .NET Core размещаются в NuGet.org.

## <a name="set-up-java-development"></a>Настройка разработки Java

Для создания служб Service Fabric с помощью Java установите средство Gradle, чтобы выполнять задачи сборки. Выполните следующую команду, чтобы установить Gradle. Библиотеки Java для Service Fabric извлекаются из Maven.


* Ubuntu

    ```bash
    curl -s https://get.sdkman.io | bash
    sdk install gradle 5.1
    ```

* Red Hat Enterprise Linux 7.4 (поддержка предварительной версии Service Fabric)

  ```bash
  sudo yum install java-1.8.0-openjdk-devel
  curl -s https://get.sdkman.io | bash
  sdk install gradle
  ```

Также необходимо установить генератор Yeo Service Fabric для исполняемых файлов Java. Убедитесь, что у вас установлен [Yeoman](#set-up-yeoman-generators-for-containers-and-guest-executables), а затем запустите следующую команду:

  ```bash
  npm install -g generator-azuresfjava
  ```
 
## <a name="install-the-eclipse-plug-in-optional"></a>Установка подключаемого модуля Eclipse (необязательно)

Подключаемый модуль Eclipse для Service Fabric можно установить из интегрированной среды разработки Eclipse для разработчиков Java или Java EE. Eclipse можно использовать для создания приложений с гостевыми исполняемыми файлами и контейнерами Service Fabric в дополнение к Java-приложениям Service Fabric.

> [!IMPORTANT]
> Для подключаемого модуля Service Fabric требуется Eclipse Neon или более поздняя версия. Чтобы проверить версию Eclipse, выполните инструкции под этим примечанием. Если у вас установлена более ранняя версия Eclipse, скачайте более новые версии с [сайта Eclipse](https://www.eclipse.org). Мы рекомендуем не устанавливать Eclipse поверх существующей установки (перезаписывать). Либо удалите ее перед запуском установщика, либо установите новую версию в другом каталоге.
> 
> В Ubuntu мы рекомендуем установить Eclipse непосредственно с сайта, а не с помощью установщика пакета (`apt` или `apt-get`). Так вы получите последнюю версию Eclipse. Можно установить среду разработки Eclipse для разработчиков Java или Java EE.

1. Откройте Eclipse и проверьте, установлена ли у вас версия Eclipse Neon или выше и последняя версия Buildship (2.2.1 или выше). Проверьте версии установленных компонентов, выбрав **Справка**  >  **о**  >  **сведениях об установке** Eclipse. Чтобы обновить Buildship, воспользуйтесь инструкциями на странице [Eclipse Buildship: Eclipse Plug-ins for Gradle][buildship-update] (Eclipse Buildship. Подключаемые модули Eclipse для Gradle).

2. Чтобы установить подключаемый модуль Service Fabric, выберите **Справка**  >  **установить новое по**.

3. В поле **работать с** введите **https: \/ /DL.Microsoft.com/Eclipse**.

4. Выберите **Добавить**.

    ![Страница с доступным программным обеспечением][sf-eclipse-plugin]

5. Выберите подключаемый модуль **ServiceFabric** и нажмите кнопку **Next** (Далее).

6. Выполните действия по установке. Потом примите условия лицензионного соглашения.

Если подключаемый модуль Eclipse Service Fabric уже установлен, убедитесь, что вы используете последнюю версию. Выберите пункт **Справка**  >  **о**  >  **сведениях об установке** Eclipse. Затем выполните поиск Service Fabric в списке установленных подключаемых модулей. Выберите **Обновить** , если доступна более новая версия.

Дополнительные сведения см. в статье [Подключаемый модуль Service Fabric для разработки приложений Eclipse на Java](service-fabric-get-started-eclipse.md).

## <a name="update-the-sdk-and-runtime"></a>Обновление пакета SDK и среды выполнения

Для установки последней версии пакета SDK и среды выполнения используйте следующие команды.

```bash
sudo apt-get update
sudo apt-get install servicefabric servicefabricsdkcommon
```
Чтобы обновить двоичные файлы пакета SDK для Java из Maven, необходимо изменить сведения о версии соответствующего двоичного файла в файле ``build.gradle``, чтобы они указывали на последнюю версию. Чтобы узнать, где необходимо обновить версию, просмотрите любой файл ``build.gradle`` в [примерах по началу работы с Service Fabric](https://github.com/Azure-Samples/service-fabric-java-getting-started).

> [!NOTE]
> Обновление пакетов может привести к остановке локального кластера разработки. Перезапустите локальный кластер после обновления, следуя инструкциям в этой статье.

## <a name="remove-the-sdk"></a>Удаление пакета SDK
Чтобы удалить пакеты SDK для Service Fabric, выполните следующие команды.

* Ubuntu

    ```bash
    sudo apt-get remove servicefabric servicefabicsdkcommon
    npm uninstall -g generator-azuresfcontainer
    npm uninstall -g generator-azuresfguest
    sudo apt-get install -f
    ```


* Red Hat Enterprise Linux 7.4 (поддержка предварительной версии Service Fabric)

    ```bash
    sudo yum remove servicefabric servicefabicsdkcommon
    npm uninstall -g generator-azuresfcontainer
    npm uninstall -g generator-azuresfguest
    ```

## <a name="next-steps"></a>Дальнейшие действия

* [Создание первого Java-приложения Service Fabric в Linux](service-fabric-create-your-first-linux-application-with-java.md)
* [Подключаемый модуль Service Fabric для разработки приложений Eclipse на Java](service-fabric-get-started-eclipse.md)
* [Создание первого приложения Azure Service Fabric](service-fabric-create-your-first-linux-application-with-csharp.md)
* [Настройка среды разработки для Mac OS X](service-fabric-get-started-mac.md)
* [Подготовка среды разработки Linux в Windows](service-fabric-local-linux-cluster-windows.md)
* [Управление приложением Azure Service Fabric с помощью интерфейса командной строки Azure Service Fabric](service-fabric-application-lifecycle-sfctl.md)
* [Различия между Service Fabric для Linux и для Windows](service-fabric-linux-windows-differences.md)
* [Azure Service Fabric command line](service-fabric-cli.md) (Интерфейс командной строки Azure Service Fabric)

<!-- Links -->

[azure-xplat-cli-github]: https://github.com/Azure/azure-xplat-cli
[install-node]: https://nodejs.org/en/download/package-manager/#installing-node-js-via-package-manager
[buildship-update]: https://projects.eclipse.org/projects/tools.buildship

<!--Images -->

[sf-eclipse-plugin]: ./media/service-fabric-get-started-linux/service-fabric-eclipse-plugin.png
[sfx-linux]: ./media/service-fabric-get-started-linux/sfx-linux.png
