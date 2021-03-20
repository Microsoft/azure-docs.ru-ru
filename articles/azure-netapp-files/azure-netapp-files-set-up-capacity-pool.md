---
title: Настройка пула емкости для Azure NetApp Files | Документация Майкрософт
description: Описание настройки пула емкости, чтобы иметь возможность создавать в нем тома.
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
ms.date: 09/22/2020
ms.author: b-juche
ms.openlocfilehash: 2b52ad50854092cddd7b9e79cbeebd4a83017081
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91325422"
---
# <a name="set-up-a-capacity-pool"></a>Настройка пула емкости

Настройка пула емкости позволяет создавать в нем тома.  

## <a name="before-you-begin"></a>Перед началом 

Должна быть создана учетная запись NetApp.   

[Создание учетной записи NetApp](azure-netapp-files-create-netapp-account.md)

## <a name="steps"></a>Шаги 

1. Перейдите в колонку управления учетной записью NetApp, а затем в навигационной панели выберите **Пулы емкости**.  
    
    ![Переход к пулам емкости](../media/azure-netapp-files/azure-netapp-files-navigate-to-capacity-pool.png)

2. Нажмите **+ Добавить пулы** для создания нового пула емкости.   
    Появится окно New Capacity Pool (Новый пул емкости).

3. Укажите следующую информацию для нового пула емкости.  
   * **имя**;  
     Придумайте название для пула емкости.  
     Название пула емкости должно быть уникальным для каждой учетной записи NetApp.

   * **Уровень обслуживания**   
     В этом поле отображается целевая производительность пула емкости.  
     Укажите уровень обслуживания для пула емкости: [**Ultra**](azure-netapp-files-service-levels.md#Ultra), [**Premium**](azure-netapp-files-service-levels.md#Premium)или [**Standard**](azure-netapp-files-service-levels.md#Standard).

    * **Изменять**     
     Укажите приобретаемый размер пула емкости.        
     Минимальный размер пула емкости равен 4 ТиБ. Можно создавать пул с размером, кратным 4 ТиБ.   

   * **Качество**   
     Укажите, должен ли пул емкости использовать тип качества обслуживания **вручную** или **автоматически** .  

     Сведения о типах QoS см. в разделе [иерархия хранилища](azure-netapp-files-understand-storage-hierarchy.md) и [вопросы производительности](azure-netapp-files-performance-considerations.md) .  

     > [!IMPORTANT] 
     > Настройка **типа QoS** " **вручную** " является постоянной. Невозможно преобразовать пул емкости QoS вручную для использования автоматического качества обслуживания. Однако можно преобразовать пул автоматического качества обслуживания, чтобы использовать QoS вручную. См. раздел [Изменение пула емкости для использования QoS вручную](manage-manual-qos-capacity-pool.md#change-to-qos).   
     > Чтобы использовать тип QoS вручную, для пула емкости требуется регистрация. См. статью [Управление пулом емкости качества обслуживания вручную](manage-manual-qos-capacity-pool.md#register-the-feature). 

    ![Новый пул емкости](../media/azure-netapp-files/azure-netapp-files-new-capacity-pool.png)

4. Нажмите кнопку **Создать**.

## <a name="next-steps"></a>Дальнейшие действия 

- [Иерархия хранилища](azure-netapp-files-understand-storage-hierarchy.md) 
- [Уровни обслуживания для Azure NetApp Files](azure-netapp-files-service-levels.md)
- [Страница цен Azure NetApp Files](https://azure.microsoft.com/pricing/details/storage/netapp/)
- [Управление пулом емкости качества обслуживания вручную](manage-manual-qos-capacity-pool.md)
- [Делегирование подсети службе Azure NetApp Files](azure-netapp-files-delegate-subnet.md)
