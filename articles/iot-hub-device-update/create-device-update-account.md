---
title: Создание учетной записи обновления устройства в обновлении для центра Интернета вещей Azure | Документация Майкрософт
description: Создайте учетную запись обновления устройства в центре обновления для центра Интернета вещей Azure.
author: vimeht
ms.author: vimeht
ms.date: 2/11/2021
ms.topic: how-to
ms.service: iot-hub-device-update
ms.openlocfilehash: 5956b7b74d27a4f9a2b79ee3950c8ac765610c70
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105558488"
---
# <a name="device-update-for-iot-hub-resource-management"></a>Обновление устройств для управления ресурсами Центра Интернета вещей

Чтобы начать работу с обновлением устройства, необходимо создать учетную запись обновления устройства, экземпляр и настроить роли контроля доступа. 

## <a name="prerequisites"></a>Предварительные требования

* Доступ к Центру Интернета вещей. Рекомендуется использовать уровень S1 (Стандартный) или выше. 
* Поддерживаемые браузеры:
  * [Microsoft Edge](https://www.microsoft.com/edge)
  * Google Chrome

## <a name="create-a-device-update-account"></a>Создание учетной записи обновления устройств

1. Переход к [портал Azure](https://portal.azure.com)

2. Щелкните Создать ресурс и выполните поиск по запросу "обновление устройства для центра Интернета вещей".

   :::image type="content" source="media/create-device-update-account/device-update-marketplace.png" alt-text="Снимок экрана обновления устройства для ресурса центра Интернета вещей." lightbox="media/create-device-update-account/device-update-marketplace.png":::

3. Щелкните Создать — > обновление устройства для центра Интернета вещей.

4. Укажите подписку Azure, которая будет связана с учетной записью обновления устройства и группой ресурсов

5. Укажите имя и расположение для учетной записи обновления устройства

   :::image type="content" source="media/create-device-update-account/account-details.png" alt-text="Снимок экрана со сведениями об учетной записи." lightbox="media/create-device-update-account/account-details.png":::

 > [!NOTE]
 > Чтобы найти регионы, в которых доступно обновление устройства для центра Интернета вещей, перейдите на [страницу продукты Azure по регионам](https://azure.microsoft.com/global-infrastructure/services/?products=iot-hub) . Если обновление устройства для центра Интернета вещей недоступно в вашем регионе, вы можете создать учетную запись в доступном для вас регионе. 

6. Щелкните "Далее: Проверка и создание>".

   :::image type="content" source="media/create-device-update-account/account-review.png" alt-text="Снимок экрана с проверкой сведений об учетной записи." lightbox="media/create-device-update-account/account-review.png":::

7. Просмотрите сведения и нажмите кнопку "создать". Вы увидите, что развертывание выполняется. 

   :::image type="content" source="media/create-device-update-account/account-deployment-inprogress.png" alt-text="Снимок экрана: выполняется развертывание учетной записи." lightbox="media/create-device-update-account/account-deployment-inprogress.png":::

8. Вы увидите, что состояние развертывания изменится на "завершено" через несколько минут. Щелкните "переход к ресурсу".

   :::image type="content" source="media/create-device-update-account/account-complete.png" alt-text="Снимок экрана развертывания учетной записи завершен." lightbox="media/create-device-update-account/account-complete.png":::

## <a name="create-a-device-update-instance"></a>Создание экземпляра обновления устройства 

Экземпляр обновления устройства связан с одним центром Интернета вещей. Выберите центр Интернета вещей, который будет использоваться с обновлением устройства. На этом этапе мы создадим новую политику общего доступа, чтобы обновление устройства использовало только необходимые разрешения для работы с центром Интернета вещей (запись в реестр и подключение к службе). Эта политика гарантирует, что доступ будет ограничен только обновлением устройства.

Создание экземпляра обновления устройства после создания учетной записи.

1. После создания нового ресурса учетной записи перейдите в колонку экземпляры управления экземплярами.

   :::image type="content" source="media/create-device-update-account/instance-blade.png" alt-text="Снимок экрана управления экземплярами в пределах учетной записи." lightbox="media/create-device-update-account/instance-blade.png":::

2. Щелкните "создать и укажите имя экземпляра и выберите центр Интернета вещей".

   :::image type="content" source="media/create-device-update-account/instance-details.png" alt-text="Снимок экрана сведений об экземпляре." lightbox="media/create-device-update-account/instance-details.png":::

   > [!NOTE] 
   > Центр Интернета вещей, связанный с ресурсом обновления устройства, не должен находиться в том же регионе, что и учетная запись обновления устройства. Однако для повышения производительности рекомендуется, чтобы центр Интернета вещей наследуется в том же регионе, что и, или близко к региону учетной записи обновления устройства. 

3. Нажмите "Создать". Экземпляр будет отображаться в состоянии "создание". 

   :::image type="content" source="media/create-device-update-account/instance-creating.png" alt-text="Снимок экрана создания экземпляра." lightbox="media/create-device-update-account/instance-creating.png":::

4. Разрешить завершение развертывания экземпляра 5-10 мин. Обновите состояние до состояния "состояние подготовки" — "успех".

   :::image type="content" source="media/create-device-update-account/instance-succeeded.png" alt-text="Снимок экрана: создание экземпляра прошло." lightbox="media/create-device-update-account/instance-succeeded.png":::

## <a name="configure-iot-hub"></a>Настройка Центра Интернета вещей 

Чтобы обновление устройства получало уведомления об изменениях из центра Интернета вещей, обновление устройства интегрируется со встроенным концентратором событий. При нажатии кнопки "настроить центр Интернета вещей" настраиваются необходимые маршруты сообщений и политики доступа, необходимые для взаимодействия с устройствами IoT. 

Настройка центра Интернета вещей

1. После того как экземпляр "состояние подготовки" вернется к "успеху", выберите экземпляр в колонке управление экземплярами. Щелкните "настроить центр Интернета вещей".

   :::image type="content" source="media/create-device-update-account/instance-configure.png" alt-text="Снимок экрана: Настройка центра Интернета вещей для экземпляра." lightbox="media/create-device-update-account/instance-configure.png":::

2. Выберите "я согласен сделать эти изменения"

   :::image type="content" source="media/create-device-update-account/instance-configure-selected.png" alt-text="Снимок экрана: согласие на настройку центра Интернета вещей для экземпляра." lightbox="media/create-device-update-account/instance-configure-selected.png":::

3. Щелкните "Обновить".

[Сведения о настроенных маршрутах сообщений.](device-update-resources.md) 


## <a name="configure-access-control-roles"></a>Настройка ролей контроля доступа

Чтобы другие пользователи могли получить доступ к обновлению устройства, пользователям необходимо предоставить доступ к этому ресурсу. 

1. Перейдите в раздел Управление доступом (IAM) в учетной записи обновления устройства.

   :::image type="content" source="media/create-device-update-account/account-access-control.png" alt-text="Снимок экрана: Управление доступом в учетной записи обновления устройства." lightbox="media/create-device-update-account/account-access-control.png":::

2. Щелкните "добавить назначения ролей".

3. В разделе "Выбор роли" выберите роль обновления устройства в соответствии с заданными параметрами.
     - Администратор обновления устройства
     - Читатель обновления устройства
     - Администратор содержимого обновления устройства
     - Средство чтения содержимого обновления устройства
     - Администратор развертываний обновлений устройств
     - Средство чтения для развертывания обновлений устройств
     
   :::image type="content" source="media/create-device-update-account/role-assignment.png" alt-text="Снимок экрана назначения ролей управления доступом в учетной записи обновления устройства." lightbox="media/create-device-update-account/role-assignment.png":::
    
    [Сведения об управлении доступом на основе ролей в обновлении устройства для центра Интернета вещей](device-update-control-access.md) 
    
4. Назначение доступа пользователю или группе Azure AD
5. Щелкните Сохранить
6. Теперь вы можете использовать возможности обновления устройства в центре Интернета вещей.

## <a name="next-steps"></a>Дальнейшие действия

Попробуйте обновить устройство с помощью одного из следующих кратких руководств:

 - [Обновление устройств в симуляторе](device-update-simulator.md)
 - [Обновление устройств на микрокомпьютере Raspberry Pi](device-update-raspberry-pi.md)
 - [Обновление устройств в рамках агента пакета Ubuntu Server 18.04 для x64](device-update-ubuntu-agent.md)

[Дополнительные сведения об учетной записи и экземпляре обновления устройства.](device-update-resources.md) 

[Дополнительные сведения о ролях управления доступом для обновления устройств. ](device-update-control-access.md) 

