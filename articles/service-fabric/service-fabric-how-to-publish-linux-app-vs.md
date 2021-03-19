---
title: Создание и публикация приложения a.Net Core в удаленном кластере Linux
description: Создание и публикация .Net Core приложений, предназначенных для удаленного кластера Linux из Visual Studio
author: peterpogorski
ms.topic: troubleshooting
ms.date: 5/20/2019
ms.author: pepogors
ms.openlocfilehash: c30eedb6782e4172d677f16e27441f28c78cdd89
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "75614355"
---
# <a name="use-visual-studio-to-create-and-publish-net-core-applications-targeting-a-remote-linux-service-fabric-cluster"></a>Создание и публикация приложений .Net Core, предназначенных для удаленного кластера Service Fabric Linux, с помощью Visual Studio
С помощью средств Visual Studio можно разрабатывать и публиковать Service Fabric .Net Core приложений, предназначенных для кластера Linux Service Fabric. Для развертывания .Net Core приложения, предназначенного для Service Fabric кластеров Linux из Visual Studio, версия пакета SDK должна быть 3,4 или выше.

> [!Note]
> Visual Studio не поддерживает отладку Service Fabric приложений, предназначенных для Linux.
>

## <a name="create-a-service-fabric-application-targeting-net-core"></a>Создание приложения Service Fabric, предназначенного для .Net Core
1. Запустите Visual Studio от имени **администратора**.
2. Создайте проект с **файлом->проектом New->**.
3. В диалоговом окне **Новый проект** выберите **приложение Cloud-> Service Fabric**.
![Создание приложения]
4. Присвойте приложению имя и нажмите кнопку **ОК**.
5. На странице **новая Service Fabric служба** выберите тип службы, которую вы хотите создать, в **разделе .NET Core**.
![Create-Service]

## <a name="deploy-to-a-remote-linux-cluster"></a>Развертывание в удаленном кластере Linux
1. В обозревателе решений щелкните приложение правой кнопкой мыши и выберите пункт **Сборка**.
![Сборка — приложение]
2. После завершения процесса сборки для приложения щелкните службу правой кнопкой мыши и выберите пункт изменить **файл CSPROJ**.
![Edit-CSPROJ]
3. Измените свойство Упдатесервицефабрикманифестенаблед с true на **false** , если служба является **типом проекта субъекта**. Если у приложения нет службы субъектов, перейдите к шагу 4.
```xml
    <UpdateServiceFabricManifestEnabled>False</UpdateServiceFabricManifestEnabled>
```
> [!Note]
> При установке для Упдатесервицефабрикманифестенаблед значения false обновления ServiceManifest.xml будут отключены во время сборки. Любое изменение, например добавление, удаление или переименование в службу, не будет отражено в ServiceManifest.xml. При внесении любых изменений необходимо либо обновить ServiceManifest вручную, либо временно установить Упдатесервицефабрикманифестенаблед в значение true, а затем создать службу, которая обновит ServiceManifest.xml, а затем вернуть ее к значению false.
>

4. Обновите Рунтимеиндетифиер с Win7-x64 на целевую платформу в проекте службы.
```xml
    <RuntimeIdentifier>ubuntu.16.04-x64</RuntimeIdentifier>
```
5. В ServiceManifest обновите программу EntryPoint, чтобы удалить. exe. 
```xml
    <EntryPoint> 
    <ExeHost> 
        <Program>Actor1</Program> 
    </ExeHost> 
    </EntryPoint>
```
6. В обозреватель решений щелкните приложение правой кнопкой мыши и выберите **опубликовать**. Откроется диалоговое окно **Publish** (Опубликовать).
7. В поле **Конечная точка подключения** выберите конечную точку для удаленного кластера Service Fabric Linux, который вы хотите назначить.
![Публикация — приложение]

<!--Image references-->
[Создание приложения]:./media/service-fabric-how-to-vs-remote-linux-cluster/create-application-remote-linux.png
[Create-Service]:./media/service-fabric-how-to-vs-remote-linux-cluster/create-service-remote-linux.png
[Сборка — приложение]:./media/service-fabric-how-to-vs-remote-linux-cluster/build-application-remote-linux.png
[Edit-CSPROJ]:./media/service-fabric-how-to-vs-remote-linux-cluster/edit-csproj-remote-linux.png
[Публикация — приложение]:./media/service-fabric-how-to-vs-remote-linux-cluster/publish-remote-linux.png

## <a name="next-steps"></a>Дальнейшие действия
* Узнайте [, как приступить к работе с Service Fabric с .NET Core](https://azure.microsoft.com/resources/samples/service-fabric-dotnet-core-getting-started/)
