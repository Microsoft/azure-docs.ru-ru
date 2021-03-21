---
title: Развертывание обновления с помощью обновления устройства для центра Интернета вещей Azure | Документация Майкрософт
description: Развертывание обновления с помощью обновления устройства для центра Интернета вещей Azure.
author: vimeht
ms.author: vimeht
ms.date: 2/11/2021
ms.topic: how-to
ms.service: iot-hub-device-update
ms.openlocfilehash: 0a11c8e8946229941c1fe60f7f2ce84d9fadb2ed
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101680027"
---
# <a name="deploy-an-update-using-device-update-for-iot-hub"></a>Развертывание обновления с помощью обновления устройства для центра Интернета вещей

Узнайте, как развернуть обновление на устройстве IoT с помощью обновления устройства для центра Интернета вещей.

## <a name="prerequisites"></a>Предварительные условия

* [Доступ к центру Интернета вещей с включенным обновлением устройства для центра Интернета вещей](create-device-update-account.md). Для центра Интернета вещей рекомендуется использовать уровень S1 (стандартный) или выше. 
* [Для подготовленного устройства успешно импортировано по крайней мере одно обновление.](import-update.md) 
* Устройство IoT (или симулятор), подготовленное для обновления устройства в центре Интернета вещей.
* [Для устройства IoT, которое вы пытаетесь обновить, назначен тег. Устройство является частью по крайней мере одной группы обновлений.](create-update-group.md)
* Поддерживаемые браузеры:
  * [Microsoft Edge](https://www.microsoft.com/edge)
  * Google Chrome

## <a name="deploy-an-update"></a>Развертывание обновления

1. Переход к [портал Azure](https://portal.azure.com)

2. Перейдите в колонку обновления устройства в центре Интернета вещей.

  :::image type="content" source="media/deploy-update/device-update-iot-hub.png" alt-text="Центр Интернета вещей" lightbox="media/deploy-update/device-update-iot-hub.png":::

3. Щелкните вкладку "Группы" в верхней части страницы. Дополнительные [сведения](device-update-groups.md) о группах устройств. 

  :::image type="content" source="media/deploy-update/updated-view.png" alt-text="Вкладка &quot;Группы&quot;" lightbox="media/deploy-update/updated-view.png":::

4. Просмотрите список группы и диаграмма соответствия обновлений. Вы должны увидеть новое обновление, доступное для группы устройств, со ссылкой на обновление в разделе ожидающие обновления (возможно, потребуется обновить один раз). [Дополнительные сведения о соответствии обновлений требованиям.](device-update-compliance.md) 

5. Выберите доступное обновление.

6. Подтвердите, что целевая группа выбрана правильно. Запланируйте развертывание, а затем нажмите "Развернуть обновление".

   :::image type="content" source="media/deploy-update/select-update.png" alt-text="Выбор обновления" lightbox="media/deploy-update/select-update.png":::

7. Посмотрите на диаграмму соответствия требованиям. По ней должно быть видно, что процесс обновления выполняется. 

   :::image type="content" source="media/deploy-update/update-in-progress.png" alt-text="Выполняется обновление" lightbox="media/deploy-update/update-in-progress.png":::

8. После успешного обновления устройства диаграмма соответствия требованиям и сведения о развертывании должны измениться, отражая одну и ту же информацию. 

   :::image type="content" source="media/deploy-update/update-succeeded.png" alt-text="Обновление выполнено" lightbox="media/deploy-update/update-succeeded.png":::

## <a name="monitor-an-update-deployment"></a>Слежение за развертыванием обновления

1. Щелкните вкладку "Развертывания" в верхней части страницы.

   :::image type="content" source="media/deploy-update/deployments-tab.png" alt-text="Вкладка &quot;Развертывания&quot;" lightbox="media/deploy-update/deployments-tab.png":::

2. Выберите созданное развертывание, чтобы просмотреть сведения о нем.

   :::image type="content" source="media/deploy-update/deployment-details.png" alt-text="Сведения о развертывании" lightbox="media/deploy-update/deployment-details.png":::

3. Щелкните "Обновить", чтобы просмотреть последние данные о состоянии. Делайте так, пока состояние не изменится на "Выполнено".


## <a name="retry-an-update-deployment"></a>Повтор развертывания обновления

Если по какой – то причине развертывание завершается сбоем, можно повторить развертывание для неисправного устройства. 

1. Перейдите на вкладку развертывания и выберите развертывание, которое завершилось сбоем. 

   :::image type="content" source="media/deploy-update/deployment-details.png" alt-text="Сведения о развертывании" lightbox="media/deploy-update/deployment-details.png":::

2. В области подробные сведения о развертывании щелкните состояние устройства сбой.

3. Щелкните "повторить попытку невыполненных устройств" и подтвердите уведомление. 

## <a name="next-steps"></a>Дальнейшие действия

[Устранение распространенных неполадок](troubleshoot-device-update.md)
