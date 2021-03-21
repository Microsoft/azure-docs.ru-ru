---
title: Настройка среды разработки Windows
description: Установите среду выполнения, пакет SDK и инструменты и создайте локальный кластер разработки. После завершения установки вы сможете создавать приложения на базе Windows.
author: peterpogorski
ms.topic: conceptual
ms.date: 06/16/2020
ms.custom: sfrev, devx-track-azurepowershell
ms.openlocfilehash: 4568b791db07eaa7a513c42066b22df04b24cb6d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103010309"
---
# <a name="prepare-your-development-environment-on-windows"></a>Настройка среды разработки для Windows

> [!div class="op_single_selector"]
> * [Windows](service-fabric-get-started.md) 
> * [Linux](service-fabric-get-started-linux.md)
> * [Mac OS X](service-fabric-get-started-mac.md)
>
>

Чтобы создавать и запускать [приложения Service Fabric][1] на компьютере для разработки Windows, установите среду выполнения Service Fabric, пакет SDK и инструменты. Также необходимо [включить выполнение сценариев Windows PowerShell](#enable-powershell-script-execution) , включенных в пакет SDK.

## <a name="prerequisites"></a>Предварительные требования

### <a name="supported-operating-system-versions"></a>Поддерживаемые версии операционных систем

Для разработки поддерживаются следующие операционные системы:

* Windows 7
* Windows 8 и Windows 8.1;
* Windows Server 2012 R2
* Windows Server 2016
* Windows 10

> [!NOTE]
> Поддержка Windows 7:
> - Windows 7 по умолчанию поставляется с Windows PowerShell версии 2.0. Для выполнения командлетов PowerShell для Service Fabric требуется PowerShell начиная с версии 3.0. [Windows PowerShell 5,1 можно загрузить][powershell5-download] из центра загрузки Майкрософт.
> - Обратный прокси-сервер Service Fabric недоступен в Windows 7.

## <a name="install-the-sdk-and-tools"></a>Установка пакета SDK и инструментов

Установщик веб-платформы (WebPI) — это рекомендуемый способ установки пакета SDK и средств. При получении ошибок, при выполнения с помощью установщика веб-платформы, также можно найти прямые ссылки на средства установки в заметках о выпуске для определенной версии Service Fabric. Заметки о выпуске можно найти в различных объявлениях о выпусках в [блоге команды Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric).

> [!NOTE]
> Обновления локального кластера разработки Service Fabric не поддерживаются.

### <a name="to-use-visual-studio-2017-or-2019"></a>Использование Visual Studio 2017 или 2019

Средства Service Fabric являются частью рабочей нагрузки разработки Azure в Visual Studio 2017 и 2019. Эту рабочую нагрузку необходимо включить при установке Visual Studio.
Кроме того, необходимо установить пакет SDK и среду выполнения Microsoft Azure Service Fabric, используя установщик веб-платформы.

* [Установка пакета SDK Microsoft Azure Service Fabric][core-sdk]

### <a name="sdk-installation-only"></a>Только установка пакета SDK

Если вам требуется только пакет SDK, можно установить этот пакет:

* [Установка пакета SDK Microsoft Azure Service Fabric][core-sdk]

Текущие версии:

* Service Fabric SDK и средства 4.2.477
* 7.2.477 среды выполнения Service Fabric

Список поддерживаемых версий см. в статье [Поддерживаемые версии Service Fabric](service-fabric-versions.md).

> [!NOTE]
> Кластер одной машины (OneBox) не поддерживаются для обновлений приложений или кластеров. Удалите кластер OneBox и заново создайте его при необходимости обновления кластера или возникли проблемы с обновлением приложения. 

## <a name="enable-powershell-script-execution"></a>Включение сценариев PowerShell

Для создания локального кластера разработки и развертывания приложений из Visual Studio в Service Fabric используются сценарии Windows PowerShell. По умолчанию ОС Windows блокирует выполнение этих сценариев. Чтобы включить их, необходимо изменить политику выполнения PowerShell. Для этого запустите PowerShell с правами администратора и введите следующую команду:

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Force -Scope CurrentUser
```

## <a name="install-docker-optional"></a>Установка Docker (необязательно)

[Service Fabric — это оркестратор контейнеров](service-fabric-containers-overview.md) для развертывания микрослужб в кластере компьютеров. Для запуска приложений контейнера Windows на локальном кластере разработки необходимо сначала установить Docker для Windows. Скачайте [Docker CE для Windows (стабильная версия)](https://store.docker.com/editions/community/docker-ce-desktop-windows?tab=description). После установки и запуска Docker щелкните правой кнопкой мыши значок в области уведомлений и выберите **Switch to Windows containers** (Переключиться на контейнеры Windows). Это необходимо для запуска образов Docker на базе Windows.

## <a name="next-steps"></a>Следующие шаги

Среда разработки настроена, и вы готовы к созданию и запуску собственных приложений.

* [Узнайте, как создавать, развертывать и администрировать приложения](service-fabric-tutorial-create-dotnet-app.md)
* [Информация о моделях программирования: Reliable Services и Reliable Actors](service-fabric-choose-framework.md)
* [Ознакомление с примерами кода Service Fabric на GitHub](/samples/browse/?products=azure)
* [Визуализация кластера с помощью обозревателя Service Fabric](service-fabric-visualizing-your-cluster.md)
* [Подготовка среды разработки Linux в Windows](service-fabric-local-linux-cluster-windows.md)
* Дополнительные сведения о [вариантах поддержки Service Fabric](service-fabric-support.md)

[1]: https://azure.microsoft.com/campaigns/service-fabric/ "Страница кампании Service Fabric"
[2]: https://go.microsoft.com/fwlink/?LinkId=517106 "VS RC"
[full-bundle-vs2015]:https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-VS2015 "Ссылка на WebPI VS 2015"
[full-bundle-dev15]:https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-Dev15 "Ссылка на WebPI Dev15"
[core-sdk]:https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-CoreSDK "Ссылка на WebPI Core SDK"
[powershell5-download]:https://www.microsoft.com/download/details.aspx?id=54616
