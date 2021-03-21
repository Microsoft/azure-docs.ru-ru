---
title: Настройка безопасных подключений к кластерам Azure Service Fabric
description: Узнайте о том, как использовать Visual Studio для настройки безопасных подключений, поддерживаемых кластером Azure Service Fabric.
author: cawaMS
ms.topic: conceptual
ms.date: 8/04/2017
ms.author: cawa
ms.openlocfilehash: d4d6b781d97d481793e69cf2ca97cca5b93ce432
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96008537"
---
# <a name="configure-secure-connections-to-a-service-fabric-cluster-from-visual-studio"></a>Настройка безопасных подключений к кластеру Service Fabric из Visual Studio
Узнайте, как использовать Azure Visual Studio для безопасного доступа к кластеру Service Fabric с настроенной политикой контроля доступа.

## <a name="cluster-connection-types"></a>Типы подключения кластера
Кластер Azure Service Fabric поддерживает два типа соединений: **небезопасные** соединения и безопасные соединения **на основе сертификата x509**. (Для Service Fabric кластеров, размещенных локально, также поддерживаются проверки подлинности **Windows** и **dSTS** .) При создании кластера необходимо настроить тип подключения кластера. После создания изменить тип подключения нельзя.

Средства Service Fabric Visual Studio поддерживают все типы проверки подлинности для подключения к кластеру для публикации. Инструкции по настройке безопасного кластера Service Fabric см. в статье [Создание кластера Service Fabric в Azure с помощью портала Azure](service-fabric-cluster-creation-via-portal.md).

## <a name="configure-cluster-connections-in-publish-profiles"></a>Настройка подключения кластера в профилях публикации
При публикации проекта Service Fabric из Visual Studio в диалоговом окне **Опубликовать приложение Service Fabric** можно выберите кластер Azure Service Fabric. В разделе **Конечная точка подключения** выберите кластер в своей подписке.

![Диалоговое окно **Опубликовать приложение Service Fabric** позволяет настроить подключение к Service Fabric.][publishdialog]

В диалоговом окне **Опубликовать приложение Service Fabric** автоматически будет проверено подключение к кластеру. Если отобразится запрос на вход в учетную запись Azure, выполните его. Если проверка проходит успешно, это означает, что в системе установлены необходимые сертификаты для безопасного подключения к кластеру или то, что кластер не является безопасным. Неудачный результат проверки может быть вызван проблемами с сетью или неправильной настройкой системы для безопасного подключения к кластеру.

![В диалоговом окне **Опубликовать приложение Service Fabric** проверяется имеющееся правильно настроенное подключение к кластеру Service Fabric.][selectsfcluster]

### <a name="to-connect-to-a-secure-cluster"></a>Безопасное подключение к кластеру
1. Убедитесь, что вам доступен один из клиентских сертификатов, которым доверяет целевой кластер. Сертификат обычно распространяется в виде файла обмена персональной информацией (.pfx). Руководство по настройке доступа клиента к серверу см. в статье [Создание кластера Service Fabric в Azure с помощью портала Azure](service-fabric-cluster-creation-via-portal.md).
2. Установите доверенный сертификат. Для этого дважды щелкните PFX-файл или импортируйте сертификаты с помощью сценария PowerShell PfxCertificate. Установите сертификат в каталог **Cert:\LocalMachine\My**. При импорте сертификата можно принять все настройки по умолчанию.
3. Выберите **Опубликовать** в контекстном меню проекта, чтобы открыть диалоговое окно **Опубликовать приложение Azure**, а затем выберите целевой кластер. Средство автоматически разрешает подключение и сохраняет параметры безопасного подключения в профиле публикации.
4. Можно изменить профиль публикации, чтобы указать безопасное подключение к кластеру (необязательно).
   
   Так как вы добавляете информацию о сертификате в XML-файл профиля публикации вручную, запишите имя хранилища сертификатов, расположение хранилища и отпечаток сертификата. Эти значения нужно будет указать в качестве имени и расположения хранилища сертификатов. Дополнительные сведения см. в статье [Практическое руководство. Извлечение отпечатка сертификата](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric).
   
   Параметры *ClusterConnectionParameters* позволяют задать параметры PowerShell для подключения к кластеру Service Fabric. Допустимы все значения параметров, которые принимаются командлетом Connect-ServiceFabricCluster. Список доступных параметров см. в статье [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster).
   
   При публикации на удаленный кластер необходимо указать соответствующие параметры для этого конкретного кластера. Ниже приведен пример небезопасного подключения к кластеру:
   
   `<ClusterConnectionParameters ConnectionEndpoint="mycluster.westus.cloudapp.azure.com:19000" />`
   
   Ниже приведен пример для безопасного подключения к кластеру на основе сертификатов x509:
   
   ```xml
   <ClusterConnectionParameters
   ConnectionEndpoint="mycluster.westus.cloudapp.azure.com:19000"
   X509Credential="true"
   ServerCertThumbprint="0123456789012345678901234567890123456789"
   FindType="FindByThumbprint"
   FindValue="9876543210987654321098765432109876543210"
   StoreLocation="CurrentUser"
   StoreName="My" />
   ```
5. Измените другие необходимые параметры, такие как параметры обновления и расположение файла параметров приложения, и снова опубликуйте приложение из диалогового окна **Публикация приложения Service Fabric** в Visual Studio.

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о доступе к кластерам Service Fabric см. в статье [Визуализация кластера с помощью Service Fabric Explorer](service-fabric-visualizing-your-cluster.md).

<!--Image references-->
[publishdialog]:./media/service-fabric-visualstudio-configure-secure-connections/publishdialog.png
[selectsfcluster]:./media/service-fabric-visualstudio-configure-secure-connections/selectsfcluster.png
