---
title: Установка изолированного клиента Service Fabric
description: Из этого учебника вы узнаете, как установить в кластере автономный клиент Service Fabric.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc
ms.openlocfilehash: ae0b343be986f4d8d5176c1f39eef6b23ca81278
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91840648"
---
# <a name="tutorial-install-and-create-service-fabric-cluster"></a>Руководство по установке и созданию кластера Service Fabric

Изолированные кластеры Service Fabric предоставляют возможность выбрать собственную среду и создать кластер в рамках подхода "любая ОС, любое облако", который применяется на платформе Service Fabric. Из этой серии учебников вы узнаете, как создать изолированный кластер, размещенный в AWS или Azure, и установить в нем приложение.

Это руководство представляет собой вторую часть цикла. В этом руководстве описывается процесс создания изолированного кластера Service Fabric.

В этой статье вы узнаете, как выполнять следующие задачи.

> [!div class="checklist"]
> * скачивание изолированного пакета Service Fabric и его установка;
> * запуск кластера Service Fabric;
> * подключение к кластеру Service Fabric.

## <a name="download-the-service-fabric-for-windows-server-package"></a>Скачивание Service Fabric для пакета Windows Server

Service Fabric предоставляет установочный пакет для создания изолированных кластеров Service Fabric.  [Скачайте пакет установки](https://go.microsoft.com/fwlink/?LinkId=730690) на локальный компьютер.  Когда он будет успешно скачан, скопируйте его по RDP-подключению на свою виртуальную машину и вставьте его на рабочий стол.

Выберите ZIP-файл, откройте контекстное меню и выберите **Извлечь все** > **Извлечь**.  После извлечения файлов на рабочем столе будет создана папка с таким же именем, как у ZIP-файла.

Дополнительные сведения о содержимом пакета установки см. в [этой статье](service-fabric-cluster-standalone-package-contents.md).

## <a name="set-up-your-configuration-file"></a>Настройка файла конфигурации

Вы создаете кластер Windows с тремя узлами, поэтому вам нужно изменить файл `ClusterConfig.Unsecure.MultiMachine.json`.

Затем внесите в три строки ipAddress, размещенные в файле в строках 8, 15 и 22, IP-адреса каждого из экземпляров.

После обновления узлов они отображаются следующим образом:

```json
        {
            "nodeName": "vm0",
            "ipAddress": "172.31.27.1",
            "nodeTypeRef": "NodeType0",
            "faultDomain": "fd:/dc1/r0",
            "upgradeDomain": "UD0"
        }
```

Затем необходимо обновить несколько свойств.  В строке 34 вам необходимо изменить строку подключения к хранилищу диагностики. Она должна выглядеть следующим образом: `"connectionstring": "C:\\ProgramData\\SF\\DiagnosticsStore"`

Наконец, в разделе `nodeTypes` файла конфигурации добавьте новый раздел для сопоставления временных портов, которые будет использовать Windows.  Файл конфигурации должен выглядеть примерно следующим образом:

```json
"applicationPorts": {
    "startPort": "20001",
    "endPort": "20031"
},
"ephemeralPorts": {
    "startPort": "20606",
    "endPort": "20861"
},
"isPrimary": true
```

## <a name="validate-the-environment"></a>Проверка среды

Скрипт *TestConfiguration.ps1* в изолированном пакете используется как рекомендуемый анализатор, проверяющий возможность развертывания кластера в конкретной среде. [Здесь](service-fabric-cluster-standalone-deployment-preparation.md) перечислены предварительные требования к среде при подготовке к развертыванию. Выполните сценарий чтобы проверить, можно ли создать кластер разработки:

```powershell
cd .\Desktop\Microsoft.Azure.ServiceFabric.WindowsServer.6.2.274.9494\
.\TestConfiguration.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.MultiMachine.json
```

Вы увидите результат, похожий на этот пример. Если для нижнего поля "Passed" (Выполнено) возвращено значение `True`, значит, проверка работоспособности пройдена и кластер готов к развертыванию на основе входной конфигурации.

```powershell
Trace folder already exists. Traces will be written to existing trace folder: C:\Users\Administrator\Desktop\Microsoft.Azure.ServiceFabric.WindowsServer.6.2.274.9494\DeploymentTraces
Running Best Practices Analyzer...
Best Practices Analyzer completed successfully.


LocalAdminPrivilege        : True
IsJsonValid                : True
IsCabValid                 :
RequiredPortsOpen          : True
RemoteRegistryAvailable    : True
FirewallAvailable          : True
RpcCheckPassed             : True
NoConflictingInstallations : True
FabricInstallable          : True
DataDrivesAvailable        : True
NoDomainController         : True
Passed                     : True
```

## <a name="create-the-cluster"></a>Создайте кластер.

После успешной проверки конфигурации кластера выполните скрипт *CreateServiceFabricCluster.ps1*, чтобы развернуть кластер Service Fabric на виртуальные машины, указанные в файле конфигурации.

```powershell
.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.MultiMachine.json -AcceptEULA
```

Если все это сработает, вы получите результат, который выглядит следующим образом:

```powershell
Your cluster is successfully created! You can connect and manage your cluster using Microsoft Azure Service Fabric Explorer or PowerShell. To connect through PowerShell, run 'Connect-ServiceFabricCluster [ClusterConnectionEndpoint]'.
```

> [!NOTE]
> Трассировки развертывания записываются на компьютер или виртуальную машину, на которой запущен сценарий PowerShell CreateServiceFabricCluster.ps1. Их можно найти во вложенной папке DeploymentTraces в каталоге, в котором был запущен сценарий. Чтобы узнать, правильно ли развернут кластер Service Fabric на компьютере, найдите установленные файлы в каталоге FabricDataRoot, как описано в разделе FabricSettings файла конфигурации кластера (по умолчанию он находится в C:\ProgramData\SF). Кроме того, процессы FabricHost.exe и Fabric.exe должны отображаться в диспетчере задач.
>
>

### <a name="open-service-fabric-explorer"></a>Открытие Service Fabric Explorer

Теперь к кластеру можно подключиться при помощи Service Fabric Explorer непосредственно с одного из компьютеров по адресу http:\//localhost:19080/Explorer/index.html или удаленно по адресу http:\//<*IPAddressofaMachine*>:19080/Explorer/index.html.

## <a name="add-and-remove-nodes"></a>Добавление и удаление узлов

Вы можете добавить узлы в изолированный кластер Service Fabric или удалить их из него в соответствии с изменениями потребностей компании. Подробные инструкции статье [Добавление узлов в автономный кластер Service Fabric или удаление узлов из него](service-fabric-cluster-windows-server-add-remove-nodes.md) .

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали о процессе передачи больших объемов произвольных данных в учетную запись хранения в параллельном режиме, который включает такие задачи:

> [!div class="checklist"]
> * Настройка строки подключения
> * создание приложения;
> * Выполнение приложения
> * Проверка количества подключений.

Перейдите к третьей части цикла, чтобы установить приложение в созданный кластер.

> [!div class="nextstepaction"]
> [Руководство по развертыванию приложения в изолированном кластере Service Fabric](service-fabric-tutorial-standalone-install-an-application.md)

<!--Image references-->
[Trusted Zone]: ./media/service-fabric-cluster-creation-for-windows-server/TrustedZone.png
