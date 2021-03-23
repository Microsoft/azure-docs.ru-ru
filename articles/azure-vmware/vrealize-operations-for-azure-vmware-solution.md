---
title: Настройка операций Вреализе для решения VMware для Azure
description: Узнайте, как настроить операции Вреализе для частного облака решения Azure VMware.
ms.topic: how-to
ms.date: 01/26/2021
ms.openlocfilehash: 8015bb61a7401b4c97807e0256e06d4967c39026
ms.sourcegitcommit: ba3a4d58a17021a922f763095ddc3cf768b11336
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104802494"
---
# <a name="set-up-vrealize-operations-for-azure-vmware-solution"></a>Настройка операций Вреализе для решения VMware для Azure


Вреализе Operations Manager — это платформа управления операциями, позволяющая администраторам инфраструктуры VMware отслеживать системные ресурсы. Эти системные ресурсы могут быть уровня приложения или инфраструктуры (как физических, так и виртуальных) объектов. Большинство администраторов VMware используют Вреализе операции для мониторинга и управления компонентами частного облака VMware — vCenter, ESXi, НСКС-T, vSAN и VMware ХККС.  Каждое подготовленное частное облако решения Azure VMware включает в себя выделенный сервер vCenter, НСКС-T, vSAN и ХККС. 

Тщательно проверьте [перед началом](#before-you-begin) и [предварительными требованиями](#prerequisites) . Затем мы рассмотрим две стандартные топологии развертывания:

> [!div class="checklist"]
> * [Локальные операции Вреализе Управление развертыванием решения Azure VMware](#on-premises-vrealize-operations-managing-azure-vmware-solution-deployment)
> * [Операции Вреализе, выполняемые в развертывании решения VMware в Azure](#vrealize-operations-running-on-azure-vmware-solution-deployment)

## <a name="before-you-begin"></a>Перед началом
* Дополнительные сведения о развертывании операций Вреализе см. в [документации по продукту вреализе Operations Manager](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.vapp.doc/GUID-7FFC61A0-7562-465C-A0DC-46D092533984.html) . 
* Ознакомьтесь с [циклом руководств](tutorial-network-checklist.md) по программно-определяемому центру обработки данных (SDDC) Решения Azure VMware.
* При необходимости ознакомьтесь с документацией по продукту [Вреализе Operations Remote Controller](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.vapp.doc/GUID-263F9219-E801-4383-8A59-E84F3D01ED6B.html) для локальных операций вреализе, управляющих развертыванием решения VMware в Azure. 


## <a name="prerequisites"></a>Предварительные требования
* [вреализе Operations Manager](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.vapp.doc/GUID-7FFC61A0-7562-465C-A0DC-46D092533984.html) установлен.
* VPN или Azure ExpressRoute, настроенные между локальной средой и решением VMware для Azure.
* Частное облако решения Azure VMware развернуто в Azure.



## <a name="on-premises-vrealize-operations-managing-azure-vmware-solution-deployment"></a>Локальные операции Вреализе Управление развертыванием решения Azure VMware
У большинства клиентов есть локальное развертывание операций Вреализе для управления одним или несколькими локальными доменами vCenter. При подготовке частного облака решения Azure VMware они подключаются к локальной среде с частным облаком с помощью Azure ExpressRoute или решения VPN уровня 3.  

:::image type="content" source="media/vrealize-operations-manager/vrealize-operations-deployment-option-1.png" alt-text="Локальные операции Вреализе Управление развертыванием решения Azure VMware" border="false":::

Чтобы расширить возможности операций Вреализе в частном облаке решения Azure VMware, создайте [экземпляр адаптера для ресурсов частного облака](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.config.doc/GUID-640AD750-301E-4D36-8293-1BFEB67E2600.html). Он собирает данные из частного облака решения Azure VMware и переносит их в локальные операции Вреализе. Локальный экземпляр Вреализе Operations Manager может напрямую подключаться к серверу vCenter и диспетчеру НСКС-T в решении VMware для Azure. При необходимости можно развернуть удаленный сборщик операций Вреализе в частном облаке решения Azure VMware. Сборщик сжимает и шифрует данные, собранные из частного облака, перед отправкой через сеть ExpressRoute или VPN в Вреализе Operations Manager, выполняющихся локально. 

> [!TIP]
> Пошаговое руководство по установке Вреализе Operations Manager см. в [документации по VMware](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.vapp.doc/GUID-7FFC61A0-7562-465C-A0DC-46D092533984.html) . 



## <a name="vrealize-operations-running-on-azure-vmware-solution-deployment"></a>Операции Вреализе, выполняемые в развертывании решения VMware в Azure

Другой вариант — развернуть экземпляр Вреализе Operations Manager в кластере vSphere в частном облаке. 

>[!IMPORTANT]
>В настоящее время этот параметр не поддерживается в VMware.

:::image type="content" source="media/vrealize-operations-manager/vrealize-operations-deployment-option-2.png" alt-text="Операции Вреализе, выполняемые в решении VMware для Azure" border="false":::

После развертывания экземпляра можно настроить операции Вреализе для получения данных из vCenter, ESXi, НСКС-T, vSAN и ХККС. 



## <a name="known-limitations"></a>Известные ограничения

- У пользователя **cloudadmin \@ vSphere. local** в решении VMware Azure есть [ограниченные привилегии](concepts-identity.md).  Виртуальные машины в решении VMware для Azure не поддерживают сбор гостевых памяти с помощью средств VMware.  В этом случае используется активный и потребляемый объем памяти.
- Оптимизация рабочей нагрузки для бизнес-целей на основе узла не работает, так как решения VMware для Azure управляют конфигурациями кластера, включая параметры DRS.
- Оптимизация рабочей нагрузки для размещения между кластерами в SDDC с использованием бизнеса на основе кластера полностью поддерживается с Вреализе Operations Manager 8,0 и выше. Однако оптимизация рабочей нагрузки не учитывает пулы ресурсов и размещает виртуальные машины на уровне кластера. Пользователь может вручную исправить его в интерфейсе vCenter Server решения Azure VMware.
- Вы не можете войти в Вреализе Operations Manager с помощью решения Azure VMware vCenter Server учетные данные. 
- Решение VMware для Azure не поддерживает подключаемый модуль Operations Manager Вреализе.

При подключении службы vCenter для Azure VMware к Вреализе Operations Manager с помощью vCenter Server облачной учетной записи вы увидите предупреждение:

:::image type="content" source="./media/vrealize-operations-manager/warning-adapter-instance-creation-succeeded.png" alt-text="Экземпляр адаптера предупреждений создан":::

Это предупреждение возникает, потому что пользователь **cloudadmin \@ vSphere. local** в решении VMware Azure не имеет достаточных привилегий для выполнения всех vCenter Server действий, необходимых для регистрации. Однако привилегии достаточно для того, чтобы экземпляр адаптера произйдет сбор данных, как показано ниже:

:::image type="content" source="./media/vrealize-operations-manager/adapter-instance-to-perform-data-collection.png" alt-text="Экземпляр адаптера для выполнения сбора данных":::

Дополнительные сведения см. в разделе [привилегии, необходимые для настройки экземпляра адаптера vCenter](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.core.doc/GUID-3BFFC92A-9902-4CF2-945E-EA453733B426.html).

<!-- LINKS - external -->


<!-- LINKS - internal -->




