---
title: Управление пулом емкости QoS вручную для Azure NetApp Files | Документация Майкрософт
description: Описывается, как управлять пулом емкости, использующим тип QoS вручную, включая настройку пула ресурсов вручную QoS и изменение пула ресурсов для использования QoS вручную.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 02/04/2021
ms.author: b-juche
ms.openlocfilehash: 1c20190ba1a997ef85f4023a54ecea3c2a77ae53
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102183834"
---
# <a name="manage-a-manual-qos-capacity-pool"></a>Управление пулом емкости качества обслуживания вручную

В этой статье описывается, как управлять пулом емкости, использующим тип QoS вручную.  

Сведения о типах QoS см. в разделе [иерархия хранилища Azure NetApp Files](azure-netapp-files-understand-storage-hierarchy.md) и [рекомендации по производительности для Azure NetApp Files](azure-netapp-files-performance-considerations.md) .  

## <a name="register-the-feature"></a>регистрация компонента
Сейчас функция типа QoS вручную доступна в предварительной версии. Если эта функция используется впервые, ее необходимо сначала зарегистрировать.
  
1.  Зарегистрируйте функцию:

    ```azurepowershell-interactive
    Register-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFFlexPool
    ```

2. Проверьте состояние регистрации компонента: 

    > [!NOTE]
    > **RegistrationState** может находиться в состоянии до `Registering` 60 минут до перехода на `Registered` . Прежде чем продолжить, подождите, пока состояние не будет **зарегистрировано** .

    ```azurepowershell-interactive
    Get-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFFlexPool
    ```
Вы также можете использовать [Azure CLI команды](/cli/azure/feature) `az feature register` , `az feature show` чтобы зарегистрировать эту функцию и отобразить состояние регистрации. 

## <a name="set-up-a-new-manual-qos-capacity-pool"></a>Настройка нового пула ресурсов качества обслуживания вручную 

Чтобы создать новый пул ресурсов, используя тип качества обслуживания вручную:

1. Выполните действия, описанные в разделе [Настройка пула емкости](azure-netapp-files-set-up-capacity-pool.md).  

2. В окне новый пул ресурсов выберите тип **качества обслуживания вручную** .  

## <a name="change-a-capacity-pool-to-use-manual-qos"></a><a name="change-to-qos"></a>Изменение пула емкости для использования QoS вручную

Вы можете изменить пул ресурсов, который в настоящее время использует автоматический тип QoS, для использования типа QoS вручную.  

> [!IMPORTANT]
> Установка для типа емкости значения QoS вручную является постоянным изменением. Невозможно преобразовать средство ручной емкости типа QoS в пул ресурсов автоматического качества обслуживания.  
> Во время преобразования уровни пропускной способности могут быть ограничены, чтобы соответствовать ограничениям пропускной способности для томов типа QoS вручную. См. раздел [ограничения ресурсов для Azure NetApp Files](azure-netapp-files-resource-limits.md#resource-limits).

1. В колонке управления для учетной записи NetApp щелкните **Пулы ресурсов** , чтобы отобразить существующие пулы мощностей.   
 
2.  Щелкните пул ресурсов, который вы хотите изменить, чтобы использовать QoS вручную.

3.  Щелкните **изменить тип качества обслуживания**. Затем задайте для **нового типа качества обслуживания** значение **вручную**. Нажмите кнопку **OK**. 

![Изменение типа QoS](../media/azure-netapp-files/change-qos-type.png)


## <a name="monitor-the-throughput-of-a-manual-qos-capacity-pool"></a>Мониторинг пропускной способности пула ресурсов ручного обслуживания вручную  

Доступны метрики, помогающие отслеживать пропускную способность чтения и записи тома.  См. раздел [метрики для Azure NetApp Files](azure-netapp-files-metrics.md).  

## <a name="modify-the-allotted-throughput-of-a-manual-qos-volume"></a>Изменение выделенной пропускной способности для тома QoS вручную 

Если том содержится в пуле ресурсов ручного качества обслуживания вручную, можно изменить выделенную пропускную способность тома по мере необходимости.

1. На странице **тома** выберите том, пропускную способность которого требуется изменить.   

2. Щелкните **изменить пропускную способность**. Укажите нужную **пропускную способность (MIB/с)** . Нажмите кнопку **OK**. 

    ![Изменение пропускной способности QoS](../media/azure-netapp-files/change-qos-throughput.png)

## <a name="next-steps"></a>Дальнейшие действия  

* [Настройка пула емкости](azure-netapp-files-set-up-capacity-pool.md)
* [Метрики для Azure NetApp Files](azure-netapp-files-metrics.md)
* [Рекомендации по ускорению Azure NetApp Files](azure-netapp-files-performance-considerations.md)
* [Устранение неполадок с пулами емкости](troubleshoot-capacity-pools.md)
* [Иерархия хранилища Azure NetApp Files](azure-netapp-files-understand-storage-hierarchy.md)
* [Уровни обслуживания для Azure NetApp Files](azure-netapp-files-service-levels.md)
* [Модель затрат для Azure NetApp Files](azure-netapp-files-cost-model.md)
* [Ограничения ресурсов для службы Azure NetApp Files](azure-netapp-files-resource-limits.md)
* [Создание тома NFS](azure-netapp-files-create-volumes.md)
* [Создание тома SMB](azure-netapp-files-create-volumes-smb.md)
* [Создание тома с двумя протоколами](create-volumes-dual-protocol.md)
