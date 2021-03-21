---
title: Замена устройства Azure Stack пограничной Pro | Документация Майкрософт
description: Описывает, как получить замену Azure Stack устройстве Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alkohli
ms.openlocfilehash: 2196c9463569dc43092b38de58e0103104efed0c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102443477"
---
# <a name="replace-your-azure-stack-edge-pro-device"></a>Замена устройства Azure Stack пограничной Pro

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

В этой статье описывается, как заменить устройство Azure Stack пограничной Pro. Устройство на замену необходимо, если на имеющемся устройстве произошел сбой оборудования или требуется обновление. 


Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
>
> * Откройте запрос в службу поддержки для устранения проблем с оборудованием
> * Создайте новый заказ на замену устройства в портал Azure
> * Установка, активация устройства замены
> * Возврат исходного устройства

## <a name="open-a-support-ticket"></a>Создание запроса в службу поддержки

Если на имеющемся устройстве произошел сбой оборудования, отправьте запрос в службу поддержки, выполнив следующие действия.

1. Откройте запрос в службу поддержки Майкрософт и укажите, что хотите вернуть устройство. Выберите тип проблемы **оборудования Azure Stack пограничной Pro** и выберите подтип **проблемы с оборудованием** .  

    ![Открыть запрос в службу поддержки](media/azure-stack-edge-replace-device/open-support-ticket-1.png)  

2. Специалист по служба поддержки Майкрософту свяжется с вами, чтобы определить, может ли модуль замены полей (FRU) решить проблему и доступен для этого экземпляра. Если FRU недоступно или устройству требуется обновление оборудования, поддержка поможет вам разместить новый заказ и вернуть старое устройство.

## <a name="create-a-new-order"></a>Создание нового заказа

Создайте новый ресурс для активации заменяющего устройства, выполнив действия, описанные в разделе [Создание нового ресурса](azure-stack-edge-gpu-deploy-prep.md#create-a-new-resource).

> [!NOTE]
> Активация заменяющего устройства для существующего ресурса не поддерживается. Новый ресурс считается новым заказом. Вы начнете получать счета за 14 дней после отгрузки устройства.

## <a name="install-and-activate-the-replacement-device"></a>Установка и активация устройства замены

Выполните следующие действия, чтобы установить и активировать устройство замены.

1. [Установите устройство](azure-stack-edge-deploy-install.md).
2. [Активируйте устройство](azure-stack-edge-deploy-connect-setup-activate.md) для нового ресурса, созданного ранее.

## <a name="return-your-existing-device"></a>Возврат существующего устройства

Выполните все действия, чтобы вернуть исходное устройство:

1. [Сотрите данные с устройства](azure-stack-edge-return-device.md#erase-data-from-the-device).
2. [Инициация возврата устройства](azure-stack-edge-return-device.md#initiate-device-return) для исходного устройства.
3. [Вызовите курьера](azure-stack-edge-return-device.md#schedule-a-pickup).
4. После получения устройства в корпорации Майкрософт можно [Удалить ресурс](azure-stack-edge-return-device.md#delete-the-resource) , связанный с возвращенным устройством.


## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [вернуть устройство Pro Azure Stack](azure-stack-edge-return-device.md).
