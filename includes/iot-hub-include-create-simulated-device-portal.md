---
title: включить файл
description: включить файл
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 02/26/2019
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 91b6f1ed06fbf1f5575650f96f4622b3df9ca083
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98186983"
---
<!-- This is the instructions for creating a simulated device you can use for testing routing.-->

Затем создайте идентификатор устройства и сохраните его ключ для последующего использования. Это удостоверение устройства используется приложением моделирования для отправки сообщений в Центр IoT. Эта возможность недоступна в PowerShell, как и при использовании шаблона Azure Resource Manager. Далее описано, как создать имитированное устройство на [портале Azure](https://portal.azure.com).

1. Откройте [портал Azure](https://portal.azure.com) и войти в учетную запись Azure.

2. Откройте **Группы ресурсов** и выберите нужную группу ресурсов. В этом руководстве используется **ContosoResources**.

3. В списке ресурсов выберите центр Интернета вещей. В этом руководстве используется **ContosoTestHub**. Из области концентратора выберите **Устройства IoT**.

4. Выберите **+ Создать**. В панели «Добавить устройство» введите идентификатор устройства. В этом руководстве используется **Contoso-Test-Device**. Оставьте пустым ключи и поставьте флажок **Автоматически создавать ключи**. Включите кнопку **Подключить устройство к центру IoT**. Щелкните **Сохранить**.

   ![Снимок экрана при добавлении устройства](./media/iot-hub-include-create-simulated-device-portal/add-device.png)

5. Теперь, когда устройство создано, щелкните его, чтобы просмотреть созданные ключи. Нажмите значок копирования для первичного ключа и сохраните его, например в блокноте, для этапа тестирования, описанного далее в этом руководстве.

   ![Сведения об устройстве, включая ключи](./media/iot-hub-include-create-simulated-device-portal/device-details.png)