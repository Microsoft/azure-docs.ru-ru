---
title: Подключение Azure ExpressRoute к облачной инфраструктуре Oracle | Документация Майкрософт
description: Подключение Azure ExpressRoute с помощью Oracle Cloud Infrastructure (OCI) Фастконнект для включения решений для приложений Oracle в разных облаках
author: dbakevlar
ms.service: virtual-machines
ms.subservice: oracle
ms.collection: linux
ms.topic: article
ms.date: 03/16/2020
ms.author: rogardle
ms.openlocfilehash: 09264f9f20411e7536eb4a1dbf12ac297e7e3ef9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101675704"
---
# <a name="set-up-a-direct-interconnection-between-azure-and-oracle-cloud-infrastructure"></a>Настройка прямого взаимодействия между Azure и облачной инфраструктурой Oracle  

Чтобы создать [интегрированную среду с несколькими облаками](oracle-oci-overview.md), корпорация Майкрософт и Oracle предлагают прямое взаимодействие между Azure и облачной инфраструктурой Oracle (OCI) через [ExpressRoute](../../../expressroute/expressroute-introduction.md) и [фастконнект](https://docs.cloud.oracle.com/iaas/Content/Network/Concepts/fastconnectoverview.htm). Благодаря взаимосвязи ExpressRoute и Фастконнект клиенты могут столкнуться с низкой задержкой, высокой пропускной способностью и частным прямым подключением между двумя облаками.

> [!IMPORTANT]
> К маю 2020 г. следующие приложения будут сертифицированы Oracle для запуска в Azure при использовании соединения между облаками Azure и Oracle Cloud:
> * E-Business Suite
> * JD Edwards EnterpriseOne;
> * PeopleSoft;
> * приложения Oracle Retail;
> * Oracle Hyperion Financial Management.

На следующем рисунке показан общий обзор соединения:

![Сетевое подключение между облаками](media/configure-azure-oci-networking/azure-oci-connect.png)

## <a name="prerequisites"></a>Предварительные условия

* Чтобы установить подключение между Azure и OCI, необходимо иметь активную подписку Azure и активную службу OCI.

* Подключение возможно только в том случае, если расположение пиринга Azure ExpressRoute находится в том же расположении, что и Фастконнект OCI. См. сведения о [доступности регионов](oracle-oci-overview.md#region-availability).

## <a name="configure-direct-connectivity-between-expressroute-and-fastconnect"></a>Настройка прямого подключения ExpressRoute и Фастконнект

1. Создайте стандартный канал ExpressRoute для подписки Azure в группе ресурсов. 
    * При создании ExpressRoute выберите **Oracle Cloud фастконнект** в качестве поставщика услуг. Чтобы создать канал ExpressRoute, ознакомьтесь с пошаговым [руководством](../../../expressroute/expressroute-howto-circuit-portal-resource-manager.md).
    * Канал ExpressRoute Azure обеспечивает детализированные параметры пропускной способности, тогда как Фастконнект поддерживает 1, 2, 5 или 10 Гбит/с. Поэтому рекомендуется выбрать один из этих параметров для соответствующей пропускной способности в ExpressRoute.

    ![Создание канала ExpressRoute](media/configure-azure-oci-networking/exr-create-new.png)
1. Запишите **ключ службы** ExpressRoute. При настройке канала Фастконнект необходимо указать ключ.

    ![Ключ службы ExpressRoute](media/configure-azure-oci-networking/exr-service-key.png)

    > [!IMPORTANT]
    > Плата за ExpressRoute будет взиматься сразу после подготовки канала ExpressRoute (даже если **состояние поставщика** **не подготовлено**).

1. Выделить два пространства частных IP-адресов (/30), которые не пересекаются с пространством IP-адресов виртуальной сети Azure или сетевого адреса OCI. Мы будем ссылаться на первое пространство IP-адресов в качестве основного адресного пространства и второго пространства IP-адресов в качестве дополнительного адресного пространства. Запишите адреса, которые понадобятся при настройке канала Фастконнект.
1. Создайте шлюз динамической маршрутизации (ДРГ). Он понадобится при создании канала Фастконнект. Дополнительные сведения см. в документации по [шлюзу динамической маршрутизации](https://docs.cloud.oracle.com/iaas/Content/Network/Tasks/managingDRGs.htm) .
1. Создайте цепь Фастконнект в клиенте Oracle. Дополнительные сведения см. в [документации Oracle](https://docs.cloud.oracle.com/iaas/Content/Network/Concepts/azure.htm).
  
    * В разделе Конфигурация Фастконнект выберите **Microsoft Azure: ExpressRoute** в качестве поставщика.
    * Выберите шлюз динамической маршрутизации, подготовленный на предыдущем шаге.
    * Выберите пропускную способность для подготовки. Для оптимальной производительности пропускная способность должна соответствовать пропускной способности, выбранной при создании канала ExpressRoute.
    * В поле **ключ службы поставщика** вставьте ключ службы ExpressRoute.
    * Используйте пространство частных IP-адресов First/30, отрезать пространство на предыдущем шаге для **основного IP-адреса BGP** и второго/30-адресного пространства для **дополнительного IP-адреса BGP** .
        * Назначьте первый пригодный для использования адрес из двух диапазонов для IP-адреса Oracle BGP (первичного и вторичного) и второго адреса для IP-адреса клиента BGP (с точки зрения Фастконнект). Первый пригодный IP-адрес — это второй IP-адрес в адресном пространстве/30 (первый IP-адрес зарезервирован корпорацией Майкрософт).
    * Нажмите кнопку **Создать**.
1. Завершите связывание Фастконнект с виртуальной облачной сетью в клиенте Oracle через шлюз динамической маршрутизации, используя таблицу маршрутов.
1. Перейдите в Azure и убедитесь, что **состояние поставщика** для канала ExpressRoute изменилось на " **подготовлено** " и что пиринг типа "частный" **Azure** был подготовлен. Это предварительные требования для следующих шагов.

    ![Состояние поставщика ExpressRoute](media/configure-azure-oci-networking/exr-provider-status.png)
1. Щелкните частный пиринг **Azure** . Вы увидите, что сведения об пиринга автоматически были настроены на основе информации, введенной при настройке канала Фастконнект.

    ![Частные параметры пиринга](media/configure-azure-oci-networking/exr-private-peering.png)

## <a name="connect-virtual-network-to-expressroute"></a>Подключение виртуальной сети к ExpressRoute

1. Создайте виртуальную сеть и шлюз виртуальной сети, если вы еще этого не сделали. Дополнительные сведения см. в пошаговом [руководств](../../../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md).
1. Настройте подключение между шлюзом виртуальной сети и каналом ExpressRoute, выполнив [сценарий terraform](https://github.com/microsoft/azure-oracle/tree/master/InterConnect-2) или выполнив команду PowerShell для [настройки фастпас ExpressRoute](../../../expressroute/expressroute-howto-linkvnet-arm.md#configure-expressroute-fastpath).

После завершения настройки сети можно проверить допустимость конфигурации, щелкнув ссылку **получить записи ARP** и **получить таблицу маршрутов** в колонке частного пиринга ExpressRoute в портал Azure.

## <a name="automation"></a>Автоматизация

Корпорация Майкрософт создала сценарии terraform, чтобы включить автоматическое развертывание сетевого соединения. Перед выполнением скрипты terraform должны пройти проверку подлинности в Azure, так как для них требуются соответствующие разрешения в подписке Azure. Проверку подлинности можно выполнить с помощью [субъекта-службы Azure Active Directory](../../../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) или с помощью Azure CLI. Дополнительные сведения см. в [документации по terraform](https://www.terraform.io/docs/providers/azurerm/auth/azure_cli.html).

Сценарии terraform и связанная документация по развертыванию межподключения можно найти в этом [репозитории GitHub](https://aka.ms/azureociinterconnecttf).

## <a name="monitoring"></a>Мониторинг

Установив агенты в обоих облаках, вы можете использовать Azure [Монитор производительности сети (NPM)](../../../expressroute/how-to-npm.md) для мониторинга производительности сквозной сети. NPM помогает легко выявление сетевых проблем и помогает устранить их.

## <a name="delete-the-interconnect-link"></a>Удаление связи Interconnect

Чтобы удалить соединение, необходимо выполнить следующие действия в указанном порядке. Несоблюдение этого действия приведет к появлению канала ExpressRoute "состояние сбоя".

1. Удалите подключение ExpressRoute. Удалите подключение, щелкнув значок **удаления** на странице для подключения. Дополнительные сведения см. в [документации по ExpressRoute](../../../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md#clean-up-resources).
1. Удалите Фастконнект Oracle из облачной консоли Oracle.
1. После удаления канала Фастконнект для Oracle можно удалить канал Azure ExpressRoute.

На этом этапе процесс удаления и отмены инициализации завершен.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о межоблачном подключении между OCI и Azure см. в [документации Oracle](https://docs.cloud.oracle.com/iaas/Content/Network/Concepts/azure.htm).
* Используйте [сценарии terraform](https://aka.ms/azureociinterconnecttf) для развертывания инфраструктуры для целевых приложений Oracle в Azure и настройки сетевого соединения. 
