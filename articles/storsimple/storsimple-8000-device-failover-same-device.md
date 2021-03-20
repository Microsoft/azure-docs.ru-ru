---
title: Руководство по отработку отказа физического устройства StorSimple на то же устройство
description: Сведения о том, как выполнить отработку отказа физического устройства StorSimple 8000 на себя в случае аварии.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/23/2017
ms.author: alkohli
ms.openlocfilehash: 17c116194aa52a82246bcee9114824e8a918ebbd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96017106"
---
# <a name="fail-over-your-storsimple-physical-device-to-same-device"></a>Отработка отказа физического устройства StorSimple на то же устройство

## <a name="overview"></a>Обзор

В этом руководстве описаны шаги, необходимые для отработки отказа физического устройства StorSimple серии 8000 на то же устройство в случае аварии. StorSimple использует функцию отработки отказа устройства для переноса данных с исходного физического устройства в центре обработки данных на другое физическое устройство. Рекомендации в этом руководстве относятся к физическим устройствам StorSimple серии 8000 под управлением версий программного обеспечения с обновлением 3 и более поздних.

Чтобы узнать больше об отработке отказа устройств и его использовании для восстановления после аварии, см. статью [Failover and disaster recovery for your StorSimple 8000 series device](storsimple-8000-device-failover-disaster-recovery.md) (Отработка отказа и аварийное восстановление для устройств StorSimple серии 8000).

Сведения об отработке отказа с одного физического устройства на другое физическое устройство см. в статье [Fail over to a StorSimple 8000 series physical device](storsimple-8000-device-failover-physical-device.md) (Отработка отказа на то же физическое устройство StorSimple серии 8000). Сведения об отработке отказа физического устройства StorSimple на облачное устройство StorSimple см. в статье [Fail over to your StorSimple Cloud Appliance](storsimple-8000-device-failover-cloud-appliance.md) (Отработка отказа на облачное устройство StorSimple).


## <a name="prerequisites"></a>Предварительные условия

- Обязательно ознакомьтесь с рекомендациями по отработке отказа устройств. Дополнительные сведения см. в статье с [общими рекомендациями по отработке отказа устройств](storsimple-8000-device-failover-disaster-recovery.md).


## <a name="steps-to-fail-over-to-the-same-device"></a>Шаги по отработке отказа на то же устройство

Если необходимо выполнить отработку отказа на то же устройство, выполните следующие действия.

1. Создайте облачные моментальные снимки всех томов на устройстве. Дополнительные сведения см. в статье [Use the StorSimple Device Manager service in Azure portal to manage backup policies](storsimple-8000-manage-backup-policies-u2.md) (Использование службы диспетчера устройств StorSimple на портале Azure для управления политиками резервного копирования).
2. Восстановите параметры устройства по умолчанию. Выполните подробные инструкции по [восстановлению параметров по умолчанию для устройства StorSimple](storsimple-8000-manage-device-controller.md#reset-the-device-to-factory-default-settings).
3. Перейдите к службе диспетчера устройств StorSimple, а затем выберите **Устройства**. В колонке **Устройства** для старого устройства должно отображаться состояние **Отключено**.

    ![Исходное устройство не в сети](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev2.png)

4. Настройте устройство и снова зарегистрируйте его в службе диспетчера устройств StorSimple. У только что зарегистрированного устройства отображается состояние **Готово к настройке**. Имя нового устройства такое же, что и у старого, но дополнено цифрой, чтобы обозначить, что устройство восстановлено до уровня заводских настроек и зарегистрировано снова.

    ![Только что зарегистрированное устройство готово к настройке](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev3.png)
5. Нужно выполнить настройку нового устройства. Дополнительные сведения см. в разделе [Выполнение минимальной настройки устройства](storsimple-8000-deployment-walkthrough-u2.md#step-4-complete-minimum-device-setup). В колонке **Устройства** состояние устройства изменится на **Включено**.

   > [!IMPORTANT]
   > **Сначала выполните минимальную настройку, иначе аварийное восстановление может завершиться сбоем.**

    ![Только что зарегистрированное устройство включено](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev7.png)

6. Выберите старое устройство (состояние "Отключено") и на панели команд щелкните **Отработка отказа**. В колонке **Отработка отказа** выберите старое устройство в качестве источника, а новое устройство укажите в качестве целевого устройства.

    ![Сводка отработки отказа](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev11.png)

    Подробные инструкции см. в разделе "Отработка отказа на другое физическое устройство".

7. Будет создано задание восстановления устройства, которое можно отслеживать в колонке **Задания**.

8. После успешного выполнения задания перейдите на новое устройство и откройте колонку **Контейнеры томов**. Убедитесь, что все контейнеры томов из старого устройства перенесены на новое устройство.

   ![Контейнеры томов перенесены](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev13.png)

9. После завершения отработки отказа старое устройство можно отключить и удалить с портала. Выберите старое устройство (отключенное), щелкните его правой кнопкой мыши и выберите **Отключить**. После отключения устройства его состояние обновится.

     ![Исходное устройство отключено](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev14.png)

10. Выберите отключенное устройство, щелкните его правой кнопкой мыши и выберите **Удалить**. Это удалит устройство из списка устройств.

    ![Исходное устройство удалено](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev15.png)



## <a name="next-steps"></a>Дальнейшие действия

* После отработки отказа, возможно, нужно будет [отключить или удалить устройство StorSimple](storsimple-8000-deactivate-and-delete-device.md).
* Сведения об использовании службы диспетчера устройств StorSimple см. в статье [Use the StorSimple Device Manager service to administer your StorSimple device](storsimple-8000-manager-service-administration.md) (Администрирование устройства StorSimple с помощью службы диспетчера устройств StorSimple).

