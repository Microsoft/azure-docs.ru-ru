---
title: Сброс и повторная активация устройства Azure Stack пограничной Pro | Документация Майкрософт
description: Узнайте, как очистить данные, а затем повторно активировать устройство Azure Stack пограничной Pro.
services: databox
author: v-dalc
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 03/03/2020
ms.author: alkohli
ms.openlocfilehash: 95fbe5dc4a934fee10c558f640bc24f261203a33
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102443559"
---
# <a name="reset-and-reactivate-your-azure-stack-edge-pro-device"></a>Сброс и повторная активация устройства Azure Stack пограничной Pro

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

В этой статье описывается, как сбросить, перенастроить и повторно активировать устройство Azure Stack пограничной Pro, если у вас возникли проблемы с устройством, или вам нужно запустить его по какой бы иной причине.

После сброса устройства для удаления данных необходимо повторно активировать устройство в качестве нового ресурса. Сброс устройства приводит к удалению конфигурации устройства, поэтому вам потребуется перенастроить устройство с помощью локального веб-интерфейса.

Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
>
> * Очистка данных с дисков данных на устройстве
> * Повторно активируйте устройство, создав новый заказ, перенастройте устройство и активируйте его.

## <a name="reset-data-from-the-device"></a>Сброс данных с устройства

Чтобы очистить данные с дисков данных устройства, необходимо сбросить устройство.

Перед сбросом при необходимости создайте копию локальных данных на устройстве. Данные можно скопировать с устройства в контейнер хранилища Azure.

>[!IMPORTANT]
> Сброс устройства приведет к удалению всех локальных данных и рабочих нагрузок с устройства и не может быть отменено. Сбросьте устройство, только если вы хотите запустить заново с устройством.

Вы можете сбросить устройство в локальном веб-ИНТЕРФЕЙСе или в PowerShell. Инструкции для PowerShell см. в статье [Сброс параметров устройства](./azure-stack-edge-connect-powershell-interface.md#reset-your-device).

[!INCLUDE [Reset data from the device](../../includes/azure-stack-edge-device-reset.md)]

## <a name="reactivate-device"></a>Повторная активация устройства

После сброса устройства потребуется повторно активировать устройство в качестве нового ресурса. После размещения нового заказа необходимо будет перенастроить и повторно активировать новый ресурс.

Чтобы повторно активировать существующее устройство, выполните следующие действия.

1. Создайте новый заказ для существующего устройства, выполнив действия, описанные в разделе [Создание нового ресурса](azure-stack-edge-gpu-deploy-prep.md?tabs=azure-portal#create-a-new-resource). На вкладке **адрес доставки** выберите **у меня уже есть устройство**.

   ![Не указывать новое устройство в адресе доставки](./media/azure-stack-edge-reset-reactivate-device/create-resource-with-no-new-device.png)

1. [Получите ключ активации](azure-stack-edge-gpu-deploy-prep.md?tabs=azure-portal#get-the-activation-key).

1. [Подключитесь к устройству](azure-stack-edge-gpu-deploy-connect.md).

1. [Настройте сеть для устройства](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md).

1. [Настройка параметров устройства](azure-stack-edge-gpu-deploy-set-up-device-update-time.md).

1. [Настройка сертификатов](azure-stack-edge-gpu-deploy-configure-certificates.md).

1. [Активируйте устройство](azure-stack-edge-gpu-deploy-activate.md).

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [подключиться к устройству Azure Stack пограничной Pro](azure-stack-edge-gpu-deploy-connect.md).
