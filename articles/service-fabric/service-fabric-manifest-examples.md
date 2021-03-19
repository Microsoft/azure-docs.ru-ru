---
title: Примеры манифеста приложения Azure Service Fabric
description: Узнайте, как настроить параметры манифестов приложений и служб для приложения Service Fabric.
author: peterpogorski
ms.topic: conceptual
ms.date: 06/11/2018
ms.author: pepogors
ms.openlocfilehash: 6f44d2ddaac11545a3a3dc52fd20e02263c6ad67
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "75451644"
---
# <a name="service-fabric-application-and-service-manifest-examples"></a>Примеры манифестов приложений и служб Service Fabric
В этом разделе содержатся примеры манифестов приложений и служб. Эти примеры предназначены не для того, чтобы продемонстрировать важные сценарии, а чтобы показать различные доступные параметры и способы их использования. 

Ниже приведен список возможностей и примеры манифестов, частью которых они являются.

|Компонент|манифеста|
|---|---|
|[Управление ресурсами](service-fabric-resource-governance.md)|[Манифест приложения Reliable Services](service-fabric-manifest-example-reliable-services-app.md#application-manifest), [манифест приложения контейнера](service-fabric-manifest-example-container-app.md#application-manifest)|
|[Запуск службы с использованием учетной записи локального администратора](service-fabric-application-runas-security.md)|[Манифест приложения Reliable Services](service-fabric-manifest-example-reliable-services-app.md#application-manifest)|
|[Применение политики по умолчанию ко всем пакетам кода службы](service-fabric-application-runas-security.md#apply-a-default-policy-to-all-service-code-packages)|[Манифест приложения Reliable Services](service-fabric-manifest-example-reliable-services-app.md#application-manifest)|
|[Создание субъектов пользователей и групп](service-fabric-application-runas-security.md)|[Манифест приложения Reliable Services](service-fabric-manifest-example-reliable-services-app.md#application-manifest)|
|совместное использование пакета данных экземплярами служб|[Манифест приложения Reliable Services](service-fabric-manifest-example-reliable-services-app.md#application-manifest)|
|[Переопределение конечных точек службы](service-fabric-service-manifest-resources.md#overriding-endpoints-in-servicemanifestxml)|[Манифест приложения Reliable Services](service-fabric-manifest-example-reliable-services-app.md#application-manifest)|
|[Выполнение сценария при запуске службы](service-fabric-run-script-at-service-startup.md)|[Манифест службы VotingWeb](service-fabric-manifest-example-reliable-services-app.md#votingweb-service-manifest)|
|[Определение конечной точки HTTPS](service-fabric-tutorial-dotnet-app-enable-https-endpoint.md#define-an-https-endpoint-in-the-service-manifest)|[Манифест службы VotingWeb](service-fabric-manifest-example-reliable-services-app.md#votingweb-service-manifest)|
|[Объявление пакета конфигурации](service-fabric-application-and-service-manifests.md)|[Манифест службы VotingData](service-fabric-manifest-example-reliable-services-app.md#votingdata-service-manifest)|
|[Объявление пакета данных](service-fabric-application-and-service-manifests.md)|[Манифест службы VotingData](service-fabric-manifest-example-reliable-services-app.md#votingdata-service-manifest)|
|[Переопределение переменных среды](service-fabric-get-started-containers.md#configure-and-set-environment-variables)|[Манифест приложения контейнера](service-fabric-manifest-example-container-app.md#application-manifest)|
|[Настройка сопоставления порта контейнера с узлом](service-fabric-get-started-containers.md#configure-container-port-to-host-port-mapping-and-container-to-container-discovery)| [Манифест приложения контейнера](service-fabric-manifest-example-container-app.md#application-manifest)|
|[Настройка проверки подлинности в реестре контейнеров](service-fabric-get-started-containers.md#configure-container-repository-authentication)|[Манифест приложения контейнера](service-fabric-manifest-example-container-app.md#application-manifest)|
|[Настройка режима изоляции](service-fabric-get-started-containers.md#configure-isolation-mode)|[Манифест приложения контейнера](service-fabric-manifest-example-container-app.md#application-manifest)|
|[Указание образов контейнеров для конкретной сборки ОС](service-fabric-get-started-containers.md#specify-os-build-specific-container-images)|[Манифест приложения контейнера](service-fabric-manifest-example-container-app.md#application-manifest)|
|[Настройка переменных среды](service-fabric-get-started-containers.md#configure-and-set-environment-variables)|[Манифест службы FrontEndService контейнера](service-fabric-manifest-example-container-app.md#frontendservice-service-manifest), [манифест службы BackEndService контейнера](service-fabric-manifest-example-container-app.md#backendservice-service-manifest)|
|[Настройка конечной точки](service-fabric-get-started-containers.md#configure-communication)|[Манифест службы FrontEndService контейнера](service-fabric-manifest-example-container-app.md#frontendservice-service-manifest), [манифест службы BackEndService контейнера](service-fabric-manifest-example-container-app.md#backendservice-service-manifest), [манифест службы VotingData](service-fabric-manifest-example-reliable-services-app.md#votingdata-service-manifest)|
|передача команд в контейнер|[Манифест службы FrontEndService контейнера](service-fabric-manifest-example-container-app.md#frontendservice-service-manifest)|
|[Импорт сертификата в контейнер](service-fabric-securing-containers.md)|[Манифест службы FrontEndService контейнера](service-fabric-manifest-example-container-app.md#frontendservice-service-manifest)|
|[Настройка драйвера тома](service-fabric-containers-volume-logging-drivers.md)|[Манифест службы BackEndService контейнера](service-fabric-manifest-example-container-app.md#backendservice-service-manifest)|

