---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: ac708eb2ac79a74b8f4e09a7306a42665b3aca94
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "67185344"
---
#### <a name="to-delete-a-cloud-appliance"></a>Удаление облачного устройства

1. Войдите на портал Azure.
2. Вы можете удалить только то деактивированное устройство, которое не содержит данных. Сначала удалите данные на устройстве, или же вы можете [выполнить отработку отказа данных](../articles/storsimple/storsimple-8000-device-failover-cloud-appliance.md) в контейнерах томов на другое устройство. После удаления данных вы будете готовы отключить устройство.
3. На странице службы диспетчера устройств StorSimple щелкните **Устройства**, а затем выберите устройство. Щелкните его правой кнопкой мыши и выберите **Отключить**.
4. После деактивации устройства щелкните его правой кнопкой мыши и выберите **Удалить**.

    ![Выбор деактивированного устройства и его удаление щелчком](./media/storsimple-8000-delete-cloud-appliance/delete-cloud-appliance1.png)

5. Введите имя устройства, чтобы подтвердить его удаление. После удаления устройства список устройств обновится.

    ![Подтверждение удаления](./media/storsimple-8000-delete-cloud-appliance/delete-cloud-appliance2.png)

6. После удаления устройства вы получите уведомление.

    ![Уведомление об успешном удалении устройства](./media/storsimple-8000-delete-cloud-appliance/delete-cloud-appliance4.png)

7. Список устройств обновляется, чтобы указать удаленное устройство.

    ![Обновленный список устройств](./media/storsimple-8000-delete-cloud-appliance/delete-cloud-appliance5.png)
