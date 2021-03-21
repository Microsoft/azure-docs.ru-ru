---
title: Справочник по Maven сети Azure Service Fabric
description: Справочная информация по использованию подключаемого модуля Maven для службы "Сетка Service Fabric"
author: suhuruli
ms.author: suhuruli
ms.date: 11/26/2018
ms.topic: reference
ms.custom: devx-track-java
ms.openlocfilehash: dd42752dc7ef2c7686073b7a2d780fe9356425c7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99625589"
---
# <a name="maven-plugin-for-service-fabric-mesh"></a>Подключаемый модуль Maven для службы "Сетка Service Fabric"

> [!IMPORTANT]
> Поддержка предварительной версии Сетки Azure Service Fabric была прекращена. Новые развертывания больше не будут разрешены через API Сетки Service Fabric. Поддержка существующих развертываний будет продолжена до 28 апреля 2021 г. включительно.
> 
> Дополнительные сведения см. в статье [Прекращение поддержки предварительной версии Сетки Azure Service Fabric](https://azure.microsoft.com/updates/azure-service-fabric-mesh-preview-retirement/).

## <a name="prerequisites"></a>Предварительные условия

- Пакет SDK для Java
- Maven
- Azure CLI с расширением сетки
- Интерфейс командной строки Service Fabric

## <a name="goals"></a>Цели

### `azure-sfmesh:init`
- Создает папку `servicefabric` с вложенной папкой `appresources`, которая содержит файл `application.yaml`. 

### `azure-sfmesh:addservice`
- Создает папку внутри папки `servicefabric` с именем службы и создает файл YAML для службы. 

### `azure-sfmesh:addnetwork`
- Создает YAML-файл `network` с указанным именем сети в папке `appresources`. 

### `azure-sfmesh:addgateway`
- Создает YAML-файл `gateway` с указанным именем шлюза в папке `appresources`. 

#### `azure-sfmesh:addvolume`
- Создает `volume` YAML с указанным именем тома в `appresources` папке.

### `azure-sfmesh:addsecret`
- Создает YAML-файл `secret` с указанным именем секрета в папке `appresources`. 

### `azure-sfmesh:addsecretvalue`
- Создает YAML-файл `secretvalue` с указанными именами секрета и секретного значения в папке `appresources`. 

### `azure-sfmesh:deploy`
- Объединяет YAML-файлы из папки `servicefabric` и создает JSON-файл с шаблоном Azure Resource Manager в текущей папке.
- Развертывает все ресурсы в среде "Сетка Azure Service Fabric". 

### `azure-sfmesh:deploytocluster`
- Создает папку (`meshDeploy`) с JSON-файлами развертывания, созданными на основе YAML-файлов, которые применяются для кластеров Service Fabric.
- Развертывает все ресурсы в кластере Service Fabric.
 

## <a name="usage"></a>Использование

Чтобы использовать подключаемый модуль Maven в приложении Maven Java, добавьте следующий фрагмент кода в файл pom.xml:

```XML
<project>
  ...
  <build>
    ...
    <plugins>
      ...
      <plugin>
        <groupId>com.microsoft.azure</groupId>
          <artifactId>azure-sfmesh-maven-plugin</artifactId>
          <version>0.1.0</version>
      </plugin>
    </plugins>
  </build>
</project>
```

## <a name="common-configuration"></a>Типичная настройка

Подключаемый модуль Maven сейчас не поддерживает типичные конфигурации подключаемых модулей Maven для Azure.

## <a name="how-to"></a>Инструкции

### <a name="initialize-maven-project-for-azure-service-fabric-mesh"></a>Инициализация проекта Maven для службы "Сетка Azure Service Fabric"
Выполните следующую команду, чтобы создать YAML-файл для ресурсов приложения.

```cmd
mvn azure-sfmesh:init -DapplicationName=helloworldserver
```

- Создает папку с именем `servicefabric->appresources` в корневой папке, которая содержит YAML-файл `app_helloworldserver` для приложения.

### <a name="add-resource-to-your-application"></a>Добавление ресурса в приложение

#### <a name="add-a-new-network-to-your-application"></a>Добавление новой сети в приложение
Выполните следующую команду, чтобы создать YAML-файл для сетевых ресурсов. 

```cmd
mvn azure-sfmesh:addnetwork -DnetworkName=helloworldservicenetwork -DnetworkAddressPrefix=10.0.0.0/22
```

- Создает в папке `servicefabric->appresources` YAML-файл `network_helloworldservicenetwork` для сети.

#### <a name="add-a-new-service-to-your-application"></a>Добавление новой службы в приложение
Выполните следующую команду, чтобы создать YAML-файл для службы. 

```cmd
mvn azure-sfmesh:addservice -DapplicationName=helloworldserver -DserviceName=helloworldservice -DimageName=helloworldserver:latest -DlistenerPort=8080 -DnetworkRef=helloworldservicenetwork
```

- Создает в папке `servicefabric->helloworldservice` YAML-файл для службы с именем `service_helloworldservice` и ссылками на `helloworldservicenetwork` и приложение `helloworldserver`.
- Служба прослушивает порт 8080.
- Служба использует образ контейнера ***helloworldserver:latest***.

#### <a name="add-a-new-gateway-resource-to-your-application"></a>Добавление ресурса шлюза в приложение
Выполните следующую команду, чтобы создать YAML-файл для ресурса шлюза. 

```cmd
mvn azure-sfmesh:addgateway -DapplicationName=helloworldserver -DdestinationNetwork=helloworldservicenetwork -DgatewayName=helloworldgateway -DlistenerName=helloworldserviceListener -DserviceName=helloworldservice -DsourceNetwork=open -DtcpPort=80
```

- Создает в папке `servicefabric->appresources` YAML-файл с именем `gateway_helloworldgateway` для нового шлюза.
- Указывает ссылку на `helloworldservicelistener` в качестве прослушивателя службы, который ожидает вызовы от этого шлюза. Также указывает ссылку на `helloworldservice` в качестве службы, `helloworldservicenetwork` в качестве сети и `helloworldserver` в качестве приложения. 
- Прослушивает запросы на порту 80.

#### <a name="add-a-new-volume-to-your-application"></a>Добавление нового тома в приложение
Выполните следующую команду, чтобы создать YAML-файл для ресурса тома. 

```cmd
mvn azure-sfmesh:addvolume -DvolumeAccountKey=key -DvolumeAccountName=name -DvolumeName=vol1 -DvolumeShareName=share
```

- Создает в папке `servicefabric->appresources` YAML-файл с именем `volume_vol1` для тома.
- Задает свойства обязательных параметров `volumeAccountKey` и `volumeShareName`, как указано выше.
- Дополнительные сведения об использовании ссылок на созданный том см. в статье [Сохранение состояния в приложении "Сетка Azure Service Fabric" путем подключения тома службы файлов Azure внутри контейнера](service-fabric-mesh-howto-deploy-app-azurefiles-volume.md).

#### <a name="add-a-new-secret-resource-to-your-application"></a>Добавление ресурса секрета в приложение
Выполните следующую команду, чтобы создать YAML-файл для ресурса секрета. 

```cmd
mvn azure-sfmesh:addsecret -DsecretName=secret1
```

- Создает в папке `servicefabric->appresources` YAML-файл `secret_secret1` для секрета.
- Дополнительные сведения об использовании ссылок на созданный секрет см. в статье [об управлении секретами](service-fabric-mesh-howto-manage-secrets.md).

#### <a name="add-a-new-secretvalue-resource-to-your-application"></a>Добавление ресурса секретного значения в приложение
Выполните следующую команду, чтобы создать YAML-файл для ресурса секретного значения. 

```cmd
mvn azure-sfmesh:addsecretvalue -DsecretValue=someVal -DsecretValueName=secret1/v1
```

- Создает в папке `servicefabric->appresources` YAML-файл `secretvalue_secret1_v1` для секретного значения.

### <a name="run-the-application-locally"></a>Локальный запуск приложения

С помощью цели `azure-sfmesh:deploytocluster` можно запустить приложение локально, выполнив следующую команду:

```cmd
mvn azure-sfmesh:deploytocluster
```

По умолчанию эта цель позволяет развернуть ресурсы в локальном кластере. Если развертывание выполняется в локальном кластере, предполагается, что вы настроили и запустили локальный кластер Service Fabric. Сейчас локальный кластер Service Fabric для ресурсов поддерживается только в операционной системе [Windows](service-fabric-mesh-howto-setup-developer-environment-sdk.md).

- Создает JSON-файлы из YAML-файлов, которые применяются для кластеров Service Fabric.
- Затем развертывает их в конечную точку кластера.

### <a name="deploy-application-to-azure-service-fabric-mesh"></a>Развертывание приложения в службу "Сетка Azure Service Fabric"

С помощью цели `azure-sfmesh:deploy` можно развернуть среду "Сетка Azure Service Fabric", выполнив следующую команду:

```cmd
mvn azure-sfmesh:deploy -DresourceGroup=todoapprg -Dlocation=eastus
```

- Создает группу ресурсов с именем `todoapprg`, если такая еще не существует.
- Создает JSON-файл с шаблоном Azure Resource Manager, объединив YAML-файлы. 
- Развертывает этот JSON-файл в среде "Сетка Azure Service Fabric".
