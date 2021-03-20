---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: a01f91a81629800d3f03b907c65f05433b6163e6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93375959"
---
#### <a name="to-stop-and-start-a-cloud-appliance"></a>Остановка и запуск облачного устройства

1. Чтобы остановить облачное устройство, перейдите к виртуальной машине данного устройства.
    ![Виртуальная машина облачного устройства StorSimple](./media/storsimple-8000-stop-restart-cloud-appliance/sca-stop-restart1.png)

2. На панели команд нажмите кнопку **Остановить**.

    ![Виртуальная машина облачного устройства StorSimple 2](./media/storsimple-8000-stop-restart-cloud-appliance/sca-stop-restart2.png)

3. При запросе подтверждения нажмите кнопку **Да**.

    ![Виртуальная машина облачного устройства StorSimple 3](./media/storsimple-8000-stop-restart-cloud-appliance/sca-stop-restart3.png)

4. При остановке виртуальная машина освобождается. Во время остановки облачное устройство переходит в состояние **Отмена выделения**. После остановки облачного устройства его состояние будет **Остановлено (освобождено)**.

    ![Виртуальная машина облачного устройства StorSimple 4](./media/storsimple-8000-stop-restart-cloud-appliance/sca-stop-restart4.png)

5. После остановки виртуальной машины нажмите кнопку **Запустить** (кнопка становится доступной), чтобы инициировать запуск. После запуска облачного устройства его состояние будет **Запущено**.

    ![Виртуальная машина облачного устройства StorSimple 5](./media/storsimple-8000-stop-restart-cloud-appliance/sca-stop-restart5.png)

Используйте следующие командлеты для остановки и запуска облачного устройства.

`Stop-AzureVM -ServiceName "MyStorSimpleservice1" -Name "MyStorSimpleDevice"`

`Start-AzureVM -ServiceName "MyStorSimpleservice1" -Name "MyStorSimpleDevice"`

#### <a name="to-restart-a-cloud-appliance"></a>Перезапуск облачного устройства

Чтобы перезапустить облачное устройство, перейдите к виртуальной машине данного устройства. На панели команд нажмите кнопку **Перезапустить**. При появлении запроса подтвердите перезапуск. Когда облачное устройство будет готово к использованию, его состояние будет **Работает**.

![Виртуальная машина облачного устройства StorSimple 6](./media/storsimple-8000-stop-restart-cloud-appliance/sca-stop-restart6.png)

Используйте следующий командлет для перезапуска облачного устройства.

`Restart-AzureVM -ServiceName "MyStorSimpleservice1" -Name "MyStorSimpleDevice"`

