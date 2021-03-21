---
title: Настройка среды разработки Windows для Service Fabricной сетки
description: Настройте среду разработки Windows таким образом, чтобы можно было создавать приложение Сетки Service Fabric и разворачивать его в Сетке Azure Service Fabric.
author: georgewallace
ms.author: gwallace
ms.date: 12/12/2018
ms.topic: conceptual
ms.openlocfilehash: fc234f6954cf263423cc517bb3dda2ba2efa3358
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99625741"
---
# <a name="set-up-your-windows-development-environment-to-build-service-fabric-mesh-apps"></a>Чтобы создавать приложения Сетки Service Fabric, настройте среду разработки Windows

> [!IMPORTANT]
> Поддержка предварительной версии Сетки Azure Service Fabric была прекращена. Новые развертывания больше не будут разрешены через API Сетки Service Fabric. Поддержка существующих развертываний будет продолжена до 28 апреля 2021 г. включительно.
> 
> Дополнительные сведения см. в статье [Прекращение поддержки предварительной версии Сетки Azure Service Fabric](https://azure.microsoft.com/updates/azure-service-fabric-mesh-preview-retirement/).

Чтобы создавать и запускать приложения Сетки Azure Service Fabric на компьютере для разработки Windows, вам потребуется:

* Docker
* Visual Studio 2017 или более поздней версии.
* Среда выполнения Сетки Service Fabric.
* Пакет SDK и инструменты для Сетки Service Fabric.

А также одна из следующих версий Windows:

* Windows 10 (Корпоративная, Профессиональная или для образовательных учреждений) версии 1709 (с обновлением Fall Creators Update) или 1803 (с обновлением Windows 10 за апрель 2018 г.).
* Windows Server, версия 1709
* Windows Server, версия 1803

Приведенные ниже инструкции помогут вам установить все компоненты, основываясь на используемой версии Windows.

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="visual-studio"></a>Visual Studio

Для развертывания Service Fabric приложений сетки требуется Visual Studio 2017 или более поздней версии. [Его версия должна быть не ниже 15.6.0][download-visual-studio] и включать следующие рабочие нагрузки.

* ASP.NET и веб-разработка.
* разработка Azure.

## <a name="install-docker"></a>Установка Docker

Если у вас уже есть установленный Docker, убедитесь, что используете последнюю версию. При истечении новой версии DOCKER может запросить новую версию, но вручную проверить, установлена ли последняя версия.

#### <a name="install-docker-on-windows-10"></a>Установка Docker для Windows 10

Скачайте и установите последнюю версию [Docker Community Edition для Windows][download-docker] для поддержки контейнерных приложений Service Fabric, используемого Сеткой Service Fabric.

Во время процесса установки при появлении запроса выберите **Использовать контейнеры Windows вместо контейнеров Linux**.

Если Hyper-V не включен на компьютере, установщик DOCKER предложит включить его. Если будет предложено, щелкните **OK**.

#### <a name="install-docker-on-windows-server-2016"></a>Установите Docker для Windows Server 2016

Если у вас не включена роль Hyper-V, откройте PowerShell как администратор и выполните следующую команду, а затем перезагрузите компьютер. Дополнительные сведения см. в статье [Install Docker Enterprise Edition for Windows Server][download-docker-server] (Установка Docker Enterprise Edition для Windows).

```powershell
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools
```

Перезапустите компьютер.

Чтобы установить Docker, запустите PowerShell как администратор и выполните следующую команду.

```powershell
Install-Module DockerMsftProvider -Force
Install-Package Docker -ProviderName DockerMsftProvider -Force
Install-WindowsFeature Containers
```

## <a name="sdk-and-tools"></a>Пакет SDK и средства

Установите Сетку Service Fabric, среду выполнения, пакет SDK и средства в следующем порядке.

1. Установите [пакет SDK Сетки Service Fabric][download-sdkmesh] с помощью установщика веб-платформы. Это позволит установить пакет SDK, средства Microsoft Azure Service Fabric и среду выполнения.
2. Установите [расширение средств службы "Сетка Service Fabric" в Visual Studio (предварительная версия)][download-tools] из Visual Studio Marketplace.

## <a name="build-a-cluster"></a>Создание кластера

> [!IMPORTANT]
> Перед созданием кластера **необходимо** запустить Docker.
> Проверьте, работает ли Docker, открыв окно терминала и выполнив команду `docker ps`, которая укажет, возникает ли ошибка. Если полученный ответ не является ошибкой, это значит что Docker работает и можно начинать создание кластера.

> [!Note]
> При разработке на компьютере с обновлением Windows Fall Creators Update (версия 1709) можно использовать только образы Docker для Windows версии 1709.
> При разработке на компьютере с обновлением Windows 10 за апрель 2018 г. (версия 1803) можно использовать образы docker как версии 1709, так и версии 1803.

Если вы используете Visual Studio, этот раздел можно пропустить, так как Visual Studio создаст локальный кластер, если у вас его нет.

Для оптимальной производительности при отладке при создании и запуске одного Service Fabric приложения за раз создайте кластер локальной разработки с одним узлом. Если вы используете несколько приложений за один раз, создайте кластер локальной разработки с пятью узлами. Этот кластер должен быть запущен при развертывании или при отладке проекта Сетки Service Fabric.

После установки среды выполнения, пакета SDK, средства Visual Studio, Docker и запуска Docker создайте кластер разработки.

1. Закройте окно PowerShell.
2. Откройте окно PowerShell с повышенными правами от имени администратора. Этот шаг необходим для загрузки модулей Service Fabric, которые были недавно установлены.
3. Чтобы создать кластер разработки, выполните следующую команду PowerShell.

    ```powershell
    . "C:\Program Files\Microsoft SDKs\Service Fabric\ClusterSetup\DevClusterSetup.ps1" -CreateMeshCluster -CreateOneNodeCluster
    ```
4. Чтобы запустить локальное средство управления кластером, выполните следующую команду PowerShell.

    ```powershell
    . "C:\Program Files\Microsoft SDKs\Service Fabric\Tools\ServiceFabricLocalClusterManager\ServiceFabricLocalClusterManager.exe"
    ```
5. Когда средство управления кластером запущено (отображается на панели задач), щелкните его правой кнопкой мыши и выберите **Start Local Cluster** (Запустить локальный кластер).

![Рис. 1. Запуск локального кластера](./media/service-fabric-mesh-howto-setup-developer-environment-sdk/start-local-cluster.png)

Теперь можно приступать к созданию приложений Сетки Service Fabric.

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь с руководством [Tutorial: Create, debug, and deploy a multi-service web application to Service Fabric Mesh](service-fabric-mesh-tutorial-create-dotnetcore.md) (Руководство. Создание, отладка и развертывание многосервисного веб-приложения в Сетке Service Fabric).

Найдите ответы на [часто задаваемые вопросы и способы решения известных проблем](service-fabric-mesh-faq.md).

[azure-cli-install]: /cli/azure/install-azure-cli
[download-docker]: https://store.docker.com/editions/community/docker-ce-desktop-windows
[download-docker-server]: https://docs.docker.com/install/windows/docker-ee/
[download-runtime]: https://aka.ms/sfruntime
[download-sdk]: https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-CoreSDK
[download-sdkmesh]: https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-SDK-Mesh
[download-tools]: https://aka.ms/sfmesh_vs2017tools
[download-visual-studio]: https://www.visualstudio.com/downloads/
